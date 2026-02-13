---
layout: post
title: "6 Best Practices for Continuous Delivery Pipelines"
date: 2018-10-17 00:00:00 +0000
---

[Derik Evangelista](http://onion.works/) and I talked about pipelines' best practices during the [Cloud Foundry summit EU 2018 in Basel (CH)](https://www.cloudfoundry.org/event_subpages/cfeu-2018-schedule/). For instance, the practices are:

1. Deploy the same way to every environment
2. Run smoke tests in all your deployments
3. Only build binaries once
4. Stop the pipeline on any failures
5. Deploy into a production-like environment
6. Changes should propagated through the pipeline immediately

The best practices above are the result of years of observations from engineers in the field, trying to solve issues that frequently arise during the software releasing process.

Watch the talk bellow if you are interested in why you should apply these practices and how to do it. If you want to go deeper into this topic, I recommend the [Continuous Delivery: Reliable Software Releases Through Build, Test, and Deployment Automation](https://www.goodreads.com/book/show/8686650-continuous-delivery) book by Jez Humble and David Farley.

{% include youtube.html id="aVyfkhSeVPo" %}
