---
title: "When to alert on what?"
date: 2023-01-14T09:10:11+02:00
tags: [sre,alerting]
categories: [technology]
---

We have all been there when we go into a new company or team, looking at their stack and especially alerting, and something feels a bit ... off? This is an honest attempt to categorize alerting into some bands or levels, to make it easier to reason about where things are and have a glimpse at the road ahead. Or in other words, to put words on those vague feelings. The levels aren't meant to be followed in sequence, some can be skipped, others not. Moreover, this is by no means a yet-another-maturity-model[^1], use it as such at your own risk. The end goal of any alert is (besides being actionable) to detect problems affecting your users fast enough, without being too trigger-happy<sup>[citation needed]</sup>.

<!--more-->

If you are confused about how this can help you, you can start by finding the level that describes your current state of alerting, then see if the service matches the level described, if not, look around to find out whether it is behind, or there is some pre-mature engineering going on.

## L0: No alerts, Only human detection!

> tl;dr when there is little to no user impact from downtime.

Let's be honest, we all have been here, maybe a weekend-project of our own, our blog, or a very early-stage startup or project. In each case, it is either lack of serious impact on the audience or the fact that someone is constantly fiddling with the service (or both) that drives no alert mode. It works up to some extent, as far as the cost of setting up or using a service for monitoring and alerting is greater than any loss of profit from degraded services.

This has its benefits, no overhead of implementing and maintaining a monitoring and alerting system, even if as easy as the instrumentation of a SaaS provider. The other benefit is little to no false positives. The downsides can be a background level of stress for operators of the system and a constant inner nag to periodically check if everything works. This consumes precious human cognitive energy and imposes unnecessary stress, more so when the service attracts more customers and downtimes begin to matter more.

## L1: Alerting on System Metrics

> tl;dr when service runs on one or two dedicated nodes.

System metrics here refer to basic operating system resources, like CPU, memory, disk, and network usage. Load average in *nix systems is also a viable source[^2]. Monitoring them and alerting against them is often easy, there are a lot of single-binary agents to collect these, and understanding them isn't that difficult, as you would be mostly dealing with percentages.

Sometimes system metrics and alerting based on them are seen as this legend of ancient times when baremetals and servers as pets were the thing. They can still make sense if you only have one or a few hosts running your services, then system metrics can correlate well with service quality (in cases other than logical issues) and are much easier to monitor. But once you have some redundancy in your system, when saturation of one host's resources doesn't immediately impact user experience, this mode stops being useful and quickly becomes noisy.

## L2: Alerting on Service Metrics

> tl;dr when service has some observability, and serves only a handful of endpoints.

Monitoring a service's outward interactions is what is often meant by Service Metrics, for APIs this would be HTTP response code and response latency. This is already a few steps close to user experience, and it takes the degradation of dependencies into account. These are best observed from a load-balancer point of view and aggregated across all (identical) instances serving traffic. You might choose to group by HTTP verb or path, but that's often the extent to which they get detailed.

Once your service has some instrumentation and basic observability going on, this would be an obvious choice. This fits the most when your service has few endpoints and only one or two user journeys. Once user journeys start to get more complex and involve multiple calls to different endpoints this stops being useful, for example traffic of some endpoints might hide some issues in other lower-traffic endpoints.

## L3: Alerting on SLI

> tl;dr when service is on the path to scale, and user journeys are a thing!

At some point in time, you might start the SRE journey, and defining SLIs is one of the first steps. You are getting familiar with the landscape and just started defining user journeys and focusing on customer happiness as the key metric for service reliability. You end up with a few SLIs for your service and alerting against that with a fixed (percentage) threshold can provide value. It is not the best of the SRE world yet, as it can be noisy at times, but this will provide a smoother transition to the next level.

This makes sense when your service has found product-market-fit, has a stable set of user journeys, and is out of the valley of death[^3], so it is there to stay, and is beginning to scale. This is when you feel the need to prioritize between different user journeys (that translates into a different set of endpoints on your API perhaps), instead of blanket treating all requests to your service the same.

## L4: Alerting on remaining Error Budget

> tl;dr when SLI and SLO are set for the service, and an Error Budget Policy document exists.

Once you continue in the SRE journey, you will inevitably get to Error Budgets[^4], in short, it describes how much an SLI can fail (be below SLO) and still be considered fine. This is a step extra on top of alerting on SLI since you don't care about every time your SLI dropped below a threshold, what you care about is how much of the budget still remains. In this sense, a low but constant rate of error won't trigger an alert, as long as it is within budget. This results in a much less trigger-happy alerting condition<sup>[citation needed]</sup>.

Having error budget alerting makes sense when you have SLIs and SLOs established for your service, and also have an error budget policy in place. The first part is about getting comfortable with the process and terminology without overburdening the team. And having an error budget policy means all stakeholders have agreed on the consequences of a depleted budget. You could skip the previous level and once you have your SLI and SLO jump into alerting based on EB, but I advise you to do so only if it is not the first time for the team.

## L5: Alerting on Multi-window Multi-burn-rate Error Budget

> tl;dr when you don't have much better things to do, or when alerting is inefficient.

Just when you thought you are at the peak of alerting hierarchy, yet another one hits you! You often don't want to let the budget burn through to take action, you want to get alerted if it is burning too fast but also recover the alert state quickly once the burning stops. This is where multi-window, multi-burn-rate alerting against the rate of error budget burn comes into the picture. It is described in good detail [here](https://sre.google/workbook/alerting-on-slos/#6-multiwindow-multi-burn-rate-alerts), so I won't go through the whys and hows here.

This level of alerting makes sense for a service that is mature enough feature-wise, has its architecture fit for SLO[^5] and receives a high volume of traffic around the clock. It also requires good knowledge of alerting and an SRE mindset within the team to create and maintain such alerts (yes the expressions can get ... ugly). This is, if the previous criteria match, the height of alerting for a service in my opinion. As it has the least amount of noise, is often pretty actionable, and doesn't hide away re-occurring issues.

[^1]: Nothing against them, they can be pretty useful when utilized correctly: https://martinfowler.com/bliki/MaturityModel.html
[^2]: Can be tricky to reason about though: https://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html
[^3]: I spent 5 minutes looking for a good source to explain this without all the VC bullshit, but didn't succeed, Google it yourself and pick one that suits you!
[^4]: Mor on that in Google's SRE book: https://sre.google/sre-book/embracing-risk/#id-na2u1S2SKi1-marker
[^5]: The more 9s the more redundancy, right?