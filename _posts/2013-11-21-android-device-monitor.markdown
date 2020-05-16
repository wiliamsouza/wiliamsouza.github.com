---
layout: post
title: "Android device monitor"
date:   2013-11-21 12:22:05 -0300
categories: android monitor
category: android
---

We use android tools gradle plugin [build](http://tools.android.com/build/gradleplugin)
to genereate `ddmlib` JAR file.

Source
------

You can get the source [here](https://github.com/wiliamsouza/monitor).

Clone the repo:

    $ git clone https://github.com/wiliamsouza/monitor.git

Gradle
------

Here you need to point to your local android tools gradle plugin repo.

{% highlight groovy %}
apply plugin: 'java'
apply plugin: 'idea'

sourceCompatibility = 1.5
targetCompatibility = 1.5

version = '0.1'

repositories {
    maven {
        url uri('/home/wiliam/devel/aosp-tools/out/host/gradle/repo')
    }
}

dependencies {
    compile group: 'com.android.tools.ddms', name: 'ddmlib', version: '22.2.0-SNAPSHOT'
}

task runMonitor(dependsOn: 'classes', type: JavaExec) {
    main = 'com.github.wiliamsouza.monitor.Monitor'
    classpath = sourceSets.main.runtimeClasspath
}
{% endhighlight %}

build
-----

to build the project run:

    $ gradle build

result:

{% highlight bash %}
:compileJava
warning: [options] bootstrap class path not set in conjunction with -source 1.5
1 warning
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 4.004 secs
{% endhighlight %}

to build the project run:

    $ gradle runMonitor

Output:

{% highlight bash %}
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:runMonitor
Demo using wait loop to ensure connection to ADB server and then enumerate devices synchronously
- 0123456789ABCDEF

BUILD SUCCESSFUL

Total time: 4.785 secs
{% endhighlight %}
