---
title: "The Case of the Redis Client, Part 1: Collecting Clues"
layout: post
excerpt_separator: <!--more-->
---

![Redis Client](https://www.code-sleuth.com/assets/images/redis-client1/redis-client.jpg)

The Case of the Redis Client is a series of posts that will explore the design and implementation of a Redis client from scratch. In this first post, we will collect the clues and information needed to solve the case, and set the stage for the investigation. This is one of my oldest 'Code Cases'.

Over the years, when trying to learn or understand a new technology, I have found that the best way to do so is to build something with it. This hands-on approach allows me to explore the technology in depth, understand how it works, and identify any issues or limitations. It also gives me the opportunity to experiment with different features and configurations, and to see how they affect the performance and behavior of the system.

One technology that I have been interested in for a while is Redis, an open-source, in-memory data structure store that can be used as a database, cache, and message broker. Redis is known for its speed, simplicity, and flexibility, and is widely used in web applications, real-time analytics, and other use cases where high performance and scalability are required.

Writing a Redis client seemed like a good way to learn more about Redis and its features, as well as to explore the design and implementation of a networked client-server application. In this series of posts, I will share my experiences in building a Redis client from scratch, and the challenges and insights that I encountered along the way.
<!--more-->

## Collecting Clues

Before we can start building the Redis client, we need to gather some information about Redis and its protocol. 
Redis provides a [detailed specification](https://redis.io/topics/protocol) of its protocol, which describes how clients and servers communicate with each other over a TCP connection. 
The protocol is text-based, and uses a simple request-response model, where clients send commands to the server and receive responses in return.

We will be focusing on the Redis Commands and Data Types that are used when Redis is uses as a Cache or Data Store. There are two versions of the Redis Protocol, the RESP2 and RESP3. We will be using the RESP2 protocol for this series. Extending the client to support the RESP3 protocol will be a future series and will be interesting if the design supports this in a clean and efficient manner.

Redis is more than just a Key-Value store, and some of its features such as Pub/Sub, Transactions, Pipelining, and Lua Scripting will be ignored in this series.

Currently, the most used Redis client library for C# is [StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis). 
We will use this a both a benchmark for performance and inspiration for some features and concepts.
The code for this library is available on [GitHub](https://github.com/StackExchange/StackExchange.Redis). 
It is very well written and uses serveral design options to optimize performance and reliability.
However, it seems to be a bit over-engineered for our purposes, so we will try to keep things simple and focus on the core functionality of the Redis client.

We will use [XUnit](https://xunit.net/) for unit testing, and [BenchmarkDotNet](https://benchmarkdotnet.org/) for performance testing.

We will also be using Docker and the [Redis Docker Image](https://hub.docker.com/_/redis/) server for testing and benchmarking the client.

## Setting the Stage
Before we can start building the Redis client, we need to set up the development environment and create a project structure for the code.
We will be using Visual Studio 2022 and .NET8, and .NET9, for this project, but you can use any other IDE that you prefer.

### Create the GitHub Repository
I created a GitHub repository for the Redis Client project, which will contain the source code, documentation, and other resources related to the project.
The repository is available at [Code Sleuth Redis Client](https://github.com/matthewDDennis/CodeSleuth.Redis).
Besides the **main** branch, the repository will have a branch for each of the blog posts in the series.

### Creating the Solution and Projects
- We will clone the repository to our local machine and open it in Visual Studio.
- We will start by creating a new solution in Visual Studio, and adding a class library project for the Redis client code.
- We will then add a folder for Unit Tests and an initial unit test project to the solution, which will contain the test cases for the Redis client.
- We will then add a folder for Performance Tests and an initial performance test project to the solution, which will contain the performance test cases for the Redis client.
- Both the Unit Tests and the Perfomance Tests will reference the Redis Client project.

We will update the solution's Nuget packages to resolve warnings about security vulnerabilities and outdated packages.

The solution structure will look like this:

![Solution Structure](https://www.code-sleuth.com/assets/images/redis-client1/solution-structure.png)


### Setting up the Docker Environment
- We will use Docker Composer to create a Redis Server container for testing and benchmarking the Redis client.
- We will create a `docker-compose.yml` file in the solution folder, which will define the Redis Server container and its configuration.

~~~yaml
version: '3.8'

services:
  redis:
    image: redis
    ports:
      - 6379:6379
~~~

### Creating the Initial Unit Test
- We will create an initial unit tests for the Redis client, which will test the basic functionality of the client, such as connecting to the server, sending commands, and receiving responses. The initial tests will not be implemented and so will fail.
Thses tests will be implemented in the next post.

~~~csharp
using System.Net.Sockets;

namespace InitialTests
{
    public class MinimalTests
    {
        /// <summary>
        /// This test verifies that a Redis server is running.
        /// </summary>
        [Fact]
        public void RedisIsRunning()
        {
            using var client = new TcpClient("localhost", 6379);
            Assert.True(client.Connected);
        }


        /// <summary>
        /// This test verifies that a connection to the Redis server can be created.
        /// </summary>
        /// <exception cref="NotImplementedException"></exception>
        [Fact]
        public void CreateConnection()
        {
            throw new NotImplementedException("CreateConnection test not implemented");
        }

        /// <summary>
        /// This test verifies that a Client to interact with the Redis server can be created.
        /// </summary>
        /// <exception cref="NotImplementedException"></exception>
        [Fact]
        public void CreateClient()
        {
            throw new NotImplementedException("CreateClient test not implemented");
        }

        /// <summary>
        /// This test verifies that a Client can send a Ping command to the Redis server
        /// and receive the response..
        /// </summary>
        /// <exception cref="NotImplementedException"></exception>
        [Fact]
        public void CanSendPingAndReceivePong()
        {
            throw new NotImplementedException("CanSendPingAndReceivePong test not implemented");
        }

    }
}
~~~

### Creating the Initial Performance Test
The initial performance test will be the Baseline for the Redis Client using the StackExchange Redis client.

~~~csharp
using BenchmarkDotNet.Attributes;

using StackExchange.Redis;

namespace Benchmarks
{
    [MemoryDiagnoser]
    
    public class BaseLine
    {
        private static ConnectionMultiplexer redis;
        private static IDatabase db;

        static BaseLine()
        {
            redis = ConnectionMultiplexer.Connect("localhost");
            db = redis.GetDatabase();
        }

        [Benchmark(Baseline = true)]
        public bool SetString()
        {
            return db.StringSet("key", "value");
        }
    }
}
~~~

## Getting a Performance Baseline
Before we start building the Redis client, we need to establish a performance baseline for the client using the StackExchange.Redis client.
This will allow us to compare the performance of our client with the existing client, and identify any areas where we can optimize the client for better performance.

The results of the are as follows:

![Base Line Benchmark](https://www.code-sleuth.com/assets/images/redis-client1/baseline-benchmark.png)

## Wrapping Up
In this post, we have collected the clues and information needed to solve the case of the Redis client, and set the stage for the investigation.
We have explored the Redis protocol, the StackExchange.Redis client, and the tools and technologies that we will be using to build the Redis client.
We have also set up the development environment, created the project structure, and created the initial unit and performance tests for the Redis client.

In the next post, we will start building the Redis client using TDD and SOLID design principles to implement the basic functionality. of the client, such as connecting to the server, sending commands, and receiving responses.