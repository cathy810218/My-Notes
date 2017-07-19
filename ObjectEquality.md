# Object Equality

When we want to compare two objects to see if they are the same, there are two ways of doing it:

1. _Shallow Equality_
2. _Deep Equality_

We need to decide whether to check the object pointer or just a few fields.

### Shallow Equality and Deep Equality

A lot of times we use `==` to check two objects without truly understanding what it's doing.

`==` is checking to see if two objects' pointers are pointing to the same object. -> _Shallow Equality_

We usually use `==` to check primitive types or scalar types.

Example:

```objective-c
NSString *name1 = @"Cathy";
NSString *name2 = name1;
```

From the example, we know that `name2` is pointing to the same `NSString` as `name1`.

So doing `if (name1 == name2)` will return `YES`.

But if we change something like this:


```objective-c
NSString *name1 = @"Cathy";
NSString *name2 = name1;
name1 = @"John";
```

`name1 == name2` will still return `YES`.

This is probably not what we alway want, but it could be occasionally.

If we want to compare strings not their pointers, we need to compare them using `isEqualToString:`

```objective-c
[name1 isEqualToString:name2];
```

So given another example:

```objective-c
NSString *str = @"cathy 123";
NSString *str1 = [NSString stringWithFormat:@"cathy %i", 123];
BOOL equalTest1 = str == str1; // NO
BOOL eqaulTest2 = [str isEqualToString:str1]; // YES
BOOL equalTest3 = [str isEqual:str1]; // YES
```

We now have two strings `str` and `str1` both pointing to their own object in memory. And these two objects have the same string `cathy 123`. However, since two pointers pointing to different objects, `==` returns `NO` whereas using `isEqualToString` and `isEqual` just compare the actual strings, so they both return `YES`. -> _Deep Equality_

### Compare your own objects

Now if we have two `Person` objects, and we want to compare to see if they're equal, we probably don't want to just compare the pointers, right?

Hence, we need to implement `isEqual:` and `hash` methods in our class.

*NOTE: The default implementations of these methods from `NSObject` class itself work such that two objects are equal iff their pointers values are exactly the same!*

So **without** implementing these two methods, using `isEqual:` is pretty much the same as using `==` to compare to objects.

Quick example on how to implement/override `isEqual:` and create a custom `isEqualToPerson:` to avoid type checking issue.

```objective-c
@interface Person: NSObject
@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, assign) NSUInteger age; // age can't be negative
@end

@implementation
// and in the implementation
- (BOOL)isEqual:(id)object {
    if ([self class] == [object class]) {
        return [self isEqualToPerson:(Person *)object];
    } else {
        return [super isEqual:object];
    }
}

- (BOOL)isEqualToPerson:(Person *)otherPerson {
    if (self == otherPerson) {
        return YES;
    }

    if (![_firstName isEqualToString:otherPerson.firstName] ||
        ![_lastName isEqualToString:otherPerson.lastName] ||
        _age != otherPerson.age) {
        return NO;
    }
    return YES;
}
@end
```

Remember to always use `isEqualToString` for comparing strings. We use `!=` to check for age equality because it's a _scalar_ type.


And there are several ways to implement `hash` method, but the best way to implement it without performance issues and also lower collision rates is as follow:

```objective-c
- (NSUInteger)hash {
    NSUInteger firstNameHash = [_firstName hash];
    NSUInteger lastNameHash = [_lastName hash];
    NSUInteger ageHash = _age;
    return firstNameHash ^ lastNameHash ^ ageHash;
}
```


Besides `NSString`, `NSArray` and `NSDictionary` also provide a specific equality method. (`isEqualToArray` and `isEqualToDictionary`)


--------------
Source: Effective Objective-C 2.0
