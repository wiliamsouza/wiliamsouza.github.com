---
layout: post
title: "Android uiautomator gradle build system"
date:   2013-10-30 12:22:05 -0300
categories: android uiautomator gradle java
category: android
---

This will guide you through the steps to write your first uiautomator test using gradle as it build system.

What is gradle?
---------------

"Gradle combines the power and flexibility of Ant with the dependency
management and conventions of Maven into a more effective way to build."

For more visit [gradle](http://www.gradle.org/) page.

What is uiautomator?
-------------------

"The uiautomator testing framework lets you test your user interface (UI)
efficiently by creating automated functional UI testcases that can be run
against your app on one or more devices."

For more visit [uiautomator](http://developer.android.com/tools/testing/testing_ui.html) page.

Java source code
----------------

Create a directory named bluetooth:

    $ mkdir bluetooth

Create a java project layout:

    $ cd bluetooth
    $ mkdir -p src/main/java/com/github/wiliamsouza/bluetooth

Create a java file BluetoothTest.java with the following content:

{% highlight java %}
package com.github.wiliamsouza.bluetooth;

import java.io.IOException;

import com.android.uiautomator.core.UiObject;
import com.android.uiautomator.core.UiObjectNotFoundException;
import com.android.uiautomator.core.UiScrollable;
import com.android.uiautomator.core.UiSelector;
import com.android.uiautomator.testrunner.UiAutomatorTestCase;

public class BluetoothTest extends UiAutomatorTestCase {

    public void setUp() throws UiObjectNotFoundException, IOException {
        getUiDevice().pressHome();
        String packageName = "com.android.settings";
        String component = packageName + "/.Settings";
        String action = "am start -a android.intent.action.MAIN -n ";

        // start settings application
        Runtime.getRuntime().exec(action + component);

        UiObject settingsUi = new UiObject(new UiSelector().packageName(packageName));
        assertTrue("Application settings not started", settingsUi.waitForExists(5000));
    }

    public void testBluetooth() throws UiObjectNotFoundException {
        UiScrollable scroll = new UiScrollable(new UiSelector().scrollable(true));
        UiObject layout = scroll.getChildByText(new UiSelector().className(android.widget.LinearLayout.class.getName()),"Bluetooth", true);
        UiObject switchBluetooth = layout.getChild(new UiSelector().className(android.widget.Switch.class.getName()));
        assertTrue("Unable to find bluetooth switch object", switchBluetooth.exists());
        switchTo(switchBluetooth, "Bluetooth");
    }

    private void switchTo(UiObject switchObject, String name) throws UiObjectNotFoundException {
        // Before start test ensure switch is off
        if (switchObject.isChecked()) {
            switchObject.click();
            sleep(3000);
        }
        
        switchObject.click();
        sleep(3000);
       	assertTrue("Unable to turn " + name + " on", switchObject.isChecked());
       	
    	switchObject.click();
        sleep(3000);
       	assertFalse("Unable to turn " + name + " off", switchObject.isChecked());
    }
}
{% endhighlight %}

Gradle
------

Create a build.gradle with the following content:

{% highlight groovy %}
apply plugin: 'java'
apply plugin: 'idea'

sourceCompatibility = 1.5
targetCompatibility = 1.5

version = '0.1'

project.ext {
    dexDir = new File('build/dex')
    distDir = new File('./dist')
}

repositories {
    mavenCentral()
}

dependencies {
    compile fileTree(dir: androidSdkHome + '/platforms/' + androidSdkTarget, include: '*.jar')
    compile group: 'junit', name: 'junit', version: '4.11'
}

jar {
    doLast {
        tasks.dex.execute()
    }
}

task dex(dependsOn: jar, type:Exec) {
    println 'Building dex...'
    project.dexDir.mkdirs()
    workingDir '.'
    commandLine androidSdkHome + '/' + androidSdkBuildToolsDir + '/' + 'dx', '--dex', '--no-strict', '--output=' + buildDir +'/dex/' + project.name + '.jar', jar.archivePath
    doLast {
        tasks.dist.execute()
    }
}

task dist(dependsOn:dex, type:Copy) {
    project.distDir.mkdirs()
    from(project.dexDir)
    into(project.distDir)
    include('*.jar')
}
{% endhighlight %}

Configure
---------

Now lets configure the build system, Create a file gradle.properties containing:

{% highlight ini %}
androidSdkHome=/home/wiliam/bin/android-sdk-linux
androidSdkTarget=android-18
androidSdkBuildToolsDir=build-tools/18.1.1
{% endhighlight %}

* androidSdkHome -- Point to the root directory of android SDK.
* androidSdkTarget -- Is the target, points to a directory inside platforms.
* androidSdkBuildToolsDir -- Location of dx program.

Build
-----

    $ gradle build

Install
-------

    $ adb push dist/bluetooth.jar /data/local/tmp/

Run
---

    $ adb shell uiautomator runtest bluetooth.jar -c com.github.wiliamsouza.bluetooth.BluetoothTest
