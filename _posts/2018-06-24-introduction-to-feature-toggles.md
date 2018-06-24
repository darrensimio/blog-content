---
ID: 225
post_title: Introduction to Feature Toggles
author: Darren Sim
post_excerpt: ""
layout: post
permalink: >
  https://darrensim.io/posts/introduction-to-feature-toggles/
published: true
post_date: 2018-06-24 18:40:40
---
As software professionals, success is measured by our ability to improve our users’ lives.

Unlike other engineering fields which have been around for hundreds of years, the software engineering discipline has existed for barely a hundred years. As an industry, software development is still largely part-art, part-science.

With the ever evolving business and technological landscape, many of our past experiences serve merely as references. Adding to the unknowns, users seldom know what features they want, but they know what they don’t want. More often than not, it takes several attempts of trial-and-error before software teams become successful in delivering features that stick.

The ability to continuously deliver working software into the hands of customers, has served as a competitive advantage for leading tech companies like Amazon and Facebook. Not only did this capability allow engineering teams to deploy code into production in small chunks, quickly, and safely, it also shortened the feedback loop - allowing teams to learn fast and respond to changes quickly.

In the practice of Continuous Delivery (CD), it's important that the application is continuously integrated (CI), and deployable at all times. This can be challenging especially in situations where a feature spans multiple commits, and, an urgent fix needs to be released before the development of that feature is complete. This highlights the need for deployment and release to be decoupled.

When a feature is completed and deployed to production, teams also need a way to experiment safely, beta testing with a small group of users and being able to roll back quickly if an undesirable experience is observed.

Feature Toggles offers options to overcome to these challenges.

In this article, we will explore: <strong><em>1) what feature toggles are</em></strong>, <em><strong>2) how we can use them</strong></em>, before <em><strong>3) discussing best practices</strong></em>.