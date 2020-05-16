---
layout: post
title: "Xchat recently played tracks from last.fm"
date:   2007-10-18 12:00:00 -0300
categories: xchat python last.fm
category: python
---

Xchat recently played tracks from last.fm.

{% highlight python %}
import xchat
import feedparser

LASTFM_USER = 'wiliamsouza83'
URL = 'http://ws.audioscrobbler.com/1.0/user/%s/recenttracks.rss' % LASTFM_USER

def do_request(word, word_eol, userdata):
    if len(word) < 2:
        xchat.command('help LISTEN')
    elif not isinstance(int(word[1]), int):
        print 'Second arg must be an int!'
    else:
        rss = feedparser.parse(URL)
        for n in range(int(word[1])):
        xchat.command('me listen %s' % rss['entries'][n]['link'])
    return xchat.EAT_ALL

xchat.hook_command('LISTEN',
                   do_request,
                   help='/LISTEN [number]. Show the latest played track from last.fm.')
{% endhighlight %}
