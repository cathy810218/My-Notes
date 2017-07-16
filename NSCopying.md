# NSCopying Protocol

Sometimes we want to create a copy of our object.
But when we do that, we need to think about what we really want to do.


There are two types of copy:

1. Shallow copy
2. Deep copy

### What is _Shallow copy_?

Copying by default for all the collection classes in Foundation is _shallow_.

Example of a Person class

```objective-c
// Person.h
#import <Foundation/Foundation.h>

@interface Person: NSObject
@property (copy, nonatomic) NSString *firstName;
@property (copy, nonatomic) NSString *lastName;
@property (strong, nonatomic) NSNumber *age;

- (instancetype)initWithFirstName:(NSString *)firstName
                         lastName:(NSString *)lastName
                           andAge:(NSNumber *)age;
@end
```

And create a `Person` in another view controller:

```objective-c
Person *personA = [[Person alloc] initWithFirstName: @"Cathy"
                                           lastName: @"Oun"
                                             andAge: @25];
Person *personB = [personA copy];

personA.age = 10;
```

In the example above, `PersonB`'s age will also be changed!
And this is call _Shallow Copy_.

If you print out `PersonA` and `PersonB`, we should notice that they have the same memory address!

Therefore, _Shallow Copy_ only copies the reference pointer.
In our case, `PersonB` is now pointing to the same object as `PersonA`.

< Object is created in Heap while references pointer is created in Stack >

What if you want to create an instance of the same person? like a clone of `PersonA`?

You will need to implement `NSCopying` protocol.

_NSCopying_ protocol only has one method that you need to implement -

```objective-c
- (instancetype)copyWithZone:(NSZone *)zone
```

And since we need the external to know that `Person` implements `NSCopying`,
we need to specify this in the header file like so:

```objective-c
@interface Person: NSObject <NSCopying>
```

Then inside your implementation file `Person.m`, you just need to add one extra method:

```objective-c
- (instancetype)copyWithZone:(NSZone *)zone {
    Person *copy = [[[self class] allocWithZone: zone] initWithFirstName: _firstName
                                                                lastName: _lastName
                                                                  andAge: _age];
    return copy;                                                                  
}
```

And that's how we implement it to allow what we call _Deep Copy_.

If there's any instance variables like an array of friends, then your `Person.m` should look like this:

```objective-c
@implementation Person {
    NSMutableSet *_friends;
}

- (instancetype)initWithFirstName:(NSString *)firstName
                         lastName:(NSString *)lastName
                           andAge:(NSNumber *)age
{
    if (self = [super init]) {
        _firstName = [firstName copy];
        _lastName = [lastName copy];
        _age = [age copy];
        _friends = [NSMutableSet new];
    }
    return self;
}

- (void)addFriend:(Person *)friend {
    [_friends addObject:friend];
}

- (void)removeFriend:(Person *)friend {
    [_friends removeObject:friend];
}

- (instancetype)copyWithZone:(NSZone *)zone {
    Person *copy = [[[self class] allocWithZone: zone] initWithFirstName: _firstName
                                                                lastName: _lastName
                                                                  andAge: _age];
    if (copy) {
        copy->_friends = [_friends mutableCopy];
    }
    return copy;
}
```

Notice here, we use **->** syntax because `_friends` ivar is internal.

We could totally create a property for it, but we just assumed that it'll never
be used externally.

And we allow user to create and remove friend from the array
by calling these two methods:

`addFriend:friend` and `removeFriend:friend`


You might notice that `_friends` uses `mutableCopy`. This is similar to `NSCopying`,
but it uses this method instead:

```objective-c
- (instancetype)mutableCopyWithZone:(NSZone *)zone
```

If you have `mutable` and `immutable` variants, implement `NSMutableCopying`.

`copy` always returns the same class, but `immutableCopy` and `mutableCopy` will return the specific variants.


------------------------

So now, if we want to compare two objects to see if they are equal or the same,
instead of using `==`, we would use `isEqual:`.

`==`: test if two pointers are pointing to the same object in memory (shallow comparison)

`isEqual`: depends on the implementations. In our case, if two `Person`s have the same name, and have same friends we will say they are the same person and return `YES`.



Override the `isEqual:object` method in `Person.m` to check equality for `Person`

```objective-c
-(BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }

    if ([self class] != [object class]) {
        return NO;
    }
    if (![_firstName isEqual:object.firstName] ||
        ![_lastName isEqual:object.lastName] ||
        ![_friend isEqual:object->_friends]) {
            return NO;  
    }
    return YES;
}

```





Source:

https://developer.apple.com/documentation/foundation/nscopying

https://congsun.wordpress.com/2015/06/04/deep-copy-and-shallow-copy/
