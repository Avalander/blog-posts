---
title: A guide to better estimates
published: false
description:
tags: estimates, agile, productivity
---

# Why estimate?

I've met many developers who thought that estimates were a minor nuisance in the best case and a tyranic abuse strategy used by managers in the worst case. These concepcions are usually the result of bad management practices, where estimates are misused, cases may range from bureocracy (e.g., we estimate because our Agile book says so) to control and abuse tactics (e.g., estimates are considered commitments).

Whatever the reason, the fact is that there are many developers out there that don't know what's the point of estimates at all, and it's not their fault.

The purpose of estimates is to **add predictability** to the substantially unpredictable business of software development. There are many reasons why having a predictable process (even if it's only a medium degree of predictability) is better than having an unpredictable process, but I won't discuss them in this article.

Some people argue the value of estimates is the discussion that they trigger, I'll comment on that before this article is over.

## How do estimates add predictability?

Let's look at a concrete example. Imagine that our team is developing the company's internal ERP system. This is our team's backlog.

| Backlog item                              |
| ----------------------------------------- |
| Handle multiple providers for inventory   |
| Billing available in USD                  |
| Create time-bounded discounts             |
| Time-bounded discounts available globally |
| Time-bounded discounts available locally  |
| Billing available in EUR                  |

The backlog is prioritised and we assume that the backlog for the team's current iteration is closed and we can't change that. Now, imagine that Chad, from finances, approaches Sarah, the product owner, and asks when billing in EUR will be available. With the information we have so far, the only sincere answer that Sarah can provide is "I don't know".

What if our backlog was estimated?

| Backlog item                              | Estimate |
| ----------------------------------------- |:--------:|
| Handle multiple providers for inventory   | 8        |
| Billing available in USD                  | 5        |
| Create time-bounded discounts             | 13       |
| Time-bounded discounts available globally | 5        |
| Billing available in EUR                  | 8        |
| Time-bounded discounts available locally  | 3        |

Let's assume that our team has been developing the ERP system for at least a few iterations and we know that, on average, we deliver 14 story points on each iteration.

Now Sarah the product owner could answer something like this: "well, let's see. Next iteration we'll take multiple providers and billing in USD. The iteration after that we'll be busy with time-bounded discounts. And the iteration after we can take on global time-bounded discounts and billing in EUR, so hopefully billing in EUR should be available in three iterations from now."

All of a sudden, Sarah the product owner can provide reasoned answers to our stakeholders and Chad from finances has a rough idea of when his feature will be available.

What if we had even more data? Let's pretend that we have tracked the accuracy of our estimates and we know that we estimate right 80% of the time (far better than any team I've worked with, but since we're pretending, let's pretend big). That means that one out of five user stories will have a wrong estimate.

With that information, Sarah the product owner could provide even more information to Chad from finances: "however, looking at how often we've estimated wrong in the past, it's likely that one of this stories takes more time than we expected and we can't deliver billing in EUR until four iterations from now". How does that sound? Now Chad doesn't only know when the team *estimates* to deliver the feature, but also that there is a margin of error and what delay such margin could cause.

## Do estimates add value by triggering discussions?

In your standard estimation meeting, the product owner will present a user story to the team, the team will ask for clarifications, and then each member will either choose a card from an estimating poker deck, raise a hand with a few fingers stretched, say a number or something similar. Sometimes different people will have different estimates. Then the facilitator will jump in and ask people what was their reasoning when they estimated the story to this or that value. The team will start discussing and they will realise that some might not have foreseen part of the complexity, or might have missed an easy solution. Whatever the final estimate is, the important thing is that the team will have discussed the story in more detail and will have a common idea of how to proceed.

While this is true in most cases, I would say that the discussions are a *byproduct* of estimates, not a direct result. If you are estimating for the sake of the discussion, you don't need to keep record of your estimates. Just ask the team to estimate each story, have the discussion, and throw the estimates away. Even better, figure a better way to have detailed discussions and throw those poker decks into the recycle bin. Extra points if the new method also triggers a discussion when everybody in the team would have estimated the story with the same value.

# How to estimate right?

## Estimates are not commitments

## Update your estimates

## Learn from reality

## Base estimates on previous experience