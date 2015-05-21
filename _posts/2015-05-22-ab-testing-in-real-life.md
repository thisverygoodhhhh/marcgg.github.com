---
layout: post
title: Real Life A/B Testing With Universal Analytics
description: Blah
blog: true
category: blog
tag: Performance
---

I've been enthusiastic about the concept of A/B testing for a very long time, but I only got to implement it for the first time about two years ago. At the time I thought it would be easy. So many companies are talking about it, there must be a clear path to do it, right?

Short answer: no.

Long answer: no, unless your website is a landing page.

_**tl;dr:** To do effective real life A/B testing, you need clearly identified KPIs, a few lines of code, enough volume and someone analysing the results. It is harder than using a drop in tool usable by anyone, but it is way more powerful and still totally doable. _

## A/B Testing 101

Alright, to be fair if you want to see if putting a picture of a cat on your homepage leads to people clicking on a button, then the internet got you covered... drop some JavaScript in and voilà!

However that's a very poor definition of conversion and a very limited set of A/B tests you can run. Let me elaborate.

### What You Actually Test

In this situation, we change something very simple. A visual element on a single page. This is nice but often, if you run something more complex than a landing page, you'll want to test more.

For instance, here are some interesting things on which you'd probably like to experiment:

- Ranking algorithms
- Emails & notifications (either the volume or the content)
- Having a feature vs not having it
- A serie of related content changes on various pages
- Entirely different page or set of pages

In a lot of these cases, you need way more than just front-end changes. If your tool promised a solution usable without any code to write, that's where you reach its limit.

### How To Mesure Success

The other limit of a lot of tools is that the measure for success is sometimes simplistic. If "people clicked on this button" means to you that your website is performing well, you might be doing something wrong.

Let's see why.

Imagine you're testing a variation of a link's copy and use "Click here, it's amazing" instead of a duller wording. Because of this change, people end up clicking more on the button. 

Great! This version is better! ... OR IS IT!?!

Well... what if people click on the button and bounce right away because it's linking to a page that's really not that amazing?

What I'm getting at is that measuring conversion is hard. Often you have to take multiple variables into consideration, so "people clicked on the button" is often not a good metric to follow.

## Real Life A/B Testing

At [Drivy][1] we use A/B testing to validate some ideas. It gives us insights on what works well and what doesn't. Obviously that's not the only input we use for decision making, but it is one of them.

Instead of relying on an existing tool to do this, we decided to build our own on top of Google's Universal Analytics. We don't use an existing tool because, at the time, nothing great existed to do this, and the code needed to get it to run is very light.

<div class="image-wrapper" style="text-align: center"><img src="/assets/blog/universal.png" style="width: 300px;"/></div>


### Before Starting To Test

#### Identify Your KPIs

First of all, don't start any split testing if you don't have clear KPIs and clear objectives. For instance, if you have an online hat store, a goal could be "user buys a hat". It could also be "user signs up for the newsletter" or "user spends more than 2 minutes reading an article".

Based on this, try to define a clear conversion funnel. If you're not familiar with the concept, allow me to quote [Wikipedia][2]:

> Conversion funnel is a technical term used in e-commerce operations to describe the track a consumer takes through an Internet advertising or search system, navigating an e-commerce website and finally converting to a sale. 
> 
> The metaphor of a funnel is used to describe the decrease in numbers that occurs at each step of the process.

For instance, in the case of the hat website:

- User goes to your website (entry point)
- 1) User searches for the best hat
- 2) User clicks on the product page for a given hat
- 3) User sees the payment form
- 4) User pays and enjoys their hat

Like this, once you are A/B testing, you will be able to say "by changing X, we saw a increase in (1) -\> (2), but a decrease in (3) -\> (4) and an overall improvement on (1) -\> (4)". This way you'll get way more interesting insights.

If you decide to change the hat ranking algorithm to display only hat with HD pictures, you could get an increase in (1) -\> (2) because the search results looks prettier. However it could decrease global conversion (1) -\> (4) because even if the search results look better, users don't want to buy the hats with HD pictures. The solution here would be to add HD pictures for all hats and keep the old ranking algorithm.

#### Have Enough Volumes

First of all, don't A/B test if you don't have much traffic.

If you don't have enough people using your website, it will take a very long time to reach a good enough level of [statistical significance][3].

I know it sounds cool to be doing A/B testing, but if you're too early, it makes very little sense and it might misguide you more than anything else.

_Also if you say "we improved conversion by 2000% thanks to a/b testing" and your website is a landing page viewed by 500 visitors a months, it honnestly sounds a bit silly._

### Starting Your Experiment

Once you have your KPIs and a decent volume of users on your website, it's time to start your "experiment". 

By the way, what I'll call experiment from now on is the process of testing multiple versions/variations of a given feature. 

#### Flagging Users

Every user going to your website must be identified as either being a part of experiment "A" or experiment "B".

To implement this, use a random number generator. It is waaaay faster than any other method and will statistically works if you have enough volume.

Once you know if the user is "A" or "B", store it somewhere. You can use cookies, or, even better, somewhere that will persist across multiple devices. Of course try to avoid large datastore overheads when running an experiment as it could slow down your site.

Once we identified the user, we send the information to Universal Analytics [using custom dimensions][4].

#### Displaying The Various Versions Of The Site

Once you know that a given user is A or B, it becomes very easy to change the website based on that information. Here's a random snippet of code using Ruby, just in case that's not clear enought:

{% highlight ruby %}
 if user.experiment?(:a)
  ...
 else
 ...
 end
{% endhighlight %}

Of course, try to do it in a way that is easy to revert. At Drivy we use a simple DSL that allows us to easily search for any part of the code that is related to an experiment. Here is what it looks when used in a HAML view:

{% highlight ruby %}
-# Sets up tracking
- start_experiment(:welcome_message)

-# Displays things
= render_experiment do |e|
  = e.first_variation do
    Hello!
  = e.second_variation do
    Hello all!
  = e.original do
    Welcome.
{% endhighlight %}

_Protip: To be faster, don't try to write robust and elegant code in an experiment. Treat it as a prototype that will be thrown away at the end - and then actually throw it away and rewrite everything once the experiment is over._

#### Analysing Results

After a while, check if you reached an acceptable level of statistical significance. Don't even look at the metrics before that since they don't mean a thing yet.

(TODO: check aymeric))

Once it is reached, you can go in Universal Analytics and segment your KPIs based on the custom dimensions you set up earlier. That's where you'll see which variation performs better.

#### Close The Experiment

Once you decided which variation performs the best, close the experiment, throw away all the code related to it and write the winning variation properly.

Never leave an experiment's code in the codebase for too long once it's considered "closed" as it will add needless complexity to your codebase.

#### Checking Tests Validity

Every once in a while, don't forget to test your system by running "dummy experiments". [AirBnB goes into details about this on their blog][5], and the tl;dr; of this is that if you run an A/A test and one of the two version wins, you must be doing something wrong. 










[1]:	https://en.drivy.com
[2]:	http://en.wikipedia.org/wiki/Conversion_funnel
[3]:	http://en.wikipedia.org/wiki/Statistical_significance
[4]:	https://developers.google.com/analytics/devguides/collection/analyticsjs/custom-dims-mets
[5]:	http://nerds.airbnb.com/experiments-at-airbnb/