---
title: "The Case of the Broken Lock"
layout: post
excerpt_separator: <!--more-->
---

![Broken Lock](https://www.code-sleuth.com//assets/images/broken-lock/broken-lock.jpg)

While planning out the next installment of the "Case of the Redis Client", I remembered a case from a few years ago that was causing us serious problems at CodeProject. We were using Redis to cache a lot of data and UI, which drastically improved the User Experience and resource usage. However, occasionally the Redis Client, [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis), would lock up and timeout on all requests. We have 4 front-end webservers, and when the issue occurred, most, if not all, of the webservers would start reporting Redis errors.

Needless to say, the site would slow down to a crawl and become unsuable. This is the story of how I uncovered the unlikely culprit.

<!--more-->

While tracking down the issue, I made the following observations:

- When the problem occurred, I could still access the Redis server using the CLI.
- Restarting the web servers would clear the problem, and all would be fine until the next occurrence.
- Once the issue started on any web server, all Redis requests would fail for that web server.
- This seemed to occur more often under high load, resulting in page requests timing out.
- This did not seem to occur in any of our .NET Core applications, only our .NET Framework applications. Our main site was still running on .NET Framework and ASP.NET WebForms at the time.

Using the Visual Studio Debugger and the source code for StackExchange.Redis, I was able to determine that the issue was caused by a deadlock in the Redis Client. The client was using a static instance of ConditionalWeakTable. Unfortunately, this class sets an _invalid flag before it starts something that could make the data in the table temporarily invalid and clears it when done. In many of its methods that modify the table, the _invalid flag is checked, and an exception is thrown if it is set.

The result of this is that once the ConditionalWeakTable gets into an invalid state, usually as a result of an OOM or ThreadAbort exception in the calling code, the table is now permanently invalid, and all future calls to it will throw an exception. This is what was causing the Redis Client to lock up and timeout.

The StackExchange.Redis library was using locks to protect the ConditionalWeakTable, but the locks were not being released when an OOM or ThreadAbort exception was thrown. This caused the locks to be held indefinitely, preventing any other threads from accessing the table and causing the deadlock. This issue is highlighted in https://devblogs.microsoft.com/dotnet/threadabortexception/.

> If your code calls Thread.Abort, be aware that the call will block if the thread being aborted is in a protected region—such as a catch block or a finally. If the thread calling Thread.Abort is holding a lock that the aborting thread needs, you may deadlock.

Unfortunately, ASP.NET WebForms uses Thread.Abort to handle timeouts and other exceptions, which is why the issue was only occurring in our .NET Framework applications.

I reported the issue to the StackExchange.Redis team in [Issue 1626](https://github.com/StackExchange/StackExchange.Redis/issues/1626). Initially, they were reluctant to fix the issue, as it was limited to what they considered a legacy problem and did not affect their .NET Core applications. Since we had good monitoring in place, we were able to work around the issue by restarting the web servers after the problem occurred, about once a month.

Fortunately, a later version of the library removed the usage of the ConditionalWeakTable, and the issue was resolved. We were able to upgrade to the new version, and the problem has not occurred since.
