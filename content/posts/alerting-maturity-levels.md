---
title: "Alerting Maturity Levels"
date: 2022-07-18T09:07:34+02:00
draft: true
---



<!--more-->

## Levels 0 to 5

* This is not to label things (like old school maturity models), but to map each team/company where in journey is standing.
* Some of the levels can be skipped some can not, there is no inherent supriority to higher levels, they are necessitated by their more complex environment.
* End goal of any good alert is to detect problems fast enough, without being trigger happy!
* Per each level: what it means, why it works?, when to move?

### L0: No alerts, Only human detection!

Let's be honest, we all have been here, maybe a weekend project of our own, our personal blog, or a very early stage startup or project. In each case, it is either lack of serious impact on audience, or the fact that someone is constantly fideling with the service (or both) that drives no alert mode. It works up to some extent, as far as the cost of setting up or using a service for monitoring and alerting is greater than any loss of profit from degraded services.

This has it's own benefits, no overhead of implementing and maintaing monitoring and alerting system, even if as instrumentaiton of SaaS provider. The other benefit is low to no false positives. The downsides can be a background level of stress for operators of the system and a contsant inner-nag to periodically check if everythin works. This consumes precious human cogniive energy and imposes unnecessary stress, more so when the service attrcts more customers and downtime begin to matter more.

When is it time to move to next level?


### L1: Alerting on System Metrics (resources saturation)

Why? if you only have one or few hosts, these are usually correlated with service quality and are often easier to monitor
When? start having redundant components, when saturation on a sing;e host wouldn't affect the whole system

### L2: Alerting on Service Metrics (api error/latency)



### L3: Alerting on SLI


### L4: Alerting on Remaining Error Budget


### L5: Alerting on Multi-window Multi-burn-rate Erorr Budget


## Bonus level: proper context with alerts

* description
* links
* screenshot of the graph
* region/account

