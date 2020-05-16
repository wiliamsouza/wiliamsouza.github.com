---
layout: post
title: "Tornado using motor and mongodb gridfs basic restful example"
date:   2013-11-07 12:22:05 -0300
categories: tornado mongodb gridfs
category: python
---

An example using [Tornado](http://www.tornadoweb.org/), [Motor](https://github.com/mongodb/motor#motor) and [MongoDB](http://www.mongodb.org/) [GridFS](http://docs.mongodb.org/manual/core/gridfs/) to create a basic [RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer) api.

{% highlight python %}
import json
import yaml
import http.client

import tornado
import tornado.web
import tornado.gen
import tornado.escape

from bson import json_util

import motor
import motor.web

client = motor.MotorClient('localhost', 27017).open_sync()
database = client.automator


class PackagesHandler(tornado.web.RequestHandler):

    @tornado.web.asynchronous
    @tornado.gen.engine
    def get(self):
        packages = []
        response = {'count': None, 'results': None}
        db = self.settings['database']
        cursor = db.fs.files.find().sort([('_id', -1)])
        while (yield cursor.fetch_next):
            fs = cursor.next_object()
            packages.append(fs)

        response['results'] = packages
        self.set_status(http.client.OK)
        self.write(json.dumps(response, default=json_util.default))
        self.finish()

    @tornado.web.asynchronous
    @tornado.gen.engine
    def put(self):
        response = {'count': None, 'results': None}

        package = self.request.files['package'][0]
        filename = package['filename']

        meta = self.request.files['meta'][0]
        metadata = yaml.load(meta['body'])

        db = self.settings['database']
        fs = yield motor.Op(motor.MotorGridFS(db).open)

        exists = yield motor.Op(fs.exists, {'filename': filename})
        if exists:
            msg = '{0}, already exist.'.format(filename)
            response['results'] = {'detail': msg}
            self.set_status(http.client.CONFLICT)
        else:
            gridfs = yield motor.Op(fs.new_file)
            yield motor.Op(gridfs.write, package['body'])
            yield motor.Op(gridfs.set, 'filename', filename)
            yield motor.Op(gridfs.set, 'content_type', package['content_type'])
            yield motor.Op(gridfs.set, 'metadata', metadata)
            yield motor.Op(gridfs.close)

            result = yield motor.Op(db.fs.files.find_one, {'_id': gridfs._id})
            response['results'] = result
            response['count'] = len(result)
            self.set_status(http.client.CREATED)

        self.write(json.dumps(response, default=json_util.default))
        self.finish()


application = tornado.web.Application([
    (r'/packages', PackagesHandler),
    (r'/packages/(.*)', motor.web.GridFSHandler, {'database': database}),
], database=database)

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

Testing
-------

Lets upload a file:

    $ curl -vv --request PUT --form package=@myfile-0.1.tar.gz --form meta=@myfile.yaml http://localhost:8888/packages

Result:

{% highlight bash %}
* About to connect() to localhost port 8888 (#0)
*   Trying 127.0.0.1... connected
> PUT /packages HTTP/1.1
> User-Agent: curl/7.22.0 (x86_64-pc-linux-gnu) libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3
> Host: localhost:8888
> Accept: */*
> Content-Length: 166919
> Expect: 100-continue
> Content-Type: multipart/form-data; boundary=----------------------------d2d58ff55a6a
>
< HTTP/1.1 100 (Continue)
< HTTP/1.1 201 Created
< Date: Thu, 07 Nov 2013 15:45:05 GMT
< Content-Length: 717
< Content-Type: text/html; charset=UTF-8
< Server: TornadoServer/3.1.1
<
* Connection #0 to host localhost left intact
* Closing connection #0
{"count": null, "results": {"contentType": "application/octet-stream", "chunkSize": 262144, "metadata": {"execute": "adb shell uiautomator myfile", "name": "myfile", "summary": "", "vendors": [[1256, "samsung"]], "version": 0.1, "products": [[26720, "Galaxy Nexus"]], "install": "ant install", "packages": []}, "filename": "myfile-0.1.tar.gz", "length": 166117, "uploadDate": {"$date": 1383839105798}, "_id": {"$oid": "527bb581e169972527029d75"}, "md5": "b2e0fc19135b24c0c73283f52c4c83d5"}}
{% endhighlight %}

Check it is realy added to mongodb:

    $ mongofiles --db automator list

Output:

{% highlight bash %}
connected to: 127.0.0.1
myfile-0.1.tar.gz 166117
{% endhighlight %}

Execute a GET request::

    $ curl -vv --request GET http://localhost:8888/packagesi

Result:

{% highlight bash %}
* About to connect() to localhost port 8888 (#0)
*   Trying 127.0.0.1... Connection refused
* couldn't connect to host
* Closing connection #0
curl: (7) couldn't connect to host
wiliam@waa:~$ curl -vv --request GET http://localhost:8888/packages
* About to connect() to localhost port 8888 (#0)
*   Trying 127.0.0.1... connected
> GET /packages HTTP/1.1
> User-Agent: curl/7.22.0 (x86_64-pc-linux-gnu) libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3
> Host: localhost:8888
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 07 Nov 2013 16:08:30 GMT
< Content-Length: 719
< Etag: "b9bafb261466d514204d124e3054148848c18a8a"
< Content-Type: text/html; charset=UTF-8
< Server: TornadoServer/3.1.1
< 
* Connection #0 to host localhost left intact
* Closing connection #0
{"count": null, "results": {"contentType": "application/octet-stream", "chunkSize": 262144, "metadata": {"execute": "adb shell uiautomator myfile", "name": "myfile", "summary": "", "vendors": [[1256, "samsung"]], "version": 0.1, "products": [[26720, "Galaxy Nexus"]], "install": "ant install", "packages": []}, "filename": "myfile-0.1.tar.gz", "length": 166117, "uploadDate": {"$date": 1383839105798}, "_id": {"$oid": "527bb581e169972527029d75"}, "md5": "b2e0fc19135b24c0c73283f52c4c83d5"}}
{% endhighlight %}
