---
title: "When to Use Prometheus Recording Rules"
date: 2020-10-20T09:03:00+02:00
draft: true
---

Overtime the use case for recording rules have changed a lot, we no longer need to use it to compensate for lack of sub-query for example. But how should they be used really? Here is my 2cents on the matter.

## Sensible Use Cases

### Complex (but reasonably fast) queries

This is primarily when you want to hide the complexity of using multiple, often nested queries to get a sensible value out. There are cases that current PromQL doesn't even allow subqueries. This especially helps for alerting expressions, you want to keep it simple and manageable for cases (like multi-thingy mentioned below). If your expression is fast enough, the extra computing resource and storage is often justified.

### Aggregation and rollup for high cardinality metrics

There are numerous cases where a service or application exposes too much data (i.e many labels) and you want to reduce that to those you need for regular use. You should beware that if you have control over these exposed metrics, it is much better to reduce cardinality at the source. There are also metric relabeling configs that can drop/merge metrics after scrape and before saving. If all those are unavailable to you, recording rules are a good alternative.

### Multi-window multi-burn-rate alerts

So you have joined the movement and identify as an SRE now, great! Those alerts need to compare and combine a lot of expressions to work. While using raw expressions is possible, it is not a good sight and soon becomes hard to read, debug, and maintain. Use recording rules for those expressions and get some peace of mind. However beware that you won’t or shouldn’t need a range greater than 1d for any expression in those rules since you would be alerting against the slope of burn rate, not the actual cumulative value.

### Because you want to and no one can tell you otherwise

Okay, whatever, but eat your veggies at least! And be prepared to see some gaps as your heavy recording rule might fail to evaluate in time.

## What to do for dearly needed heavy recording rules

* Specify steps in the [range vector](https://prometheus.io/docs/prometheus/latest/querying/basics/#range-vector-selectors) to reduce the resolution (e.g `sum(rate(http_reqs_total[10m:60s]))`)
* Specify [group interval](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#rule_group) to run them less frequently (e.g every `5m` instead of the default `20s` interval)
* Rewrite the expression: to get a close approximate

### Specify steps: reduce the resolution

There are two things almost invisible to humans in the computer age, second page of Google search results, and resolution part of a range vector in PromQL: you don’t know they exist until you need them.

### Specify group interval: run them less frequently

### Rewrite the expression: to get a close approximate

### Ask yourself if it is really needed: usually not!

