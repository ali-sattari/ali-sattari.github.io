---
title: "Alerting Maturity Levels"
date: 2022-07-18T09:07:34+02:00
draft: true
---

OR
When to alert on what?
OR What your alerts say about you/your org?

<!--more-->

## Preface

We have all been there, when we go into a new company, or team, looking at their stack and specially alerting, and something feels a bit ... off. This is an honest attempt to categorize alerting into some bands or levels, in order to make it easier to reason about where things are and have a glimpse at the road ahead. Or in other words, to put words on those vague feelings. The levels don't mean to be followed in sequence, som ecan be skipped, others not. Moreover, this is by no means a yet-another-maturity-model, use it as such at your own risk. The end goal of any alert is (besides being actionable) to detect problems affecting your users fast enough, without being too trigger happy.
If you are confused on how this can help you, you can try by finding the level that describes your current state of alerting, then see if the service match the stage described, if not, look around to find out whether it is behind, or have some pre-mature engineering going on.

### L0: No alerts, Only human detection!

Let's be honest, we all have been here, maybe a weekend project of our own, our personal blog, or a very early stage startup or project. In each case, it is either lack of serious impact on audience, or the fact that someone is constantly fideling with the service (or both) that drives no alert mode. It works up to some extent, as far as the cost of setting up or using a service for monitoring and alerting is greater than any loss of profit from degraded services.

This has it's own benefits, no overhead of implementing and maintaing monitoring and alerting system, even if as easy as instrumentaiton of SaaS provider. The other benefit is low to no false positives. The downsides can be a background level of stress for operators of the system and a contsant inner-nag to periodically check if everythin works. This consumes precious human cognitive energy that imposes unnecessary stress, more so when the service attracts more customers and downtimes begin to matter more.

> Little to no user impact from downtime, or someone from the team always interacting with the product and can detect downtime reasonably fast.

### L1: Alerting on System Metrics (resources saturation)

System metrics refer to basic operating system resources, like CPU, memory, disk, and network usage. Load average in *nix systems is a notion mixing all these together (kinda). Monitoring them and alerting against them is often easy, there a re a lot of single-binary agents to collect these, and understanding them isn't that difficult, as you would be mostly dealing with percentages.

Sometimes system metrics and alerting based on them are seen as this legend of ancient times, when baremetals and servers as pets where the thing. They can still make sense, if you only have one or few hosts runing you services, then system metrics often correlate well with service quality and are much easier to monitor. But once you have some redundancy in your system, when saturation of one host's resources doesn't immediately impact user exerience, this mode stops being useful, and quickly becomes noisy.


### L2: Alerting on Service Metrics (api error/latency)

Monitoring a service's outward interactions is what is often meant by Service Metrics, for APIs this would often be HTTP responce code, and responce latency. This is already a few steps close to user experience, and it takes degradation of dependencies into account. These are best observed from a load-balancer point of view, and aggregated across all (identical) instances serving traffic. You might choose to group by HTTP verb or path, but that's often the extent to which they get detailed.

Once your service has some instrumentaiton and basic observability going on, this would be an obvious choice. This fits most when your service has few endpoints, and only one or two user journeys. Once user journeys start to get more complex, and involve multiple calls to different endpoints this stops being useful, e.g. traffic of some endpoints might hide some issues in other endpoints.

### L3: Alerting on SLI

Why? you are at the start of the SRE journey, getting familar with the landscape and just started defining user journeys and focusing on customer happiness as the key metric for service health. this can provide a smoother transition to the next level.
When? when you feel confident enough with you SLI capturing customer happiness, and start to get sick of alerting noise.

### L4: Alerting on Remaining Error Budget

Why? It is not trigger happy, meaning transiet issues won't wake you up. 
When? When you have SLIs and SLOs established for your service, and have an error budget policy in place.

### L5: Alerting on Multi-window Multi-burn-rate Error Budget

Why? you don't want to let the budget burn through to take action, you want to get alerted if it is burning too fast, but also recover quickly
When? mature service, architurecture fit for SLO, high volume of traffic round the clock

## Bonus level: proper context with alerts

* description
* links
* screenshot of the graph
* region/account

## Common scenarios

* being at SLI level, but alerting based on system -> lots of false positives, one node high CPU may not affect users noticably
