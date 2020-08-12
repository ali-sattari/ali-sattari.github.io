---
title: "Alerting issues with Alertmanager and Prometheus"
date: 2020-08-10T17:22:35+02:00
draft: true
---

So you are runing your own monitoring stack with Prometheus and Alertmanager, and since you are someon‍‍‍e with proper reliability in mind, running at least thr Alertmanager as a cluster with 3 nodes, perfect! This works for a while perhaps with no issues, ...

# intro
## the settings that these can happen and proper context (AM cluster 3 nodes + Prometehus or Thanos Rule)
So you are running your own monitoring stack with Prometheus and Alertmanager, and since you are someone with proper reliability in mind, you have a cluster of at least three Alertmanagers, perfect! This works for a while perhaps with no issues, but entropy and chaos are alway on the hunt. It is not that no alerts come through or all instances are down, it works but with some occasional annoyances, like alerts flipping or receiving duplicate notification of a state. As you might have experienced, blackouts are relatively easy to fix, brownouts on the other hand, are often elusive and mind bendingly hard to resolve.

# issue definition
## flipping: firing and resolving 
## duplicates: receiving the same alert (state) more than once while ongoing
I will elaborate on two common issues with alerting here, namely flipping and duplicate notifications. Flipping happens when an alert goes into firiing and resolved state repeatedly in a short window of time, like triggering again right after it was resolved few minutes ago. This can be misleading, you want to rely on the alert state to some extent to triage issues or to decide whether you should get out of the bed at 3am, and when flipping, that trust is quickly eroded, along with the annoyance it causes. Duplicate notifiations are notifications delivered more than once while the situation that triggered the alert is still ongoing. They add to the noise and chaos, you might have a Slack channel for your alerts and prefer to have a clear view of last notifications in that stream, duplicates mess that up. So in a sense flipping is th emore serious of the two, but both are annoying.

# how prometheus and alertmanager work
Like a good troubleshooter that you are, you would want to find the source of the problem! But to get to the source, you need to know the path. So here goes a quick primer on how Prometheus and Alertmanager work together to deliver alerts and wake you up. Prometheus has the data (metrics and meta-metrics [link to UP] scraped from targets) and evaluates rules that are defined in the configuration [link to rules doc]. Both scraping and rule evaluation happen at regular, pre-defined intervals [ref to doc]. Most important part of an alert rule to this topic are `expr` and `for`. Prometheus evaluates the alert rule and if the `expr` returns `true` (result of a binary comparision or non-empty results) the alert goes into `pending` state for the duration defined by `for`, this is to help weed out the transient issues. When the `for` duration is passed and `expr` still evals to `true`, alert goes into `firing` state and is now sent to Alertmanager. This happens on all consequent eval intervals from this point until the `expr` no longer evals to `true`. Meanwhile there are couple of intervals and waits configured in Alertmsnager as well, such as `group_wait`, `group_interval` and `repeat_interval`. Alertmanager waits for the duration `group_wait` the first time an alert (uniqness define by `group_by`) is received to [reason here], then it waits for `group_interval` to see if any further alerts comes in with same `group_by` criteria so the can be merged and notified only once together, and finally a notification (with proper message from templates [ref to doc]) is generated, goes through routing [ref to docs] and is sent to the intended receivers [ref to doc]. Easy, huh? Remember that Prometheus continues to fire alerts toward Alertmanager on `eval_interval` while the issue persists and after the initial notification, Alertmanager sends notification only on `repeat_interval` (so that a firing alert isn't forgotten) or when it is resolved (if configured to send resolves [ref to doc]). But when is an alert resolved in Alertmanager? There is a `resolve_timeout` in the configuration, if no alert is received by Prometheus in that duration the alert is considered resolved. Prometheus can also send resolved state (can it?!)...
How can you observe this? good news is there is `ALERTS` metric exposed by Prometheus that records the state of each alert (by name) throughout the time. So you can see when it went into pending, firing and stopped firing (resolved). Bad news is this is the only thing! there are no alert specific metrics exposed ny Alertmanager, there are log entries for some actions, but they often don't match 1:1 to the flow described above.


# how to narrow down the issue
Now that you know the flow, it is easier to look for the source of the problem. First thing is to check the alert `expr` and `ALERTS{alertname="YOUR SHINY ALERT"}` when flipping happened, if both are continuos graphs with no change of state, you know the alert rule definion is fine. If however, you see a lot of pending -> firing in that window, you should look more closely at your alert definition. It can be the case that `for` is missing (no `pending` state, alert fires with first eval to true) or it is short (my advice is 3-4x of eval interval) or finally the expression is at fault and fluctuates below and above a static treshold.

## checking ALERTS metric and alert expression -> which side is flipping? Prom or AM?
### if ALERTS is constant (short pending + continuios firing) Prometheus is fine and it is and AM issue
### if not, check `for` or the expression itself (low static threshold?), 
### also rarely incomplete data can happen (e.g Thanos distributed setup)

## how to debug AM
A subtle but important point is that the way Alertmanager cluster works, it expects Prometheus to send firing alerts to all Alertmanager instances, they then gossip to sync state and deduplicate. If hoewever, Prometheus is only sending alerts to one instance (e.g they are behind an ingress or loadbalancer), the cluster doesn't get to do it's job and each instance develops a brain of it's own to some extent. So while one Alertmanager has recently received an alert from Prometheus, another one already thinks it is resolvbed (through `resolve_timout`) and sends out resolved notification, and then receives the alert again and considers it a new alert and so on. Make sure Prometheus point to all instances of the Alertmanager [doc ref].

Alertmanager in cluster mode [doc ref] uses a separate port to connect instances in order to indentify peers and communicate events. If cluster parameters aren't properly set, a quoroum can't form and deduplication and state sync won't happen. This can be observed both in logs [docs or sample] and in metrics exposed by Alertmanager: `alertmanager_cluster_members` records number of peers each instance can see, this should be equal among instances, meaning they can all see each other, so for a cluster of 3 Alertmanagers this should be 3 all the time, except briefly for restarts or reschedulings of instances. There is also `alertmanager_peer_position` that shows where each instance stands, this should also be a fixed number per instance through time except when one is rescheduled and the positions might reshuffle.

Let's say you have found and fixed the issues, now all run perfectly and faith in alerting is restored, great accomplishment. But, then, there, is, always, entropy :sad:. This can happen again, a version upgrade, change in CI/CD pipelines, IaC migration or refactoring, adding features or new integrations to the system? who knows, someday someone or something might inadverently ruin this beatiful setup. How to guard against it? changing the profession is always an option, but luckily there is also alerting! :mindblown: So meta! isn't it? Here are some alerts to detect discussed issues:



## useful alerts on Alerting!
### cluster members on instances
### alerts being received by all