---
title: "Aggregating SLIs"
date: 2024-08-28T12:13:14+00:00
tags: [sre,sli]
categories: [technology]
---
With a great number of SLIs come great ~~responsibility~~ aggregation issues. While aggregation provides more readability in overviews, it also inadvertently hides nuances and details. Our aim should be to propose a sensible aggregation method that ensures we extract the most valuable insights, with a promise to review it often in order to keep it fit.

<!--more-->

## When do you need to aggregate?

Leaders managing large organisations, which often include many services and lots of SLIs, usually ask for overview dashboards and reports. Dashboards can give a sense of how systems are doing at a glance, and reports can show how systems are doing overtime. Is some service (as a whole) having subpar reliability month over month? Are all indicators green while we are getting public complaints over social media? Answering these questions with some rough confidence requires single (or very few) numbers, not all the detailed SLIs of the underlying services. To get to such an overview, we might need to aggregate SLIs per service, per product, or sometimes at the organization level. The leadership asks are valid concerns with high-level aims of resource allocation and planning; how we respond to that ask can make a lot of difference in what view we provide.

## What are some aggregation options for SLIs?

This is a non-exhaustive list to show different ways of aggregating indicators to get a single number. There is no one correct way of aggregation in each case; like any form of aggregation, we lose some signal in the process, and thus we need to find the method that keeps the most amount of signal relevant to us per case while keeping it simple.

Remember that SLIs are indicators, and as such, they are already the result of some formula for aggregation, often in the form of ratios. At the same time, they are far simpler than [composite indicators](https://link.springer.com/referenceworkentry/10.1007/978-3-642-04898-2_15), where many indicators are combined to form a new index, such as GDP or CPI. So they can’t be treated just like mere numbers, but they are much easier to deal with than their more complex cousins.

### Summing events

This means taking each data point (e.g., HTTP requests) and summing them for numerator (successful requests) and denominator ([valid requests](https://blog.alexewerlof.com/p/valid-vs-total)). This gives more weight to individual events (e.g. each request) than each different SLI. In general, this is only advised if different SLIs have the same base rate, so one SLI with higher traffic won't overshadow a lower-traffic one. If the scale of SLIs being aggregated varies widely (e.g., one has 10 RPs and another 100 RPs), you better use other methods; the alternative is to [normalise](https://en.wikipedia.org/wiki/Normalization_\(statistics\)) the numbers.

> 💡 One could argue if some SLIs can be aggregated in this way, aren’t they basically just one SLI to begin with? while this is theoretically correct, in practice these SLIs might fall into boundaries of different teams or services, so it could be beneficial to have them separate, and aggregate them like this, instead of sharing one SLI across team and diluting the ownership.

### Weighted average

This is about averaging the percentages, first simply by assuming equal weight for all. Equal weights would put more emphasis on each defined SLI rather than each event. Calculation is done by summing each indicator (percentage) and dividing by count of indicators. A more sophisticated approach is needed if SLIs differ widely in their impact; for that, we assign weights to individual SLIs based on their impact and calculate a weighted average.

> 💡 An obvious shortcoming of this method is what statisticians call “compensability among indicators”, meaning that with averages higher indicators can compensate for lower ones and hiding flaws. Geometric average is what is often used as an easy remedy to prevent compensability, but I felt it is too complex for SLIs to include here.

### Percentiles

Instead of averages, use percentiles (e.g., p50, p95) to account for potential outliers that could skew the results. This makes more sense for visualisations such as [box plots](https://en.wikipedia.org/wiki/Box_plot) to show the range of values in a population that can also be compared with relative ease.

### Count of SLO Compliance

Count the number of services meeting their SLO targets to assess the overall reliability of the organisation, simple shown as `{count of compliant SLOs} / {total SLOs}` figure over a table, ideally with a simple `↘/-/↗` to show change over the past period. This is what Google also suggests in [their SRE workbook](https://sre.google/workbook/implementing-slos/#slo-compliance-report).

## Start simple, Iterate regularly

Start simple and iterate regularly to keep it up to date with reality. Your rate of iteration should match rate of change in your SLIs or the needs of the audience. Once or twice a year is often enough for such entities with low rate of change.

On each iteration, you should look at two broad areas: your SLIs and your audience. You looked at your SLIs on the first run and decided upon a categorisation (e.g. per service) and an aggregation method (e.g. equally weighted average). Are those choices still valid? or has the reality changed and now some of the SLIs have different scale of traffic? or part of the bigger services is chipped away into new services? Next area is the audience and more importantly, their needs, Has the audience changed? was it initially just for use of CTO but now all VPs and EMs rely on the aggregate for different use-case? Did it start as a steering indicator for non-functional vs feature development, but now it is tied to annual bonuses? Reevaluating the way the aggregated figures are being used can inform decision on aggregation method.

> 💡 One important signal that your aggregated figure is in need of an update is when a clear discrepancy is observed. This can show as indicator showing all systems green while there is a noticeable and damaging negative impact on services, or the converse.

## What about a target (SLO) for now aggregated SLIs?

SLIs are useful indicators on their own<sup>[citation needed]</sup>, especially if we look at their trend over time, like month over month; as such, there is no immediate need for a target when we aggregate SLIs into one indicator. Many times we might want to even resist the urge to set a target for such aggregated SLI to avoid [Goodhart’s trap](https://en.wikipedia.org/wiki/Goodhart%27s_law). But if we absolutely must, the simpler way is to set a new target for the aggregate SLI, using historical data and business needs as input. A more complicated approach can be calculating a composite SLO, beautifully described [here](https://alexewerlof.medium.com/calculating-composite-sla-d855eaf2c655) in details. This approach makes more sense for service-level aggregate SLIs than for product- or organization-level ones.


## Resources

- [Managing complexity with aggregation](https://www.coursera.org/lecture/site-reliability-engineering-slos/managing-complexity-with-aggregation-pLdiv)
- [Defining an SLO > Aggregating SLOs](https://medium.com/@asuffield/defining-an-slo-6302f60b218a#e3be)
- [Why you should be careful when averaging percentages](https://www.robertoreif.com/blog/2018/1/7/why-you-should-be-careful-when-averaging-percentages)
- [Yes! You CAN Average Averages](https://weallcount.com/2020/11/02/yes-you-can-average-averages/)
- [Methods For Constructing Composite Indices: One For All Or All For One? (PDF)](https://www.istat.it/it/files/2013/12/Rivista2013_Mazziotta_Pareto.pdf)
- [Calculating a composite SLO](https://blog.alexewerlof.com/p/composite-slo)
