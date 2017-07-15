# Cancel Blocks in GCD

In iOS8, Apple introduced canceling blocks feature in GCD.
Before iOS8, we would need to use `NSOperation`

[Link here to read more about `NSOperation`](NSOperation.md)

The way the canceling works is to use [`dispatch_after`](https://developer.apple.com/documentation/dispatch/1452876-dispatch_after).

Definition:
```objective-c
void dispatch_after(dispatch_time_t when,
                    dispatch_queue_t queue,
                    dispatch_block_t block);
```

Usage:

```objective-c
dispatch_after(dispatch_time(DISPATCH_TIME_NOW,(int64_t)(10 * NSEC_PER_SEC)),
               dispatch_get_main_queue(),
               ^{
  // do something
});
```

How to use [`dispatch_time`](https://developer.apple.com/documentation/dispatch/dispatch_time_t?language=objc):

```objective-c
dispatch_time_t dispatch_time(dispatch_time_t when, int64_t delta);
```
`when`: Pass `DISPATCH_TIME_NOW` to create a new time value relative to now.

`delta`: The number of nanoseconds to add to the time in the when parameter.

`NSEC_PER_SEC`: The number of nanoseconds in one second.
(i.e `10 * NSEC_PER_SEC` gives 10 seconds)

-----------

However, if the block is already in the middle of getting executed, there is **no way** to cancel it.

In order to cancel the block before it even gets executed, we need to have a reference to the block using [`dispatch_block`](https://developer.apple.com/documentation/dispatch/dispatch_block_t?language=objc).

```objective-c
dispatch_block_t myBlock = dispatch_block_create(0, ^{
  // do something
});
// Then pass this block in to dispatch_after
dispatch_after(dispatch_time(DISPATCH_TIME_NOW,(int64_t)(10 * NSEC_PER_SEC)),
               dispatch_get_main_queue(),
               myBlock);
```

Then finally, use [`dispatch_block_cancel`](https://developer.apple.com/documentation/dispatch/1431058-dispatch_block_cancel?language=objc) to cancel the operation.


```objective-c
dispatch_block_cancel(myBlock);
```


The way to check if a block is canceled successfully, we use [`dispatch_block_testcancel`](https://developer.apple.com/documentation/dispatch/1431046-dispatch_block_testcancel)

Definition:
```objective-c
long dispatch_block_testcancel(dispatch_block_t block);
```

We can simply just pass in our block. If the returned value is a non-zero value, that means the block is canceled, otherwise `zero`.

Example:

For every second, I'm checking if the block gets canceled.

```objective-c
for (int i = 0; i < 10; i ++) {
      [NSThread sleepForTimeInterval:1];

      if (dispatch_block_testcancel(myBlock) != 0) {
          NSLog(@"It got canceled!");
          return;
      }
}

```






-------------------------
Source: http://www.mattrajca.com/2016/04/23/canceling-blocks-in-gcd.html
