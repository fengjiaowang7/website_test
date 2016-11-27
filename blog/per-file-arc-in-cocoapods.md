% Per File ARC in CocoaPods
% 2013-05-04

tl;dr

```ruby
Pod::Spec.new do |s|
    s.name = 'MyPod'
    ...
    non_arc_files = 'Classes/FirstNonARCClass.m',
    'Classes/SecondNonARCClass.m',
    'Classes/ThirdNonARCClass.m'
    s.requires_arc = true
    s.source_files = 'Classes/*.{h,m}'
    s.exclude_files = non_arc_files
    s.subspec 'no-arc' do |sna|
        sna.requires_arc = false
        sna.source_files = non_arc_files
    end
end
```

## ARC

Automatic reference counting (ARC) is a good thing. It's a great compile-time technology that makes it easier, in most cases, to not accidentally introduce trivial memory leaks in modern Objective-C apps.

But sometimes, you have older code with traditional memory management; it's not really worth converting to ARC if it's correct, and possibly finely tuned. And sometimes ARC introduces [a little unnecessary overhead](http://www.learn-cocos2d.com/2013/03/confirmed-arc-slow/) that can balloon out of control when handling large numbers of objects. There are also good reasons to put Objective-C objects into structs or other odd places that ARC prohibits or makes difficult (Objective-C++ anyone?).

That said, ARC is almost always a good idea. And so disabling ARC should be the exception, not the rule.

## CocoaPods

[CocoaPods](http://cocoapods.org/) is an excellent dependency management system for Objective-C projects. Dependencies are distributed as 'Pods' which are described in podpecs. A podspec is a kind of build description, that specifies where the source is to be found, dependencies it has, and how it should be configured and compiled.

CocoaPods lets you specify ARC on a spec or subspec basis, but doing so on a per-file basis is not planned for the spec DSL. [The suggested way of doing this](https://github.com/CocoaPods/CocoaPods/issues/589#issuecomment-9350801) is a little unsatisfactory because it has you listing each file to exclude twice. Luckily we're working with ruby and we can just extract the list of files to exclude into a list we can use to exclude from the sources and include in a non-ARC subspec.
