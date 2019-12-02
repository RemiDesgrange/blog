---
title: "Why You Should Use Timestamptz"
date: 2019-07-10T10:30:46+02:00
draft: false
tags: ["Postgresql", "Database", "Time", "Timestamp", "Time management"]
categories: ["Postgresql", "Database", "English"]
---

I see a lot project using `timestamp` data type in there databases. I tend to think that this is a
bad idea.

# Dev Is Not Prod

Your local setup may have a timezone which reflect your physical location. This may not be the case
of your server(s), which could have a specific timezone, or, UTC time. Which is better by the way (more
on than later). So you _will_ experience bug and issues if you do not pay attention to it.

# You Are Now Cloud Native

For now you, your opensource project, or your company are running your software [^1] on premises on
your server, you have control over the timezone of your server. And you only have customer in your
country anyway (but... US, Russia, etc... Yeah more on that later too).

[^1]: sorry "app".

You will migrate, maybe just for integration or stuff in the cloud. 
[75% of database are migrating to the cloud](https://www.gartner.com/en/newsroom/press-releases/2019-07-01-gartner-says-the-future-of-the-database-market-is-the).
This is huge. And guess what cloud database _do not_ care about your timezone.

# The cancer of server with timezone set to they're physical location

Server should have timezone set to UTC. Period. In my country, France, we have only one timezone.
But it change twice a year for winter/summer hour (it's due to change be 2021). So you'll have
inconsistency in your data based on when you insert them... Quite nice.

I have another example of how this time without timezone affect real project. Takes for example the
French standard to exchange infrastructure data about fiber (between ISP) : [GraceTHD](https://github.com/GraceTHD-community/GraceTHD).
They use `timestamp`. You may think, ok but, France, it's this :
!["France Metropolitaine" as we say. Image by https://fr.wikipedia.org/wiki/France_m%C3%A9tropolitaine](/images/Metropolitan_France.svg)

But it's not, France is this:
![France all of it, image by https://fr.wikipedia.org/wiki/France_m%C3%A9tropolitaine](/images/France_Overseas.svg)

So you may imagine that in [Mayotte](https://fr.wikipedia.org/wiki/Mayotte) the timezone is not same, in fact it's UTC+3.


# Conclusion

* set your server to UTC
  * it is better for logs
  * it is better for debugging
  * it is better for your team
  * it is better for _everything_
* Change the datatype from `timestamp` to `timestamptz`. Or `timetz`.
* Handle timezone in your software [^1].
