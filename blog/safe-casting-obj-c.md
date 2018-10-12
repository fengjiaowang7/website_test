% Safe Casting in Objective-C
%
% 2014-02-15


I wrote a small safe casting library for Objective-C. It's available on <a href="https://github.com/fcanas/SafeCast">GitHub</a>.

## The Story

Somewhere on the web I took a quiz to determine how well I know Objective-C. One question 
infuriated me at the time asked something like  “if you have an array `NSArray *a`, how 
do you make it an `NSMutableArray`?” The answer was supposedly 

```objc
NSMutableArray *mutableArray = (NSMutableArray *)a;
```

But this is tremendously dangerous, wrong, crazy… The real answer is something like

```objc
NSMutableArray *mutableArray = [a mutableCopy];
```

Unless you want to actually modify the same instance that `a` is _if_ it’s a mutable array,
in which case you have to check whether it's mutable.

```objc
NSMutableArray *mutableArray;
if ([a isKindOfClass:[NSMutableArray class]]) {
    mutableArray = a;
}
```

At which point you proceed to mutate `mutableArray`, which is also `a`, possibly ignoring the
fact that `mutableArray` may be `nil`. Or maybe you return early if it’s `nil`.

## Casting is Dangerous

If you cast between object that are the same foundational type, no work is done. What I 
mean is that if you cast from an float to an int, since they’re represented differently in
the machine, some work is done to represent that float as a int that changes the value
you’re working with.

In Objective-C, when we’re dealing with objects, we’re dealing with pointers to objects. 
These are really _just_ pointers. They represent a memory address, not a real usable
value. So if you cast an `NSArray` to an `NSMutableArray`, nothing happens. Same if you
case an object to an `int *`. (Which is coincidentally prohibited by ARC. Before ARC,
casting an object to an `int *` used to be perfectly valid, if usually insane.)

Almost every time I see a cast involving an Objective-C object, I get very worried. 
Objective-C has a type system and a compiler that does its best to enforce correct types.
We should use it. Our compiler helps us move finding bugs from run time to compile time. 
Casting is a way to tell the compiler “don’t worry, I know what I’m doing.” But I’ve seen
too many cases of people not knowing what they’re doing. They forget that method 
signatures specify a more general type for a reason. If you are passed an object with type
`<MyProtocol>`, you shouldn’t even assume you’re working with an NSObject. And if you’re
passing around raw `id`s all the time, you should strongly consider coming up with good 
protocols. Or maybe you’re taking a parameter that may be one of two very distinct types 
you say? Well that’s certainly odd, but I hope you’re at least doing the Objective-C
casting dance.

## Getting Rid of the Casting Dance

I wrote the casting dance out above. But here it is again with a little more detail.

```objc
NSMutableArray *ma;
if ([a isKindOfClass:[NSMutableArray class]]) {
    // You have a mutable array!
    ma = a;
} else {
    // You don't have a mutable array!
}
// `ma` is `nil` if `a` wasn't originally an `NSMutableArray`
```

I want to get rid of that, and many things like it. So this project aims to reduce the 
above down to this

```objc
NSMutableArray *ma = [NSMutableArray cast:a];
// `ma` is `nil` if `a` wasn't originally an `NSMutableArray`
```

### Conditional Code

Often what we really want to do is run certain code if an object is of a specific type.

```objc
if ([a isKindOfClass:[NSMutableArray class]]) {
    NSMutableArray *ma = a;
    [ma addObject:x];
}
```

Technically, this could have the same effect

```objc
[[NSMutableArray cast:a] addObject:x];
```

Practically, this ends up getting more complex. Likely I will want to include ways of running a block with the passed object if the object is the kind that’s expected. Maybe something like

```objc
[NSMutableArray cast:a intoBlock:^(NSMutableArray *ma){
    [ma addObject:x];
    [ma addObject:y];
    [ma addObject:z];
}];
```

or even

```objc
BOOL success = [NSMutableArray cast:a intoBlock:^(NSMutableArray *ma){
    [ma addObject:x];
    [ma addObject:y];
    [ma addObject:z];
}];
if (!success){
    // Do whatever it is you need to do if `a` is not mutable...
}
```

## But what’s the point?
### Conciseness

It is more concise, while also being idiomatic.

### Education

Every Objective-C programmer knows how to cast. Sadly, many don’t understand that it
doesn’t give you any guarantees. It just tells the compiler “shut up, I know what I’m 
doing”. [Here’s at least one example more related to fast enumeration](https://stackoverflow.com/questions/18043971/fast-enumeration-does-not-give-correct-object-type-in-objective-c).
But surely you’ve explained this sort of thing to somebody in the past? It’s an honest 
enough mistake to make.

It’s too easy to gloss over the casting dance, correct code that looks very familiar (and
if and a cast), and not realize that there’s a _very_ important technical decision 
being made there. I would rather somebody come across a weird cast: method in my code base 
and ask why it’s there. Then I can go on about the dangers of casting.

### Clarity

The closer that code can read to prose the better the intent of the programmer can be 
understood. It also means that there’s fewer places for bugs to creep in. Consider doing 
something with items in an array.

```objc
NSArray *a;
for (int i = 0; i < a.count; i++) {
    NSLog(@"%@", a[i]);
}
```

Then fast enumeration came along, and off-by-one and out-of-bounds errors became rarer.

```objc
NSArray *a;
for (id object in a) {
    NSLog(@"%@", object);
}
```

And I even prefer block enumeration sometimes.

```objc
NSArray *a;
[a enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    NSLog(@"%@", a);
}];
```

This is especially helpful when dealing with other collection types. Some people don’t 
know off the top of their heads whether fast enumeration of a dictionary iterates over 
objects or keys.

## Digress and Reflect

With all the recent chatter about Apple needing to [replace Objective-C](http://ashfurrow.com/blog/we-need-to-replace-objective-c), 
one common call is to move away from C. That “we’re one NULL pointer dereference away 
from a crash”. I don’t remember the last time code I’ve worked closely with crashed from 
a NULL pointer dereference. It happens when you’re using C language features, and most 
code where you can’t avoid C is code that belongs in C (image manipulation, DSP). 
I think it’s clear that the very best Objective-C developers need to know and love C 
inside and out. But you can make a lot of pretty solid apps without knowing much about C.

I think it’s _much_ more common for an app to crash with unrecognized selector sent
to instance. And here’s why I brought up the “Apple needs to drop Objective-C” thing. 
Objective-C is remarkable for its powers of introspection and runtime manipulation.
