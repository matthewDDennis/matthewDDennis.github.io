---
title: "The Case of the Redis Client, Part 1: Puzzle Pieces"
layout: post
comments: true
excerpt_separator: <!--more-->
---

![Broken Lock](/assets/images/redis-client2/redis-puzzle.jpg)

This is the second post in a series of posts about building a Redis client from scratch. In the [first post](https://www.code-sleuth.com/the-redis-client-1-collecting-clues/), we explored the Redis protocol and the basic commands that a Redis client needs to support.

Having collected most of the clues we need to solve the mystery of the Redis client, we can now start putting the pieces together and see how they fit. Once we have identified the key components of the Redis client, we can start to understand how they interact with each other and how they can be used to build a working client.

I will use TDD to both articulate our understanding of the Redis client and to validate our assumptions. We will start by writing a test that will fail because the Redis client does not exist yet. We will then create the Redis client and implement the minimum functionality required to make the test pass. We will continue to write tests and implement the necessary functionality until we have a working Redis client.

Using TDD allows me to focus on the immediately relevant parts of the Redis client without getting distracted by gold-plating, nice-to-have features, and premature optimization. This also means we can safely build case incrementally and safely revise our design as new facts become apparent. This approach will help us to understand the Redis client better and to identify any gaps in our knowledge. It will also help us to identify any missing pieces of the puzzle that we need to find.
<!--more-->

I also realize that I got ahead of myself in writing any tests, so the existing test will be removed and we will start from scratch.

## Identifying the Key Components
The [Redis Protocol Specification](https://redis.io/docs/latest/develop/reference/protocol-spec/) and the [Redis Commands Reference](https://redis.io/docs/latest/commands/) provide us with a good starting point for identifying the key components of the Redis client. We can use this information to identify the key components of the Redis client and to understand how they interact with each other.

So, what are the key components of the Redis client?
First of all there is the `RedisClient` itself. The `RedisClient` is responsible for establishing a TCP connection on at a URL using a port to the Redis server. It is also responsible for sending and receiving commands to and from the server and parsing the responses from the server so the results can be returned to the caller. The `RedisClient` is the main component of the Redis client and is responsible for coordinating the other components.

Whether we need separate classes for th `RedisConnection`, `RedisCommandEncoder` and the `RedisResponseParser` is not clear yet. We will start with a single `RedisClient` class and refactor it as necessary. It may become obvious as we proceed to create the solution that Single Responsibility and growing complexity will dictate a refactoring into multiple components. Always remember that the goal is to build a working Redis client, not to build the perfect Redis client. We will only add complexity when it is absolutely necessary. We will look at performance optimizations and other nice-to-have features later.

Also, a Redis Server contains a number of databases, each of which can contain multiple keys and values.The user uses the `RdisClient` t select a database and use that database to set and get keys and values in that database. The Redis client also needs to be able to handle the different data types that Redis supports, such as strings, lists, sets, and hashes.

Different sets of commands are available for each  database, the server and the connection. The Redis client also needs to be able to handle the different data types that Redis supports, such as strings, lists, sets, and hashes. Each of the Commands and Responses and Data Types will be implemented as separate classes.

Redis has grown to be a very feature-rich database, and it is not possible to implement all of the features in a single blog post. We will focus on the most commonly used commands and data types, such as `SET`, `GET`, `DEL`, `INCR`, `DECR`, `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `SADD`, `SMEMBERS`, `HSET`, `HGET`, and `HGETALL`. We will also focus on the most commonly used data types, such as strings, lists, sets, and hashes. The compliment of commands has grown since I last looked at Redis, so we will focus on the most commonly used commands and data types.

## Writing the First Tests
We will start by writing a test that will fail because the Redis client does not exist yet. We will then create the Redis client and implement the minimum functionality required to make the test pass. We will continue to write tests and implement the necessary functionality until we have a working Redis client.

The first test will create a RedisClient and verify that it can connect to the Redis server.

```csharp
using CodeSleuth.Redis;

namespace InitialTests
{
    public class ConnectionTests

    {
        /// <summary>
        /// This test vwe can connect to the Redis server.
        /// </summary>
        [Fact]
        public void CanConnectToRedis()
        {
            var client = new RedisClient();
            Assert.True(client.Connected);
        }
    }
}
```

In order to get this to compile, we need to create a `RedisClient` class with a IsConnected method. We will also create an `IRedisClient` interface that the `RedisClient` class will implement. This will allow us to mock the `RedisClient` class in our tests, or inject alternate implemenations.

```csharp
public interface IRedisClient
{
    bool IsConnected { get; }
}
```

```csharp
public class RedisClient : IRedisClient
{
    public bool Connected => true;
}
```

We now have the simplest implementation of the Redis client that will allow the test to pass. We can now add a test to check IsConnected is false when the client is not connected. Thi will require a change to the `RedisClient` class to actually attempt to create a TCP connection to the Redis server at a URL and port.

We will also need to add a constructor to the `RedisClient` class that takes a URL and a port as parameters. This will allow the client to connect to different Redis servers on different ports.  We will also need to add a `Dispose` method to the `RedisClient` class that will close the TCP connection to the Redis server when the client is disposed.

As you can see, the first tests has already led us to a number of new requirements that we need to implement. This is the power of TDD: it allows us to focus on the immediately relevant parts of the Redis client without getting distracted by gold-plating, nice-to-have features, and premature optimization. We can safely build the case incrementally and safely revise our design as new facts become apparent. This approach will help us to understand the Redis client better and to identify any gaps in our knowledge. It will also help us to identify any missing pieces of the puzzle that we need to find.

```csharp
public interface IRedisClient
{
    bool Connected { get; }
}
```

```csharp
using System.Net.Sockets;

namespace CodeSleuth.Redis
{
    public class RedisClient : IRedisClient, IDisposable
    {
        public string Host { get; init; }
        public int Port   { get; init; }

        private readonly TcpClient _tcpClient;
        private bool disposedValue;

        public RedisClient(string v1, int v2)
        {
            Host = v1;
            Port = v2;

            _tcpClient = new TcpClient(Host, Port);
        }

        public bool Connected => _tcpClient.Connected;

        protected virtual void Dispose(bool disposing)
        {
            if (!disposedValue)
            {
                if (disposing)
                {
                    _tcpClient.Dispose();
                }

                // TODO: free unmanaged resources (unmanaged objects) and override finalizer
                // TODO: set large fields to null
                disposedValue = true;
            }
        }

        // // TODO: override finalizer only if 'Dispose(bool disposing)' has code to free unmanaged resources
        // ~RedisClient()
        // {
        //     // Do not change this code. Put cleanup code in 'Dispose(bool disposing)' method
        //     Dispose(disposing: false);
        // }

        public void Dispose()
        {
            // Do not change this code. Put cleanup code in 'Dispose(bool disposing)' method
            Dispose(disposing: true);
            GC.SuppressFinalize(this);
        }
    }
}
```

Running the tests now will fail because the Redis server is not running with the error "System.Net.Sockets.SocketException : No connection could be made because the target machine actively refused it. [::ffff:127.0.0.1]:6380". This highlights that the creation of the TCP client is dependent on the Redis server being available and is indpendent of whether the a succesfully created TCP client was created. We will need to refactor the tests to handle this situation.
```csharp
using CodeSleuth.Redis;

using System.Net.Sockets;

namespace InitialTests
{
    public class ConnectionTests

    {
        /// <summary>
        /// This test vwe can connect to the Redis server.
        /// </summary>
        [Fact]
        public void CanCreateRedisClientWhenRedisServer()
        {
            var client = new RedisClient("localhost", 6379);
        }

        /// <summary>
        /// This test verifies that the Constructor throw an exception when there is no Redis server.
        /// </summary>
        [Fact]
        public void IsConnectedReturnsFalseWhenNoRedisServer()
        {
            Assert.Throws<SocketException>(() =>
            {
                using var client = new RedisClient("localhost", 6380);
            });
        }
    }
}
```
Now when we run the tests, the first test will fail and the second will pass. In order to correct this we will need to start up Redis server in a Docker container to test the connection. To accomplish this will to use a TestContainer that will start up a Redis server in a Docker container. The test class will not look like this:
```csharp
using CodeSleuth.Redis;

using System.Net.Sockets;

using Testcontainers.Redis;

namespace InitialTests
{
    public class ConnectionTests : IAsyncLifetime
    {
        private readonly  RedisContainer _redisContainer;

        public ConnectionTests()
        {
            _redisContainer = new RedisBuilder()
                .WithImage(RedisBuilder.RedisImage)
                .WithPortBinding(RedisBuilder.RedisPort)
                .Build();
        }

        public async Task InitializeAsync()
        {
            await _redisContainer.StartAsync();
        }
        public async Task DisposeAsync()
        {
            await _redisContainer.DisposeAsync();
        }

        /// <summary>
        /// This test vwe can connect to the Redis server.
        /// </summary>
        [Fact]
        public void CanCreateRedisClientWhenRedisServer()
        {
            var client = new RedisClient("localhost", 6379);
        }

        /// <summary>
        /// This test verifies that the Constructor throw an exception when there is no Redis server.
        /// </summary>
        [Fact]
        public void CantCreateRedisClientWhenNoServer()
        {
            Assert.Throws<SocketException>(() =>
            {
                using var client = new RedisClient("localhost", 6380);
            });
        }
    }
}
```

Now when we run the tests, both pass.

## Sending and Receiving Commands

The next step is to implement the ability to send and receive commands to and from the Redis server. We will start by writing a test that will fail because the Redis client does not yet support sending and receiving commands. We will then implement the minimum functionality required to make the test pass. We will continue to write tests and implement the necessary functionality until we have a working Redis client.

For this we will use the **Ping** command to test the request/response. The test will look like this:
```csharp
/// <summary>
/// This test verifies that we can send a Ping command to the Redis server and get a Pong response.
/// </summary>
[Fact]
public async Task CanSendPingAndGetPong()
{
    using var client = new RedisClient("localhost", 6379);
    var response = await client.SendCommandAsync("PING");
    Assert.Equal("+PONG", response);
}
```

In order for this to compile, we need to add the SendCommandAsync method to the IRedisClient interface and the RedisClient class. The RedisClient class will need to implement the SendCommandAsync method to send a simple Ping command to the Redis server and receive the response from the Redis server. The initial implementation will just throw a NotImplementedException just so we can get the test to compile and verify that the test fails. After that will implement the SendCommandAsync method to send the command to the Redis server and receive the response from the Redis server. This againg will be the minumum implementation to get the test to pass and expects a single line response from the Redis server.
```csharp
/// <summary>
/// Send a command to the Redis server and return the response.
/// </summary>
/// <param name="command"></param>
/// <returns>The response from the Redis server.</returns>
public async Task<string?> SendCommandAsync(string command)
{
    // Send the command to the server.
    var encodedCommand = Encoding.UTF8.GetBytes($"*1\r\n${command.Length}\r\n{command}\r\n");
    await _tcpClient.GetStream().WriteAsync(encodedCommand, 0, encodedCommand.Length);

    // Read the response from the server.
    var responseStream = _tcpClient.GetStream();
    var reader = new StreamReader(responseStream, Encoding.UTF8);
    var response = await reader.ReadLineAsync();

    return response;
}
```

## Summary
We have create a simple Redis client that can connect to a Redis server, send a Ping command to the server, and receive a Pong response from the server. We have used TDD to build the Redis client incrementally and to validate our assumptions. We have also used TDD to identify any gaps in our knowledge and to identify any missing pieces of the puzzle that we need to find.

In the next post, we will continue to build the Redis client by adding support for commands with arguments, such as the SET and GET commands. We will also add support for handling of more complex responses.