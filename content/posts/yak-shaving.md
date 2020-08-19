---
title: "Axe Sharpening or Yak Shaving?"
date: 2020-08-19T08:09:10+02:00
tags: [preparation,meta]
categories: [technology]
---

We often use many tools in our everyday activities. Tools enable us to be faster and more precise while allowing us to engage issues at higher levels. Using tools we are leveraging other people’s innovation and could stand on the shoulder of giants to gaze further horizons. This is so fundamental that knowing one's toolset in and out is considered a sign of mastery or expertise in most crafts. By now you are thinking what is the catch? Tools are not always ready for our objective, we usually need to choose, gather and even tinker with the tools, and this is where the good, bad and ugly enter!

We often find ourselves in situations that require tool preparation; we buy a new laptop, we start a new project, we join a new company, etc. This triggers the need for quick decision making, what is my minimum required toolset to start with? Without conscious decision, we go head first and may spend hours and takes without reaching the ready status. In such cases deciding factors should be the project or task context, rather than our personal likings and adventurousness, right? Well, not very often!

## The Two Phenomena
In computer science literature there have been references to two phenomena that cover the pros and cons of preparation, Axe Sharpening in the book [Dreaming in Code](https://www.goodreads.com/book/show/32475.Dreaming_in_Code) and Yak Shaving in [Jargon Files](http://www.catb.org/~esr/jargon/html/Y/yak-shaving.html).

### Axe Sharpening

> Give me six hours to chop down a tree and I will spend the first four sharpening the ax. 
>
> -- Abraham Lincoln

That is to say, preparing tools is part of the job and maybe the better part of it. In this analogy, the task at hand (chopping down a tree) would be done faster, with less effort and less risk with a prepared tool (sharp ax).
Such preparations have clear indications:
* They are directly linked to the objective
* They form an exhaustive list of steps or items
* Their result often can be reused or shared
* The time spent on these activities will cut from the total time

These are examples of such activities in a software development context:
* Setting up IDE: downloading, installation, configuring language helpers, adding plugins, etc.
* Setting up the new environment: creating an staging environment for a new project
* Looking for a specialized library: e.g AWS API in a certain language or JPG Exif extractor

### Yak Shaving
> What you are doing when you're doing some stupid, fiddly little task that bears no obvious relationship to what you're supposed to be working on, but yet a chain of twelve causal relations links what you're doing to the original meta-task.
>
> -- Jeremy H. Brown

There is [a story](https://seths.blog/2005/03/dont_shave_that/) about a father that was trying to wash the family car in a Sunday afternoon. When he went to the garage he remembered the hose is broken, “no worries” he says to himself, I will borrow one from our neighbor, but then he recalls the boy was at neighbor’s house the other night and was carried back home with neighbor’s pillow and blanket, surely he should return those first before asking for something else! When he went inside to fetch the pillow and the blanket, his wife said kids were roughhousing this morning and torn the pillow on beds frame and some of the stuffing has become dirty. He quickly inspected the pillow and realized the wool inside is of a special breed of Yak living in Tibet, “very well then” he muttered, I shall book a flight to China, go to Tibet on foot, shave a yak, take the wool to processing plant, bring it back, stuff it in the pillow, take the pillow and the blanket to the neighbor, ask for their hose and have the car washed, easy!

The term “Yak Shaving” was first introduced in an [MIT mailing list](https://projects.csail.mit.edu/gsb/old-archive/gsb-archive/gsb2000-02-11.html) by Jeremy Brown and later was referenced in Eric Raymond’s Jargon Files.
This kind of preparation has indications too:
* There is little or no link between each step and the actual objective
* There is no list of items or map of the way
* They pop as we go and can’t be predicted beforehand

There are many similar stories in software development:

You decide to use a ready-made script to automate some routine task on your laptop. That script requires the latest version of `lib_xyz`, which is few versions ahead of what you currently have. You start updating packages to get the library to the latest version, and you end up updating the kernel too. But heck the VGA driver is flaky on the new kernel and you need to download and compile the latest driver from the vendor. You start installing the new driver that you realize `lib_yuw` needs updating too…

And so on:
{{< figure src="https://imgs.xkcd.com/comics/automation.png" caption="[Automation by XKCD](https://xkcd.com/1319/)" >}}

## What to do?
It is not always easy to tell whether you are sharpening the ax or shaving the yak, because there is a spectrum between these two extremes, in some situations the work you are doing isn’t directly related to the objective but needs to be dealt with to prevent further issues. Perhaps the only way to shed some light is to go systematic and ask following questions before jumping in:
* What is the end result or objective?
* What is the scope and lifetime of this project?
* What are the resource limitations? (e.g time, money, maintenance costs)
* How familiar are we with the subject?
* Can we make a list or plan with clear milestones?

You can answer those questions in our head for quick one person tasks or be more formal and write them down in a bigger group project, whatever suits you. This is one of those issues that self-awareness alone would help in making better decisions.
