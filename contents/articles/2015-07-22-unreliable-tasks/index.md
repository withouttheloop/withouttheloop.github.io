---
title: Dealing With Unreliable TPL Tasks
author: Liam McLennan
date: 2015-07-22 20:32
template: article.jade
---

Suppose you have a collection of .net TPL tasks and the knowledge that some of them may fail (throw an exception) and some may take an unacceptably long period of time to resolve. In the case of the errored tasks you wish to collect information to assist diagnosis. In the case of the long running tasks you wish to ignore them and allow the program to carry on so that throughput is maintained.

This is not a simple problem when using the .net TPL (Task<T>, async, await, etc), which seems to be designed with an all or nothing mentality. Consider the following motely crew of tasks:

    Task<int?>[] calcTasks = new [] {
        Task.Run(() => {
            Thread.Sleep(200);
            var div = 0;
            return new Nullable<int>(5/div);
        }),
        Task.Run(() => {
            Thread.Sleep(100);
            return new Nullable<int>(42);
        }),
        Task.Run(() => {
            Thread.Sleep(1500);
            return new Nullable<int>(99);
        })
    };

All three are asynchronous operations including a time delay. The first task attempts to divide by zero and will therefore throw a `DivideByZero` exception. The last task sleeps for 1.5s, which exceeds the acceptable timeout for my use case. Only the second task will succeed, ultimately producing the value `42`.

