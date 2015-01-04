---
layout:     post-no-pic
title:      ".NET 4.5 Asynchronous Programming"
subtitle:   "Among the great new features brought into the .NET Framework with the version 4.5, the improvement done to the Asynchronous programming is definitely my favourite."
date:       2013-07-28 12:00:00
author:     "Thomas Barthelemy"
---

Among the great new features brought into the .NET Framework with the version 4.5, the improvement done to the Asynchronous programming is definitely my favourite and most expected.

# Introduction

Before we ends up diving into the Async programming, let's talk a little about why would we need that: Performance and User experience.

Let's take a small example: in order to display some content to the user, we need to access the web one or more times. We all know accessing something can take some time depending on the user's location, internet connection, traffic...

In a **Synchronous** code (as opposed to **Asynchronous**), we would need to download those information one by one, one after the other. During this time the software will be waiting until it is finished, which means it **won't** answer to the user during this time. You all experienced some "not responding" software, you know how it feels.

**Asynchronous **programming will allow us to access the web multiple times at the same time (better performances) and to keep the software responding for the user (better user experience).

# Our first step with Async
In this example we will code a little bit of code that allows us to search into one or multiple files. Let's start by the synchronous simple version:

{% highlight csharp %}
        public static bool SearchInFile(string filePath, string searchText)
        {
            var result = File.ReadAllText(filePath).Contains(searchText);
            return result;
        }
{% endhighlight %}

In order to make this method **Asynchronous** we will need to use some new keywords and a new class: **Async**, **Await** and **Task**.

        public static async Task<bool> SearchInFileAsync(string filePath, string searchText)
        {
            return await Task.Run(() => SearchInFile(filePath, searchText));
        }

* **async** will state that our method is **Asynchronous**, that's a good start!
* **Task** represents an operation (task) to be started, in progress or finished, an **async** method can only return a **Task**, **Task&lt;T&gt;**, or **void**.
* **await** can only be used in an **async** method, therefore you should always use **async** and **await** together. **Await** have to be applied on a task, and will return you the **Task** result when it will be finished.

Why this is so cool? The **await** keyword allow you to write code in a **synchronous** way that is easy to read and understand, but to be executed in an **asynchronous** way that is more efficient.

# Dealing with tasks
We already stated, we can **await** a task in order to get its result, so if you **await** a **Task&lt;bool&gt;** you will get a **bool**. That's nice but you got many more options.

Let's see what we can do with multiple tasks: we are know searching into a list of files:

        public static bool SearchInFiles(IEnumerable<string> filesPath, string searchText)
        {
            return filesPath.Any(filePath => SearchInFile(filePath, searchText));
        }

        public static async Task<bool> SearchInFilesAsync(IEnumerable<string> filesPath, string searchText)
        {
            var tasks = filesPath.Select(filePath => SearchInFileAsync(filePath, searchText)).ToList();

            return await Task.WhenAll(tasks).ContinueWith(task => tasks.Any(t => t.Result));
        }

Our first step here is to run a task on each file, but we do not await them right away, instead we store them in a list of tasks. By not awaiting the tasks, we allow them to run simultaneously which will give us better  efficiency.

Once we have our tasks running, we use the static method from the Task class: **Task.WhenAll** that will gives us a task that will be completed **when all** the tasks from the collection are completed. Then we **Continue with** a little test: is there **any** positive** result**, is there a file containing what I was looking for.

So here the **ContinueWith** method allows us to run a lambda expression, a delegate or an anonymous method when a task is finished.
# How to cancel a task
Since here we only care for one single positive result, we could definitely stop to search when we found what we were looking for, which would greatly improve the performances. We could also search our files line by line and stop when something is found.

        public static async Task<bool> SearchInFileAsync(string filePath, string searchText, CancellationToken ct)
        {
            var s = File.Open(filePath, FileMode.Open);
            var sr = new StreamReader(s);
            var result = false;
            while (!sr.EndOfStream)
            {
                if (ct.IsCancellationRequested)
                    break;

                var line = await sr.ReadLineAsync();
                if (!line.Contains(searchText)) continue;

                result = true;
                break;
            }

            sr.Close();
            s.Close();
            return result;
        }

We introduced one more object here: the **Cancellation token**. Whenever we start a task, we are able to give a token in parameter to let us interact further with the task, for example here we check regularly if the task was cancelled to stop searching when necessary.

Let's see where this token comes from:

        public static async Task<bool> AdvSearchInFilesAsync(IEnumerable<string> filesPath, string searchText)
        {
            var cts = new CancellationTokenSource();
            var tasks = filesPath.Select(filePath => SearchInFileAsync(filePath, searchText, cts.Token)).ToList();

            while (tasks.Count > 0)
            {
                var task = await Task.WhenAny(tasks);
                tasks.Remove(task);

                if (! await task) continue;

                cts.Cancel();
                return true;
            }
            return false;
        }

The **Cancellation Token Source** contains a **cancellation token** and allows us to cancel (or cancel after an amount of time) tasks using this token.

Our starting code remains the same, we get our tasks into a list, but this time we will use the "**WhenAny"** method that will be triggered as soon as a task is completed in the list.

Concretely here, each time a task is completed:

* We remove it from the list, else we would be in an infinite loop.
* We check if this task got a positive result, if not we continue.
* In case of a positive result, we cancel the other tasks using the **Cancellation Token Source** and we return our result. All the other tasks in progress will be cancelled as soon a possible.

**Why can't we just simply stop the task, checking manually if a cancel is requested is annoying!?**

Yes, yes it is annoying, but killing a task directly would be very very unsafe and against any best practice: What if our task is in the middle of writing in a file? What if it's in the middle of a request to a database? In order to avoid those high-risk, you have to implement the cancellation process yourself, in order to close your streams and connections for example.

# Conclusion
In this tutorial we barely touched the surface of awesomeness that is asynchronous behaviour, but it's a good start and a **must** for all developers. Remember that plenty **Async** method have been added in the .NET Framework, time to **await** them all!

For those who wants to go further, I would advise you the excellent guide:    
[Asynchronous Programming with Async and Await (C# and Visual Basic)](http://msdn.microsoft.com/en-us/library/vstudio/hh191443.aspx).
