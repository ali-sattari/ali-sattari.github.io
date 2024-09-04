---
title: "Alerting issues with Alertmanager"
date: 2020-08-13T09:00:00+02:00
tags: [prometheus,alertmanager,alerting]
categories: [technology]
---

Alerting is a vital part of operations, as it frees humans from watching systems and let them tend to more creative and constructive activities, until something breaks... at 3 am, but then resolves in few minutes, then fires again 10 minutes later, and that's when you know alerting isn't working properly!

<!--more-->

You are running your monitoring stack with Prometheus and Alertmanager, and since you are someone with proper reliability in mind, you have a cluster of at least three Alertmanagers, perfect! Perhaps this works for a while with no issues, but entropy and chaos are always on the hunt. It is not that no alerts come through or all instances are down, it works but with some occasional annoyances: alerts flapping or receiving duplicate notification of a state. As you might have experienced, blackouts are relatively easy to fix, brownouts, on the other hand, are often elusive and hard to resolve.

Let's define the issues. flapping happens when an alert goes into firing and resolved state repeatedly in a short window of time, like triggering again right after it was resolved a few minutes ago. This can be misleading, you want to rely on the alert state to some extent to triage issues or to decide whether you should get out of the bed at 3 am, and when alerts flap, that trust is quickly eroded. Duplicate notifications are notifications delivered more than once while the situation that triggered the alert is still ongoing. They add to the noise and chaos, you might have a Slack channel for your alerts and prefer to have a clear view of last notifications in that stream, duplicates mess that up. So in a sense flapping is the more serious of the two, but both are annoying.

## Quick primer on how alerting works in Prometheus

Like a good troubleshooter that you are, you would want to find the source of the problem! But to get to the source, you need to know the path.

- Prometheus has the data (metrics and meta-metrics) and evaluates rules that are defined in the [configuration](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/).
- Both scraping and rule evaluation happens at regular, pre-defined intervals.
- Prometheus evaluates the alert rule and if the `expr` returns `true` the alert goes into `pending` state for the duration defined by `for`, this is to help weed out the transient issues.
- When the `for` duration is passed and `expr` still evals to `true`, the alert goes into `firing` state and is now sent to Alertmanager. This happens on all consequent eval intervals from this point until the `expr` no longer evals to `true`.
- Meanwhile, there are a couple of intervals and waits configured in Alertmsnager as well, such as `group_interval` and `repeat_interval`. Alertmanager waits for `group_interval` to see if any further alerts come in with the same `group_by` criteria so they can be merged, and finally, a notification is generated, goes through [routing](https://prometheus.io/docs/alerting/latest/configuration/#route) and is sent to the intended [receivers](https://prometheus.io/docs/alerting/latest/configuration/#receiver).
- Remember that Prometheus continues to fire alerts toward Alertmanager on `eval_interval` while the issue persists and after the initial notification, Alertmanager sends notification only on `repeat_interval` (so that a firing alert isn't forgotten) or when it is resolved. But when is an alert resolved in Alertmanager? There is a `resolve_timeout` in the configuration, if no alert is received by Prometheus in that duration the alert is considered resolved. Prometheus can also send resolved state (by setting the EndsAt timestamp).

How can you observe this process? good news is there is `ALERTS` metric exposed by Prometheus that records the state of each alert. So you can see when it went into pending, firing, and stopped firing (resolved). The bad news is this is the only thing! there are no alert specific metrics exposed by Alertmanager, there are log entries for some actions, but they often don't match 1:1 to the flow described above.

## How to find the source?

Now that you know the flow, it is easier to look for the source of the problem. First thing is to check the alert `expr` and `ALERTS{alertname="YOUR_SHINY_ALERT"}` when flapping happens, if both are continuous graphs with no change of state, you know the alert rule definition is fine. If however, you see a lot of pending -&gt; firing in that window, you should look more closely at your alert definition. It can be the case that `for` is missing, which means no `pending` state, alert fires with the first eval to true. Or maybe `for` is too short or finally, the expression is at fault and fluctuates below and above a static threshold.

A subtle but important point is that the way Alertmanager cluster works, it expects Prometheus to send firing alerts to all Alertmanager instances. If however, Prometheus is only sending alerts to one instance (e.g they are behind an ingress or loadbalancer), the cluster doesn't get to do its job and each instance develops a brain of its own to some extent. So while one Alertmanager has recently received an alert from Prometheus, another one already thinks it is resolved (through `resolve_timout`) and sends out a resolved notification, and then receives the alert again and considers it a new alert and so on. Make sure Prometheus [points](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#alertmanager_config) to all instances of the Alertmanager by static config or proper service discovery.

Alertmanager in [cluster mode](https://github.com/prometheus/alertmanager#high-availability) uses a separate port to connect instances to identify peers and communicate events. If `--cluster.*` parameters aren't properly set, a quorum can't form, and deduplication and state sync won't happen. This can be observed both in logs and in metrics exposed by Alertmanager:

- `alertmanager_cluster_members`records the number of peers each instance can see; this should be equal among instances, meaning they can all see each other, so for a cluster of 3 Alertmanagers this should be 3 all the time, except briefly for restarts or reschedulings of instances.
- `alertmanager_peer_position` shows where each instance stands; this should also be a fixed number per instance through time, except when one is rescheduled and the positions might reshuffle.

## Useful alerts on alerting!

Let's say you have found and fixed the issues; now all runs perfectly and faith in alerting is restored, a great accomplishment. But then there is always entropy! This can happen again: a version upgrade, change in CI/CD pipelines, IaC refactoring, adding features, or new integrations to the system? who knows, someday someone or something might inadvertently ruin this beautiful setup. How to guard against it? changing the profession is always an option, but luckily there is also alerting! so meta! isn't it?

### Changes in cluster members

If this holds true for \~10m, you have a problem in your Alertmanager cluster.

```SQL
avg(alertmanager_cluster_members) by (your_cluster)
!=
max(alertmanager_cluster_members) by (your_cluster)
```

### Alerts not being sent to all instances

This marks deviations more than 5% in the number of alerts received by each instance compared to the cluster average; if this persists more than \~10m, alerts are not being sent to all Alertmanager instances.

```SQL
abs(
  rate(alertmanager_alerts_received_total{status="firing"}[10m])
  - on (your_cluster) group_left()
  avg(rate(alertmanager_alerts_received_total{status="firing"}[10m])) by (your_cluster)
)
/
rate(alertmanager_alerts_received_total{status="firing"}[10m])
* 100 > 5
```
