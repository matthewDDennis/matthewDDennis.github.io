---
title: "he Case of the Redis Client, Part 1: Puzzle Pieces"
layout: post
comments: true
excerpt_separator: <!--more-->
---

![Broken Lock](/assets/images/redis-client2/redis-puzzle.jpg)

Having collected most of the clues we need to solve the mystery of the Redis client, we can now start putting the pieces together and see how they fit. In this post, we will take a closer look at the Redis client's architecture and the role of the `RedisClient` and other classes. We will also discuss the  responsibilities and the roles of the these classes. Finally, we will discuss the `RedisClient`'s `connect` method and how it interacts with the `RedisConnection` class.

We will also create some tests to verify the behavior of the classes. These tests will help us understand the classes' responsibilities and how they interact with each other. We will also use the tests to verify the behavior classes as we evolve them.

<!--more-->
