---
layout: post
title: "Receiving systemd dbus signals"
date:   2011-04-07 12:00:00 -0300
categories: python dbus systemd
category: python
---

It demonstrates how to connect to systemd dbus signal. run this and stop/start crond using systemctl.

{% highlight python linenos %}
import dbus
import dbus.mainloop.glib
dbus.mainloop.glib.DBusGMainLoop(set_as_default=True)

bus = dbus.SystemBus()

proxy = bus.get_object(
    'org.freedesktop.systemd1',
    '/org/freedesktop/systemd1'
    )

interface = dbus.Interface(proxy, 'org.freedesktop.systemd1.Manager')

interface.Subscribe()

def on_properties_changed(*args, **kargs):
    print 'Status Changed....'
    print args
    print kargs

properties_proxy = bus.get_object(
    'org.freedesktop.systemd1',
    interface.GetUnit('crond.service')
    )

properties_interface = dbus.Interface(properties_proxy,
                                      'org.freedesktop.DBus.Properties')

properties_interface.connect_to_signal('PropertiesChanged',
                                       on_properties_changed)

import gobject

loop = gobject.MainLoop()
loop.run()
{% endhighlight %}