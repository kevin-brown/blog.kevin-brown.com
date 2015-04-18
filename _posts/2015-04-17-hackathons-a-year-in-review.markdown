---
layout: post
title: Hackathons - A Year In Review
author: kevin-brown
date: 2015-04-17 20:00:00 EDT
category: hackathons
tags: hackathons
---

After [HackUMass][hackumass], the final hackathon that I attended this school year, I decided that I would do a "year in review" for the hackathons that I attended. I attended my first hackathon this year, [HackMIT][hackmit], and by the end of the year I was able to attend four others (for a total of five). Each one put a new spin on what I thought a hackathon was and inspired me to continue attending hackathons in the future.

Hackathons are designed to be a learning experience, where you are given free reign over what you want to create, and you have the opportunity to interact and collaborate with sponsors to create new things. For each hackathon, I'm going to discuss what we made and why separated the hackathon from the others that I attended.

## HackMIT

My first ever hackathon was [HackMIT][hackmit], hosted at [MIT][mit]. It was a 24 hour hackathon that took place on October 4th and 5th and we went there (first time for everyone) with two teams that made a grand total of five people. There were a little more than 1,000 other hackers there with us, all packed into the [Johnson Ice Rink][johnson-arena] with tons of sponsors who were interested in letting us use their APIs.

The team that I was part, consiting of [@gunthercox][gunthercox] and I, created [BuzzFinder][buzzfinder] which was a tool to find the best days to read BuzzFeed. It was a web application, written in Python, that would analyze the sentiment of comments in [BuzzFeed][buzzfeed] articles using [HP's IDOL OnDemand][idol-on-demand] sentiment analysis API. This was our first hack, and while it was a great experience at MIT, we learned to come prepared to hackathons in the future with an idea of what we want to do.

The other team that we went with choice to do a hack involving hardware, specifically Synaptics experimental touch drivers and sensors. They would eventually go on to win the Synaptics prize, as no other teams could get the hardware to function, and learned the value of asking sponsors for help.

HackMIT was the first of many hackathons we went to, and kicked off the interest in hackathons at our university.

## HackRPI

At [HackRPI][hackrpi], we had almost the same group as HackMIT minus a few people (due to things coming up at the last minute) and split into two teams again. Unlike HackMIT, where there were over 1,000 hackers packed into the same room as the spnsors, at HackRPI there were only around 500 hackers and we were spread out across a few classrooms.

The team that I was part of consisted of myself, two other students from my university, and a high school hacker who we met when we were talking with sponsors. Together we created [SpeakForMe][speak-for-me], a web application that was designed to parse incoming sound files and split words up into individual, reusable parts that could be reconstructed to create different words. In theory, this would have created a very realistic text-to-speech program based on anyone's voice. In reality, we hit major roadblocks when parsing the speech and were not able to accomplish what we were hoping to.

The other team, which consisted of one student from our university and a few others from [RPI][rpi], created [TiMan][timan], a program that was designed to improve your time management skills and allow you to better estimate future tasks.

## hackNY

The third hackathon we went to, [hackNY][hackny], took place on March 7th and 8th. It was also the first one where we tried to get as many new hackers involved as possible, and we were able to get two additional hackers registered before registration closed. We took early morning trains (one of which was delayed a few hours) to get to [Columbia University][columbia], which made hackNY the first hackathon where we didn't drive there in a car.

We went into the hackathon knowing we would need to recoup our investment in train tickets, and we knew that [MongoDB][mongodb] usually gave out a prize at hackathons that they sponsored. So we took MongoDB as a starting location and decided that we would hook it up to a few different [Open Data sources][nyc-open-data] as they are typically non-relational and lack a defined schema. The result was the [New York Startup Index][nysi], an interactive website powered by [Flask][flask] which used [Leaflet.js][leaflet] to display different data sources on a map of New York City.

This hackathon was also the first hackathon where the team that I was on won a prize - the MongoDB one that we set out for. What impressed us, and the MongoDB judges, the most was how well MongoDB could handle the [geospatial data][mongo-geospatial] which was used to overlay data points (public transportation, crimes, technology companies, etc.) onto the map of New York City.

## Hack Holyoke

The next hackathon we went to, [Hack Holyoke][hack-holyoke], took place on April 3rd and 4th. Because there were only 150 hackers being admitted on an application basis, and there was a hackathon the weekend after it, we didn't spend a lot of time trying to get new hackers admitted.

This was the first hackathon where the team I was on had nothing to show during the demonstration time, so we left early. We attempted to get the [Intel Edison][intel-edison] to work with [Johnny-Five][johnny-five] and [Cylon.js][cylonjs], but suffered from obscure issues which chewed away the time we had to use.

## HackUMass

The next weekend after Hack Holyoke, we went to [HackUMass][hackumass] which was hosted at the Amherst campus of the [University of Massachusetts Amherst][umass]. Because the registration, which involved getting a ticket on [EventBrite][eventbrite], was closed in around 22 hours, we were only able to secure three tickets for our team. Throughout the month after that, one of the tickets was passed on to a new hacker and another ticket was moved on to our team, giving us a team of four who attended.

This was the first hackathon we attended which was not 24 hours long, instead it took place from the morning of April 11th to the night of April 12th for a total of 36 hours. The four of on on the team were [Python][python] programmers, but we were all interested in trying out new things. Knowing that Intel would again be at HackUMass, most likely with the [Intel Edison][intel-edison], we decided to port the robotics library [Cylon.js][cylonjs] over to Python. The result of our efforts was [Zorg][zorg], a library which shared a similar interface to Cylon but was designed with common Python patterns in mind.

Zorg would eventually go on to win the "Best use of Intel Edison" prize at HackUMass. The project is still being actively developed, and support is being expanded to other platforms such as the [Raspberry Pi][raspberry-pi] and [Arduino Firmata][firmata] as well as other devices that can be controlled.

[buzzfeed]: http://buzzfeed.com/
[buzzfinder]: http://challengepost.com/software/buzzfinder
[columbia]: http://www.columbia.edu/
[cylonjs]: http://cylonjs.com/
[eventbrite]: https://www.eventbrite.com/e/hackumass-ii-tickets-15487208658
[firmata]: https://www.arduino.cc/en/Reference/Firmata
[flask]: http://flask.pocoo.org/
[gunthercox]: https://github.com/gunthercox
[hack-holyoke]: http://www.hackholyoke.org/
[hackmit]: https://hackmit.org/
[hackny]: http://hackny.org/a/spring2015hackathon/
[hackrpi]: http://hackrpi.com/
[hackumass]: http://hackumass.com/
[idol-on-demand]: https://www.idolondemand.com/
[intel-edison]: http://www.intel.com/content/www/us/en/do-it-yourself/edison.html
[johnny-five]: http://johnny-five.io/
[johnson-arena]: https://www.google.com/maps/place/Johnson+Ice+Rink/@42.358466,-71.096413,17z/
[leaflet]: http://leafletjs.com/
[mit]: http://mit.edu/
[mongodb]: https://www.mongodb.com/
[mongo-geospatial]: http://docs.mongodb.org/manual/applications/geospatial-indexes/
[nyc-open-data]: https://nycopendata.socrata.com/
[nysi]: http://challengepost.com/software/new-york-startup-index
[raspberry-pi]: https://www.raspberrypi.org/
[rpi]: http://rpi.edu/
[python]: https://www.python.org/
[speak-for-me]: http://challengepost.com/software/speak-for-me-5ecze
[timan]: http://challengepost.com/software/timan
[umass]: https://umass.edu/
[zorg]: https://zorg-framework.github.io/
