---
title: "Alerting Maturity Levels"
date: 2022-07-18T09:07:34+02:00
draft: true
---

OR
When to alert on what?

<!--more-->

## Levels 0 to 5

* This is not to label things (like old school maturity models), but to map each team/company where in journey is standing.
* Some of the levels can be skipped some can not, there is no inherent supriority to higher levels, they are necessitated by their more complex environment.
* End goal of any good alert is to detect problems fast enough, without being trigger happy!
* Per each level: what it means, why it works?, when to move?

### L0: No alerts, Only human detection!

Let's be honest, we all have been here, maybe a weekend project of our own, our personal blog, or a very early stage startup or project. In each case, it is either lack of serious impact on audience, or the fact that someone is constantly fideling with the service (or both) that drives no alert mode. It works up to some extent, as far as the cost of setting up or using a service for monitoring and alerting is greater than any loss of profit from degraded services.

This has it's own benefits, no overhead of implementing and maintaing monitoring and alerting system, even if as instrumentaiton of SaaS provider. The other benefit is low to no false positives. The downsides can be a background level of stress for operators of the system and a contsant inner-nag to periodically check if everythin works. This consumes precious human cognitive energy and imposes unnecessary stress, more so when the service attracts more customers and downtimes begin to matter more.

When is it time to move to next level?


### L1: Alerting on System Metrics (resources saturation)

Why? if you only have one or few hosts, these are usually correlated with service quality and are often easier to monitor
When? start having redundant components, when saturation on a single host wouldn't affect the whole system

### L2: Alerting on Service Metrics (api error/latency)

Why? when the services and user journeys are simple and you have some basic observability going, this is the easiest to do.
When? when user journeys start to get more complex and involve multiple calls to different endpoints.

### L3: Alerting on SLI

Why? you are at the start of the SRE journey, getting familar with the landscape and just started defining user journeys and focusing on customer happiness as the key metric for service health. this can provide a smoother transition to the next level.
When? when you feel confident enough with you SLI capturing customer happiness, and start to get sick of alerting noise.

### L4: Alerting on Remaining Error Budget

Why? It is not trigger happy, meaning transiet issues won't wake you up. 
When? 

### L5: Alerting on Multi-window Multi-burn-rate Error Budget

Why? you don't want to let the budget burn through to take action, you want to get alerted if it is burning too fast, but also recover quickly
When? mature service, high volume of traffic round the clock

## Bonus level: proper context with alerts

* description
* links
* screenshot of the graph
* region/account

## Common scenarios

* being at SLI level, but alerting based on system -> lots of false positives, one node high CPU may not affect users noticably
