% CocoaPods and GYP
% 2013-02-20

## CocoaPods

[CocoaPods](https://cocoapods.org) is an excellent dependency management system for Objective-C projects. Dependencies are distributed as 'Pods’ which are almost exactly like [gems](http://rubygems.org/). A podspec is a kind of build description, that specifies where the source is to be found, dependencies it has, and how it should be configured and compiled. The best part is that they’re extremely legible.

This is a great improvement over the typical git submodule, or linked Xcode project for each dependency. To use CocoaPods, you specify your application’s dependencies in a Podfile. This is typically a concise list of the pods required to build the application. CocoaPods generates a Pods project with a single target consisting of a single static library custom-built for your app to contain all the dependencies in one place. The Pods project that gets put into a workspace along with your project, so that everything plays nice with Xcode as both and IDE and as a build system. The Pods project doesn’t get checked in to your repository because it can be completely regenerated from your Podfile.

CocoaPods turns out to be a partial build system that takes some of the bite out of Xcode. With respect to dependencies, Xcode is now a mere IDE — it’s no longer the keeper of build configurations. But you still need a project to describe an build an application. I want to cut Xcode out of everything that’s not building and day-to-day code writing. I want to write iOS apps and not check in Xcode project files.

## GYP

[GYP](http://code.google.com/p/gyp/) is a meta build specification that creates build specifications for real build systems. In simpler words, you use GYP to describe a build in a generic way in JSON, and GYP builds an Xcode project for you. I could probably extend CocoaPods to do this, and keep one format for specifying builds of dependencies and targets. But GYP already exists. GYP was built and maintained by Google for the Chromium project.

Gathering tidbits from the web, here’s what I’ve been able to put together.

pulse_poll.gyp

```json
{
    "targets": [
        {
            "target_name": "PulsePoll",
            "type": "executable",
            "mac_bundle": 1,
            "include_dirs" : [
                "substructure",
            ],
            "sources": [
                "src/substructure/main.m",
                "src/substructure/PPAppDelegate.h",
                "src/substructure/PPAppDelegate.m",
                "src/substructure/PulsePoll-Info.plist",
            ],
            "link_settings": {
                "libraries": [
                    "UIKit.framework",
                    "Foundation.framework",
                    "CoreGraphics.framework",
                ],
            },
            "xcode_settings" : {
                "INFOPLIST_FILE" : "src/substructure/PulsePoll-Info.plist",
                "SDKROOT": "iphoneos",
                "TARGETED_DEVICE_FAMILY": "1,2",
                "CODE_SIGN_IDENTITY": "iPhone Developer",
                "IPHONEOS_DEPLOYMENT_TARGET": "5.0",
                "ARCHS": "$(ARCHS_UNIVERSAL_IPHONE_OS)",
                "HEADER_SEARCH_PATHS": "$(inherited)",
                "CLANG_ENABLE_OBJC_ARC": "YES",
            },
        },
    ]
}
```

The four source files are the minimum requirements to build and launch a conventional iOS app. main is the code entry point (your projects have one whether you’re aware of it or not), and HSAppDelegate fulfills a role familiar to any iOS developer. The plist is a standard one that you can lift from the Xcode app template, or any existing app. Xcode will not be happy without one, and you won’t be able to have a valid executable bundle without one either.

CocoaPods and Caveats

To integrate with CocoaPods, I only had to make one otherwise unnecessary change, and that is to specify the deployment target in the project’s Podfile. The reason appears to be that GYP does not set its one and only target as the default target. This apparently confuses CocoaPods into thinking that the deployment target is iOS 4.3, and will then complain loudly if you include any pods that don’t work on that system.

There was trouble around the frameworks. [GYP’s documentation](http://code.google.com/p/gyp/wiki/GypUserDocumentation) suggests using `$(SDKROOT)` as a prefix for importing libraries. But this left the project unable to run on the simulator, though it would run on devices. This is clearly due to settings `SDKROOT` to `iphoneos`, but I couldn’t see how to avoid that. Including frameworks as listed above shows them in red when see them in the project, but it builds and runs in both the simulator and on devices.

GYP doesn’t support generating groups of source files in Xcode projects. It imposes its own structure separating out source files from Frameworks, build files, and Products.

## Thoughts

I think the purpose of a source repository is to represent the knowledge required to build a product. For a CocoaPod, its podspec together with its source code and documentation serves that purpose beautifully. For a while, I will try and use GYP to capture the same

I’m a little concerned with pinning the build description of an application to a technology that only exists to serve the Chromium project and that I have little interest in modifying and maintaining. But in the mean time, I’m satisfied with GYP because it’s a build specification I can read. It also lets me continue using Xcode as an IDE. But seeing just [how large](http://src.chromium.org/svn/trunk/src/build/common.gypi) GYP files can get for a complex project makes me not want to really recommend GYP for any serious work. GYP really seems to shine if you’re building a truly cross-platform project (It can generate Visual Studio projects as well as Xcode, ninja, and probably others).

I would like to see CocoaPods, or a tool more closely aligned with CocoaPods, become capable of specifying Xcode projects and targets. The foundations are there in CocoaPods and [Xcodeproj](https://github.com/CocoaPods/Xcodeproj). I might try and work that out some time.

<div class="none"></div>
### Useful links
[GYP Language Specification](http://code.google.com/p/gyp/wiki/GypLanguageSpecification)
[GYP User Documentation](http://code.google.com/p/gyp/wiki/GypUserDocumentation)
[The Skia project’s GYP file for iOS](https://code.google.com/p/skia/source/browse/trunk/gyp/SimpleiOSApp.gyp?r=5702)
[A useful post on GYP’s Google Group](http://groups.google.com/group/gyp-developer/browse_thread/thread/f683ae11a54301b1/8be8243080675559?lnk=gst&q=iphone#8be8243080675559)
[A big chunk of Chromium’s GYP](http://src.chromium.org/svn/trunk/src/build/common.gypi)
