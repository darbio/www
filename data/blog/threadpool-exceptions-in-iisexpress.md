---
title: ThreadPool exceptions in IIS Express
date: 2016-01-12
tags: ['rest', 'architecture', 'api']
draft: false
summary: How to run async code in IIS Express on a dev machine without running out of thread pools.
---

When debugging a WebAPI application recently which used an asynchronous library, I came accross a situation where the application kept on throwing the following `InvalidOperationException`:

> There were not enough free threads in the ThreadPool to complete the operation.

Running the following code from a WebAPI controller:

```csharp
int workers;
int completions;
System.Threading.ThreadPool.GetMaxThreads(out workers, out completions);
```

Results in the following:

> workers = 2
>
> completions = 2

This is too low to run any other `async` tasks. For example, when run in a console application, we are returned significantly higher numbers.

> workers = 1023
>
> completions = 1000

After [posting a question on stack overflow](http://stackoverflow.com/questions/34780226/threadpools-in-iis-express/34802401#34802401), and doing some digging, I came accross the following github issue:

[https://github.com/aspnet/Home/issues/94](https://github.com/aspnet/Home/issues/94)

> The default values for the thread pool seem really low under Helios. I personally get workerThreads = completionPortThreads = 16 on a Core i7 machine with 4 cores and hyper-threading enabled (= 8 cores).
>
> An ASP.NET Web Forms app running IIS Express on the same machine comes with far higher default values: workerThreads = 4096 and completionPortThreads = 1000.
>
> You should consider increasing these values to get rid of these weird exceptions:
>
> InvalidOperationException: There were not enough free threads in the ThreadPool to complete the operation.
> In the meantime, manually calling ThreadPool.SetMaxThreads with higher values will do the trick.

Specifically, the [following comment](https://github.com/aspnet/Home/issues/94#issuecomment-48424228) explains the issue:

> This is an issue with Helios specifically. WebEngine4 sets the limits to 2 \* cpuCount and normally System.Web sets the limits to 1000 and 4096 (basically the autoConfig settings in web.config).

The fix is to change the calculation method for how Helios calculates the max threadpool count. To do this yourself, add the following code to your `Startup` class:

```csharp
// When webengine4.dll first starts, it sets the max thread pool size to an artificially low number, and it depends
// on System.Web.dll to set it back. Since we're replacing System.Web.dll, we need to perform this fixup manually.
// For now we'll use 100 * numCPUs.
int newLimits = 100 * Environment.ProcessorCount; // this is actually # cores (including hyperthreaded cores)
int existingMaxWorkerThreads;
int existingMaxIocpThreads;
ThreadPool.GetMaxThreads(out existingMaxWorkerThreads, out existingMaxIocpThreads);
ThreadPool.SetMaxThreads(Math.Max(newLimits, existingMaxWorkerThreads), Math.Max(newLimits, existingMaxIocpThreads));
```

This will be the fix implemented in Helios by the ASP.Net team [in the future](https://github.com/aspnet/Home/issues/94#issuecomment-77884761).

For now, I have set the `Startup` class in my WebAPI application to call this code on construction. The code is wrapped in conditional compilation symbols to ensure that the code is only run when the build configuration is `DEBUG`.

```csharp
public Startup()
{
#if DEBUG
    // HACK
    if (System.Diagnostics.Process.GetCurrentProcess().ProcessName == "iisexpress")
    {
        // Set worker threads as Helios sets them too low for IIS Express
        // https://github.com/aspnet/Home/issues/94
        int newLimits = 100 * Environment.ProcessorCount; // this is actually # cores (including hyperthreaded cores)
        int existingMaxWorkerThreads;
        int existingMaxIocpThreads;
        System.Threading.ThreadPool.GetMaxThreads(out existingMaxWorkerThreads, out existingMaxIocpThreads);
        System.Threading.ThreadPool.SetMaxThreads(Math.Max(newLimits, existingMaxWorkerThreads), Math.Max(newLimits, existingMaxIocpThreads));
    }
#endif
}
```

Happy coding!