What is needed now is a function to process a collection of tasks, in the context of a particular timeout duration, and return a collection of results. Each result should describe the fate of one task. The class I use to represent the result of a Task that may error or timeout is `AsyncTimedResult<TResult>` described below:

    public enum AsyncResultStates
    {
        Unknown, TimedOut, Resolved, Faulted
    }

    public class AsyncTimedResult<TResult>
    {
        public TResult Result { get; private set; }
        public Exception Error { get; private set; }
        public AsyncResultStates State { get; private set; }
        public string Message { get; private set; }

        protected AsyncTimedResult()
        {
            Message = "";
            State = AsyncResultStates.Unknown;
        }

        ...

To process a collection of unreliable tasks I use a function called `ExtractResults`, like so:

    AsyncTimedResult<int?>[] results = await
        UnreliableTasks.ExtractResults(calcTasks, TimeSpan.FromSeconds(1));

I can then categorize the results by their state:

    var succeeded = results.Where(UnreliableTasks.IsResolved);
    var faulted = results.Where(UnreliableTasks.IsFaulted);
    var timedout = results.Where(UnreliableTasks.IsTimedOut);

How Does it Work?
-----------------

The TPL doesn't include any support for adding timeouts to tasks. We simulate timeouts with a bit of a hack. For each task that we wish to evaluate against a timeout, we create a new task that will resolve at the timeout limit, then we race the two tasks using `Task.WhenAny()` which results in the first task to resolve. If the timeout task resolves first then we know that our task timed out.

    var taskICareAbout = Task.Run(() => {
        Thread.Sleep(200);
        var div = 0;
        return new Nullable<int>(5/div);
    });
    var timerTask = Task.Delay(TimeSpan.FromMilliseconds(100))
      .ContinueWith(t => default(int?));
    var first = await Task.WhenAny<int?>(taskICareAbout, timerTask);
    if (first == taskICareAbout) {
      // my task completed first
    } else {
      // timer task completed first
    }

`timerTask` has a continuation just to convert it to the same type as our other task. For `ExtractResults` I also need to process the results and categorize them by state. The full implementation is:

    public static async Task<AsyncTimedResult<T>[]> ExtractResults<T>(IEnumerable<Task<T>> tasks, TimeSpan timeout)
    {
        var tasksWithTimeouts = tasks.Select(t =>
            Task.WhenAny<T>(Task.Delay(timeout).ContinueWith(t2 => default(T)), t));
        var everything = Task.WhenAll(tasksWithTimeouts)
            .ContinueWith(t => t.Result.Select(ti =>
            {
                if (ti.IsFaulted || ti.Exception != null)
                {
                    return AsyncTimedResult<T>.Reject(ti.Exception);
                }
                if (ti.Result == null || ti.Result.Equals(default(T)))
                {
                    return AsyncTimedResult<T>.TimeOut();
                }

                return AsyncTimedResult<T>.Resolve(ti.Result);
            }));
        var results = await everything;
        return results.ToArray();
    }

All Together
--------

Here is a complete console app demonstrating the method:

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;

    namespace TasksC
    {
        class Program
        {
            static void Main(string[] args)
            {
                Do().Wait();
            }

            private static async Task Do()
            {
                Task<int?>[] calcTasks = new[] {
                    Task.Run(() => {
                        Thread.Sleep(200);
                        var div = 0;
                        return new Nullable<int>(5/div);
                    }),
                    Task.Run(() => {
                        Thread.Sleep(100);
                        return new Nullable<int>(42);
                    }),
                    Task.Run(() => {
                        Thread.Sleep(1500);
                        return new Nullable<int>(99);
                    })
                };

                var results = await UnreliableTasks.ExtractResults(calcTasks, TimeSpan.FromSeconds(1));
                var succeeded = results.Where(UnreliableTasks.IsResolved);
                var faulted = results.Where(UnreliableTasks.IsFaulted);
                var timedout = results.Where(UnreliableTasks.IsTimedOut);

                Console.WriteLine(string.Join(",", succeeded.Select(n => n.Result)));
                Console.WriteLine("\nErrors:\n");
                Console.WriteLine(string.Join(",", faulted
                    .Select(n => n.Error.InnerException.Message).ToArray()));
                Console.WriteLine("\nTimeouts:\n");
                Console.WriteLine(timedout.Count());
                Console.ReadLine();
            }
        }

        public class UnreliableTasks
        {
            public static async Task<AsyncTimedResult<T>[]> ExtractResults<T>(IEnumerable<Task<T>> tasks, TimeSpan timeout)
            {
                var tasksWithTimeouts = tasks.Select(t =>
                    Task.WhenAny<T>(Task.Delay(timeout).ContinueWith(t2 => default(T)), t));
                var everything = Task.WhenAll(tasksWithTimeouts)
                    .ContinueWith(t => t.Result.Select(ti =>
                    {
                        if (ti.IsFaulted || ti.Exception != null)
                        {
                            return AsyncTimedResult<T>.Reject(ti.Exception);
                        }
                        if (ti.Result == null || ti.Result.Equals(default(T)))
                        {
                            return AsyncTimedResult<T>.TimeOut();
                        }

                        return AsyncTimedResult<T>.Resolve(ti.Result);
                    }));
                var results = await everything;
                return results.ToArray();
            }

            public enum AsyncResultStates
            {
                Unknown, TimedOut, Resolved, Faulted
            }

            public static bool IsResolved<T>(AsyncTimedResult<T> result)
            {
                return result.State == AsyncResultStates.Resolved;
            }

            public static bool IsFaulted<T>(AsyncTimedResult<T> result)
            {
                return result.State == AsyncResultStates.Faulted;
            }

            public static bool IsTimedOut<T>(AsyncTimedResult<T> result)
            {
                return result.State == AsyncResultStates.TimedOut;
            }

            public class AsyncTimedResult<TResult>
            {
                public TResult Result { get; private set; }
                public Exception Error { get; private set; }
                public AsyncResultStates State { get; private set; }
                public string Message { get; private set; }

                protected AsyncTimedResult()
                {
                    Message = "";
                    State = AsyncResultStates.Unknown;
                }

                public static AsyncTimedResult<TResult> Resolve(TResult result)
                {
                    return new AsyncTimedResult<TResult>() { Result = result, State = AsyncResultStates.Resolved };
                }

                public static AsyncTimedResult<TResult> Reject(Exception ex)
                {
                    return new AsyncTimedResult<TResult>() { Error = ex, State = AsyncResultStates.Faulted };
                }

                public static AsyncTimedResult<TResult> TimeOut()
                {
                    return new AsyncTimedResult<TResult>() { State = AsyncResultStates.TimedOut };
                }
            }
        }
    }
