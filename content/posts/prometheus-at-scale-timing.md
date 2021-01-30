---
title: "Prometheus at Scale with Thanos: Timing Parameters"
date: 2021-01-30T18:32:02+01:00
draft: true
---

The story begins a few weeks or months after you have had the epiphany to scale out your Prometheus setup with Thanos. Flashback to before that, you were probably struggling with long term storage (Prometheus gets chonky with long retentions) and cross region or cluster queries to provide a unified view. That setups runs smooth for a while with default parameters and you can shovel more resources from time to time to improve performance. Such beautiful and simple times will soon be gone. One day you wake up to the reality that some heavy queries are taking most of the resources and grinding the whole system to a halt under load. This is about timing parameters in those components that help to make the system more robust.

The way these issues manifest is often through slowness and unresponsiveness of the whole system[^1], which eventually can recover on their own but take long. But why this happens? two things are inevitable: users and Kubernetes, and maybe taxes, yes taxes for sure. Users of your system might hate you, themselves or the whole system and put a query like `histogram_quantile(0.99,sum(rate(some_high_cardinality_metric[28d])))` in a Grafana dashboard with `5s` refresh interval just for the heck of it. K8S on the other hand is agent of chaos, your services can and will get rescheduled for variety of reasons, specially if you are cost conscious and opt to use cheaper machines like [GCP preemptible](https://cloud.google.com/compute/docs/instances/preemptible) or [AWS Spot](https://aws.amazon.com/ec2/spot/) for your workloads. Thanos components [rely on each other](https://thanos.io/tip/thanos/service-discovery.md/#service-discovery) and wait for some time to drop an unresponsive peer out of their list. Combine all of these and you get the picture: heavy queries or chaos kills a component, other components still wait for it for a long time before giving up and hence slow down the whole system. What can you do? minimize the blast radius of course.

## The Parameters

For a quick recap, in a [multi-cluster setup](https://banzaicloud.com/blog/multi-cluster-monitoring/#metric-query-flow), you would be having at least these set of components: 
* Thanos Sidecar: to upload TSDB blocks and also respond to queries for fresh[^2] data
* Thanos Store: to serve the previously uploaded TSDB blocks 
* Thanos Query[^3]: To connect these pieces together, do the deduplication you were promised and provide a single endpoint for users

### Query Timeout
This is the simplest, amount of time a query can be in flight before it is aborted (with error in various layers). This is not set by default, meaning there is no limit so a heavy query could clog the system for a long time. By setting this in multiple layers (i.e global query and local query) you can have better control and hard limits against system wrecking queries.

This applies to [query](https://thanos.io/tip/components/query.md/#flags) component as `--query.timeout` CLI flag.

### Store Response Timeout
This can be interpreted as the equivalent of [time-to-first-byte](https://en.wikipedia.org/wiki/Time_to_first_byte) in Thanos world. Store components only have one key job to do: find data blocks that match the series select criteria and start streaming their content. This should be fairly fast or at least start fairly fast. By setting this value we are excluding the instances that take long to send data, which usually means they are dead, dying, or bogged down under another heavy query. Since we have at least two of any store instances (be it sidecar or Thanos store) this is fine. Even if both instances hit this timeout at least we fail fast and free up the hot path for other (hopefully) non-failing queries.

This applies to [query](https://thanos.io/tip/components/query.md/#flags) component as `--store.response-timeout` CLI flag.

### gRPC Grace Period
This is the period that the gRPC server continues to listen when receives an interrupt. In K8S this happens when a pod is being evicted.

This applies to [query](https://thanos.io/tip/components/query.md/#flags), [store](https://thanos.io/tip/components/store.md/#flags) and [sidecar](https://thanos.io/tip/components/sidecar.md/#flags) components as `--grpc-grace-period` CLI flag.

### Series Sample Limit
This flag on Thanos store limits the number of samples that a whole query (including subqueries) will touch. The number of samples can increase through cardinality and range of time. So a low cardinality metric for `60d` might touch a lot of samples, the same as a high cardinality metrics over `1d` might. Why would this matter? because these are often heavy queries that are in dire need of optimization, either by rewriting the query expression or changing the metric from the source.

This applies to [store](https://thanos.io/tip/components/store.md/#flags) component as `--store.grpc.series-sample-limit` CLI flag.

## The Reasoning

We are addressing two problems:
* Few heavy queries can take all of the system resources, not leaving any room for lighter ones
* Chaotic environment, as we always have some components dying (usual restart or reschedules) across clusters and lack of tighter timeouts and deadlines which gets especially bold in the fan-out model[^4] of Thanos query

We need some solutions for both:
* In lieu of proper prioritization mechanism in this stack, limit the impact of heavy queries
* Improve the responsiveness of the whole system when (redundant) component unavailability events happen

The only levers we have to limit queries are query timeout and sample limit. Query timeout can cap the duration that the said query will keep the system engaged and the sample limit can stop the processing when there are too many samples to return Tightening these parameters will lead to the heaviest queries to fail most -if not all- of the time, but also make the system more resilient as a result. Finding proper values for each required a series of experiments with trial and error approach.

For the chaotic part, the faster we let go of the fallen ~~comrades~~ components in the path, the better. Since there is usually a good amount of redundancy in place and the requests are fanned-out. So one failing component shouldn’t drive the latency of the response. In the case of non-failing but slow component (usually stores) if the answer isn’t being streamed within a short time, there is a good chance that it never will be (in time), so again cutting early leads to better performance and at worst results in failing fast vs failing slow. By setting tight grace periods and response time we minimize the effect of any slow or failing component on global query evaluation.

## The Recommendation

Of course like many things that you read over the Internet, you shouldn't take them as is, these are just recommendation. After reading this you hopefully understand them and will tune them best to your environment and use case.

Sidecar 
  * `--grpc-grace-period=5s`

Store
  * `--store.grpc.series-sample-limit=50000`
  * `--grpc-grace-period=5s`

Local Query
  * `--query.timeout=2m`
  * `--store.response-timeout=10s`
  * `--grpc-grace-period=5s`

Global Query
  * `--query.timeout=5m`
  * `--store.response-timeout=10s`
  * `--grpc-grace-period=5s`

[^1]: This can be seen if you graph query timing of your global view query or rule evaluation time of your Thanos Rule
[^2]: Fresh means whatever your Prometheus retention is set to. Minimum is usually two hours.
[^3]: Most probably multiple layers of query, for local views and the global view.
[^4]: This depends on partial-response strategy of the asking component, but this often applies: https://paulcavallaro.com/blog/fanouts-and-percentiles/