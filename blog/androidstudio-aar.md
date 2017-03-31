I have been struggling lately with integrating an Android Archive (AAR) built
by gomobile into my Android Studio project.

Adding the AAR by itself was simple. But I just couldn't get Android Studio to
reindex the archive when I made changes, nor did it build my app with the
latest contents of the AAR. Some search showed up that I'm not the only one
struggling with this [1]. Unfortunately, I also couldn't get the
experimental gobind Gradle plugin to work.

At last, it turned out to

- create an AAR module in Android studio
- hit the `Sync Project with Gradle File` button after each AAR refresh

And that's it. The rest of this post just outlines these two steps in more
detail.

# One-time setup
I have a test project in my Go path containing both the Android and Go sources
(that's not a requirement, but let's have a concrete example):

    /home/rsto/go/src/gitlab.com/rsto/myproject/MyApplication-Android
    /home/rsto/go/src/gitlab.com/rsto/myproject/mypkg

`MyApplication-Android` is a vanilla Android Studio project. `mypkg` contains
a small Go package that I want to use in my Android project.

    $ cat mypkg/mypkg.go
    package mypkg

    func Foo() string {
        return "foo"
    }

First, I built an AAR from this package (make sure that the `ANDROID_HOME`
environment variable is set).

    $ echo $ANDROID_HOME
    /Users/rsto/Library/Android/sdk

    $ gomobile bind -target android gitlab.com/rsto/myproject/mypkg

This creates a file `mypkg.aar` in the local directory.

Next, I add this AAR file as a AAR module to Android Studio (`File`->
`New`->`New Module`->`Import .JAR/.AAR Package`). Lookup the `mypkg.aar`
file in the filesystem and click `Finish`.

And add the `mypkg` module to the `app` dependencies:

    $ cat MyApplication-Android/app/build.gradle
    apply plugin: 'com.android.application'
    
    android {
		// ...
    }
    
    dependencies {
        // ...
        compile project(':mypkg')
		// ...
    }

# Refreshing the AAR
Let's say I add a new function `Bar` to the `mypkg` package.

    $ cat mypkg/mypkg.go
    package mypkg

    // ...
    
    func Bar() string {
        return "bar"
    }

To make my Android project aware of these changes I rebuild and overwrite the
package:

    $ gomobile bind -target android gitlab.com/rsto/myproject/mypkg && mv mypkg.aar MyApplication-Android/mypkg/

and then I refresh the Gradle projects by hitting the
`Sync Project with Gradle File` button in Android Studio.

Sanity!

[1] https://encrypted.google.com/search?hl=en&q=android%20studio%20aar%20reload#hl=en&q=android+studio+aar+update&*
