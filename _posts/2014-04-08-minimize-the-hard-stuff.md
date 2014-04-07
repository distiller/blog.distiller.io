## Overview

We're huge proponents of operating lean and validating our assumptions as quickly as possible. It's not only *why* we built [Distiller](http://distiller.io), it's *how* we built it.

From an implementation perspective, this approach requires that we clearly define our core thesis and focus on it. In doing so, we figure out what the truly hard parts are and solve them, but make anything that's not valuable into an "easy" part and find a way to not build it. In order to feel comfortable with that approach, we reduce the overhead of later changes through pragmatic isolation.

## Focusing on the Idea

[Solene Maitre](http://twitter.com/solenema) recently wrote a great post on the [Minimum Viable Stack](https://medium.com/design-startups/7dcb59c9fe1f). We're 100% on board with this approach. As engineers, it may have taken us a while to get there, but we've learned this lesson a million times. Testing an idea is really hard. Regardless of what you're building, there will be lots of technical hurdles. There's really just no need to invent more.

## The Hard Part

Consistently building for iOS is surprisingly hard. It only takes a few minutes on [Stack Overflow](http://stackoverflow.com/questions/tagged/xcodebuild) to see *how* hard. The iOS developer ecosystem outside of Apple has produced some amazing tools, such as [CocoaPods](http://cocoapods.org), [Kiwi](https://github.com/allending/Kiwi), and [TestFlight](https://testflightapp.com/) to name a few. But developers have to make consistently herculean efforts to keep these tools running within Apple's toolchain. And that's for one team on one project. We're supporting every tool we can find in any manner that a developer has used it.

Additionally, running these tools requires managed OS X environments. No [Docker](https://www.docker.io/), no [Heroku](https://www.heroku.com/), no [AWS](http://aws.amazon.com/). Stripped of our usual bag of tricks, we've had a lot to figure out.

Given that set of circumstances, why would we try to do *anything* fancy for any of the other pieces?

## The Easy Parts

"Easy" is probably not the right word, but these are solved problems. When you are hosting a service for building, you need a lot more than a build script. Web hosting, queueing, authentication, and billing, just for starters. But those pieces do nothing to either differentiate the product or help figure out what matters. So we've made the minimum viable investment in each.

### Web + Data

[Distiller](http://distiller.io) was born of a need that we discovered while iterating on iOS apps. We built those apps using [Parse](http://parse.com), including investing some time making it really fast for us to build [websites and APIs on top of Parse.](https://github.com/utahstreetlabs/parseapp-cljs) So when we needed a web interface for our service, there was really no reason to use anything else.

Conveniently enough, Parse also gives us decent data storage. If we were building from scratch, would we use [MongoDB](https://www.mongodb.org/) for our data? Probably not. But that can be a problem for another day. It's good enough for our current needs and we don't have to manage it. Done.

In addition to providing a website to manage your projects, view builds and deploy, we have tasks that need to be run on a scheduled basis. Collecting metric data and performing data cleanup are two such cases. Parse's [background jobs](https://www.parse.com/docs/cloud_code_guide#jobs) handle these tasks perfectly and couldn't be easier to write using our existing data model.

### Queueing + On-Demand Jobs

We integrate the OS X build hosts (VMs) with configuration and event data using queues. This approach has at least three great advantages:

* linear scalability and distribution of build hosts
* clean system boundary between data management and build systems
* build boxes (where client code is present for the duration of a build) require no inbound port access

There's obviously a ton of queueing software out there, but it's not like we're pushing the boundaries. Low overhead so often trumps architectural purity. [IronMQ](http://www.iron.io/mq) is as easy as it gets for setup and use, with support for the basics that we need from a queue.

[Iron.io](http://www.iron.io/) also happens to offer [IronWorker](http://www.iron.io/worker), which we can trigger directly off of their ["push queues"](http://dev.iron.io/mq/reference/push_queues/). When we have on-demand jobs that don't require the precious resources of our OS X build boxes, we use these. For example, requesting a deploy only requires fetching the built assets, adding the release notes, and pushing them to TestFlight.

### Other

There are many other pieces required to get a build service off the ground, and we'll probably get to talking about our use of these over time, but for now here's a short list of other services we've leveraged.

* **Authentication**: [GitHub OAuth](https://developer.github.com/v3/oauth/)
* **Email**: [SendGrid](http://sendgrid.com)
* **Customer Metrics**: [Google Analytics](http://analytics.google.com), [Mixpanel](https://mixpanel.com)
* **Operational Metrics**: [Librato](https://metrics.librato.com/)
* **Alerting**: [PagerDuty](http://www.pagerduty.com/)
* **Status**: [StatusPage.io](https://www.statuspage.io/)

## Isolation

Even among engineers who have rallied behind building a service on top of existing infrastructure, there's always that discussion about abstracting the service provider in case of a later need to change providers (or replace with an in house solution). There's no need to write anything new to cover that kind of thinking. If you need a refresher, try any of these:

* [You Aren't Gonna Need It](http://xp.c2.com/YouArentGonnaNeedIt.html)
* [Premature Generalization](http://c2.com/cgi/wiki?PrematureGeneralization)
* [Planning for vs. Reacting to Change](http://devlicio.us/blogs/billy_mccafferty/archive/2006/09/20/Planning-for-vs.-Reacting-to-Change.aspx)

But you also don't want to go digging in every corner of your source when that one change does come up. You also don't want to listen to an endless stream of "I told you so." The great news is, the kind of programming structure that keeps us sane on a daily basis (i.e. some semblence of organization) is all that's needed to prepare for this case. If you can't figure out where all the email code is when you decide to switch providers, how do you figure it out when you need to modify how you use the current provider? Stick to that kind of thinking and when the $$$ start rolling in, you can deal with any issues in your current infrastructure.

## Conclusion

Building a business is hard, but it's getting easier all the time. The pieces that you need to get a viable product into the market are not only readily available, but they can be easily rewired to let you iterate while you figure it out. We're totally confident in our ability to scale a service, but we're not going to spend any time scaling until we know we have it figured out.

If the business you're figuring out happens to be on iOS, we'd be happy to [help you get there faster.](http://distiller.io)
