---
layout: post
title: jQuery DOM Performance
author: kevin-brown
date: 2014-12-14 20:30:00 EST
category: select2
tags: javascript jquery select2 programming
---

One of the biggest issues with older versions of Select2 was the slow performance when working with thousands of options in a `<select>`. This was primarily because Select2 had to generate a JavaScript object for each element, as well as a DOM element within the dropdown for each possible option. There was a [long-standing ticket][select2-performance] in the Select2 repository for improving the performance, with a few ideas having come up over the years and some improvements that eventually made their way into the code.

For the [rewrite of Select2][select2-rewrite], we decided that fixing the performance issues would be a major focus. As Select2 4.0 would be using the `<select>` element for all cases, we realized that telling everyone to use a remote data set wouldn't be the magical fix as we now generated all of the results in a consistent format. This post will cover the biggest improvements that were discovered along the process, as well as the reason why they were slow in the first place.

# Handling diacritics

The easiest issue to fix in Select2 involved how it stripped out the diacritics to allow for a fuzzy search to be done on results. This has the benefit of making Select2 much easier to use when working with other languages, but it had the downside of making result matching considerably slower at the same time. The issue was originally pointed out by [@ycdtosa after using a profiler][select2-diacritics], and after a [few jsPerf tests][jsperf-diacritics] we were able to get it to a state where it could be integrated into Select2.

We settled on the badly-named "What?" test from jsPerf, which outperformed the "String builder smarter" test (which is what we previously used) and gave us around an 8x increase in performance.

## How the old code worked

The old code worked by manually replacing each character that had a diacritic mark with the normalized version. It used a for loop, which would be almost perfect if every letter had a diacritic, but this is almost never the case as diacritic marks tend to be few and far between when working with large data sets.

~~~ js
function removeDiacritics(str) {
    var newStr = "", letter;
    for (var i = 0, len = str.length; i < len; i++) {
        letter = str[i];
        newStr += diacriticsMap[letter] || letter;
    }
    return newStr;
}
~~~

This code iterates over every letter in the string, even if it probably doesn't need to be checked. While this doesn't affect performance _that_ much on smaller data sets, when working with large data sets it adds up very quickly. The code checks to see if the letter is in the diacritics map, using the normalized value if it is, and adds the letter to the string containing the normalized data.

There is clearly a few places that can be optimized, the first one being the fact that it checks _every single character_ in the string. The second place that we can improve is the for loop, as there are other way to replace characters in a string, such as [JavaScript's `String.replace` method][mdn-string-replace].

## How the new code works

The new code works using the `replace` method provided by JavaScript, in combination with a regular expression and a callback function. In most cases, the `replace` method is used to replace one string with another, but it can be used to [match regular expressions][mdn-regular-expressions] and replace only the part of the string that matches them.

~~~ js
function removeDiacritics(str) {
    return str.replace(/[^\u0000-\u007E]/g, function(a) {
        return diacriticsMap[a] || a;
    });
}
~~~

This uses a regular expression matching all characters not within the [basic ASCII character set][ascii], allowing us to skip the characters which we _know have no diacritics_ and will never need to be replaced. It also uses a callback function when replacing the characters which will quickly check if the character is in the diacritics map and replace it if needed.

# Some tips before we go further

As Select2 can be used with many different types of data, ranging from a standard `<select>` to a remote data source, it internally uses a standardized object representation to determine how it should be rendered in different parts of Select2. The internal object stores information such as if the option is disabled or selected, the value that should be sent to the server (the `id`), and the option text. It also includes information such as if the option has children (it's an `<optgroup>`), along with the objects for _those_ children.

As these objects are used just about everywhere, any performance issues in creating them will slow down _the entire plugin_, but there are some things which were identified which we could improve. Before we go over those, there is something which needs to be covered, and that is how to find performance issues within your own plugins.

## Learn to love JavaScript profilers

We extensively used the [JavaScript profiler provided with Google Chrome][chrome-js-profiler] when we were searching for the slow parts of our code. Since we knew that the performance issues happened when working with large data sets, we knew to look for the parts of the code that were called the most number of times, not just the parts of the code that were slow. Improving a function which normally takes 30ms to run and making it run in 10ms is great, but if it only runs once or twice then it doesn't help the problem that much. Fixing small performance issues works well if there are many of them and they are easy to identify, but it is quicker if you can fix the larger ones.

This means looking at the waterfall and looking for the slow performing methods, but also knowing how to filter them. If you start looking down the waterfall and you see the internal methods of jQuery are your slow spots, then look around and see if you can either avoid using them or make them work faster.

## Use caching where possible

If you have functions which perform the same calls on an object (in our case, a DOM element), and the result from the function is not dependent on other information, you may have better luck caching the data in a central place instead of wasting time computing the information and getting the same result.

> The fastest call is the one that is never made.

In our case, since we constantly needed to get the object form of an `<option>` tag, we could cache the object itself on the element and avoid computing it if we had already done it before. We had the option of either using our own caching mechanism, such as a global object which mapped elements to objects, or we could use a caching mechanism already provided with jQuery. jQuery allows you to associate data with an element using the `$el.data()` method, which was perfect for us as our key is a DOM element.

# Don't use `.data()` unless you need `data-*` attribute information

jQuery provides a [`.data` method][jquery-el-data] that can be called on jQuery elements, which will allow you to set and retrieve any type of data, not just the string data that is limited to the HTML5 `data-*` attributes. This allows users of your plugin to specify options using the `data-*` attributes which you can then use within your plugin, and it also allows you to set arbitrary data on an element, in the same way that a [key-value store or data][kv-store] works.

## jQuery gets the `data-*` attributes every time `.data()` is called

If you don't need information from the data attributes, or you do but you've already stored it elsewhere, there is no need to get all of the information from the attributes before getting the information you have stored on the element. There is a large amount of overhead in getting the data attributes from the element, around 10ms per call, that will add up quickly when you need to cache data on a large number of elements.

## Use `$.data` for caching information

jQuery provides the [`$.data` method][jquery-data] for saving data on a DOM element, completely independent of the HTML5 data APIs. Internally, it is what jQuery uses when setting the data on an element and it does not alter the element itself.

[ascii]: https://en.wikipedia.org/wiki/ASCII
[chrome-js-profiler]: https://developer.chrome.com/devtools/docs/cpu-profiling
[jquery-data]: https://api.jquery.com/jQuery.data/
[jquery-el-data]: https://api.jquery.com/data/
[jsperf-diacritics]: http://jsperf.com/diacritics/29
[kv-store]: http://dba.stackexchange.com/questions/607/what-is-a-key-value-store-database
[mdn-regular-expressions]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions
[mdn-string-replace]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace
[select2-diacritics]: https://github.com/ivaynberg/select2/issues/781#issuecomment-38979100
[select2-performance]: https://github.com/ivaynberg/select2/issues/781
[select2-rewrite]: https://github.com/ivaynberg/select2/pull/2743
