---
title: "When to Use Prometheus Recording Rules"
date: 2020-10-20T09:00:00+02:00
draft: true
---

Overtime the use case for recording rules have changed a lot, we no longer need to use it to compensate for lack of sub-query for example. But how should they be used really? Here is my 2cents on the matter.

## Sensible Use Cases

### Complex (but reasonably fast) queries

This is primarily when you want to hide the complexity of using multiple, often nested queries to get a sensible value out. There are cases that current PromQL doesn't even allow subqueries. This especially helps for alerting expressions, you want to keep it simple and manageable for cases (like multi-thingy mentioned below). If your expression is fast enough, the extra computing resource and storage is often justified.

### Aggregation and rollup for high cardinality metrics

There are numerous cases where a service or application exposes too much data (i.e many labels) and you want to reduce that to those you need for regular use. You should beware that if you have control over these exposed metrics, it is much better to reduce cardinality at the source. There are also [metric relabeling configs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#metric_relabel_configs) that can drop/merge metrics after scrape and before saving. If all those are unavailable to you, recording rules are a good alternative.

### Multi-window multi-burn-rate alerts

So you have joined the movement and identify as an SRE now, great![^2] Those alerts need to compare and combine a lot of expressions to work. While using raw expressions is possible, it is not a good sight and soon becomes hard to read, debug, and maintain. Use recording rules for those expressions and get some peace of mind. However beware that you won’t or shouldn’t need a range greater than `3d` for any expression in those rules since you would be alerting against the slope of burn rate, not the actual cumulative value. This might be less of a use case if you use tools like [Sloth](https://github.com/slok/sloth).

## What to do for dearly needed heavy recording rules

### Specify steps: reduce the resolution

There are two things almost invisible to humans in the computer age, the second page of Google search results, and resolution part of a [range vector](https://prometheus.io/docs/prometheus/latest/querying/basics/#range-vector-selectors) in PromQL: you don’t know they exist until you need them. The second and optional part of any range vector in PromQL is the resolution that the data are picked for calculation[^1], and defaults to scrape interval when you don't specify. Often times in big ranges you don't need the actual resolution for queries (like rate of something over a day or more!), so by setting a larger value for resolution, you allow Prometheus to jump over many data points and fetch less raw data to do calculation. You might loose some precision with this, but you gain more in performance. So in `sum(rate(http_reqs_total[1d:5m]))`, the part `[1d:5m]` is our range vector and `5m` is our custom set resolution for this expression. To keep your sanity, set the same resolution for all ranges in a single huge multi-function query, or not and go insane, your choice!

### Specify group interval: run them less frequently

Prometheus allows setting custom [group interval](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#rule_group) for rules (both recording and alerts) to override the global eval interval. This allows you to run heavy rules less frequently (e.g every `5m` instead of the global `20s` interval). Beware that setting this above `5m` could result in some amusing situations when graphing range queries, due to [staleness](https://prometheus.io/docs/prometheus/latest/querying/basics/#staleness) and stuff.

### Rewrite the expression: to get a close approximate

Yes, we are all familiar with this kind of compromise, opting for the second best thing we can get for a better performance. In prometheus world this often comes down to dealing with summaries and histograms, as `histogram_quantile` is a heavy function and you could gain a lot of performance if you opt not to use them for long ranges.


[^1]: This is documented under [sub queries section](https://prometheus.io/docs/prometheus/latest/querying/basics/#subquery) in official

[^2]: Don't worry if you haven't, [here](https://sre.google/workbook/alerting-on-slos/) is a reference to familiarize yourself with the concept.
