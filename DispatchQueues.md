# Dispatch Queues

The most common ways of handling multi-threading/concurrency in Cocoa:

**Grand Central Dispatch** and **NSOperation**.


First, let's talk about WHY we need to care about concurrency.

In general, we want to have better user experience. Concurrency provides us a way to do some operations while user is interacting with our app.

Imagine, you have a social platform app like Facebook. When user first logs in, your app will go send a request to fetch user's timeline feed, but your app just freezes.

Well, not technically freezes or crashes, but it pauses for a few seconds to get all data back from the server and updates its interface.

Hmmm, the user who's using the app might want to at least go to his/her profile while the app is downloading something, but your request is blocking the entire thread! So now the user is not able to do anything until the data is sent back.

That's when the concurrency or multi-threading comes in to play!

We basically create a thread or normally use a background thread to download images, then when we get the data from the server, we call a completion handler to update our interface in _main thread_.



### What are dispatch queues?

In your app, blocks (tasks) are being enqueued into these dispatch queues.

There are two types of dispatch queues:

**1. Serial queues**

**2. Concurrent queues**

The tasks that are assigned to both queues are being executed in separate threads than the thread they were created on.


### 1. Serial Queues
It can only execute one task at a time. Each task will be performed after one another. However, they don't know about the tasks in separate queues, so the tasks can still be executed concurrently by different serial queues.

(An example is going to a supermarket and you need to wait in line to get checked out, but there are other lines are cashing other customers out at same time)

### 2. Concurrent Queues
It can execute multiple tasks at same time. But the tasks are executed in the order in which they are enqueued.
It guarantees that all tasks start in the order of queue, but you don't know which tasks gets finished, the order of execution and the number of tasks being executed at any given time.

---------------------------

By default, the system provides each application with a single serial queue and four concurrent queues.

The main dispatch queue is the globally available serial queue that executes tasks on the application’s _main thread_

And four global concurrent queues:

- DISPATCH_QUEUE_PRIORITY_HIGH
- DISPATCH_QUEUE_PRIORITY_DEFAULT
- DISPATCH_QUEUE_PRIORITY_LOW
- DISPATCH_QUEUE_PRIORITY_BACKGROUND

Use `dispatch_get_global_queue` to create it.


```objective-c
dispatch_queue_t dispatch_get_global_queue(long identifier,
                                           unsigned long flags);
```

Example:
```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);

dispatch_async(queue, ^{
      NSURL *url = [NSURL URLWithString:urlString];
      NSData *responseData = [NSData dataWithContentsOfURL:url];
      UIImage *avatarImage = [UIImage imageWithData:responseData];

      if (avatarImage) {
          dispatch_async(dispatch_get_main_queue(), ^{
              completion(avatarImage);
          });
      }
});
```

------------------
source:

https://www.appcoda.com/ios-concurrency/
