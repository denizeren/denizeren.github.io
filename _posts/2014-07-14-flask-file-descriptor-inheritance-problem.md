---
layout: post
title: Flask file descriptor inheritance problem
---

Last month my friend was writing a service monitoring and controlling app using [flask](http://flask.pocoo.org). The app was simply checking status of service in the server and also start, stop and restart functions were available to users in this app.

In about a week my friend came across with an expected bug, file descriptor inheritance. He was complaining about port already in use errors. When I checked who was listening to that port using [netstat](http://linux.die.net/man/8/netstat) it was interesting, the services managed by the app were listening to app's port.

{% highlight bash %}
crazy-server# netstat -nlp|grep 8000
tcp        0      0 0.0.0.0:8000           0.0.0.0:*               LISTEN      1958/mysql
{% endhighlight %}

The problem was obviously file descriptor inheritance. after service monitor app starts or restarts a service its open file descriptors were inherited by the new started processes.

Solution was setting CLOEXEC flag using fcntl. To do that we first had to get all open file descriptors since flask was only creating sockets we needed to find only socket objects. Using python's garbage collector we fetched all socket objects.

{% highlight python %}
filter(lambda x: type(x) == socket._socketobject, gc.get_objects())
{% endhighlight %}

After that we set CLOEXEC flag on all socket file descriptors.
{% highlight python %}
for sock in filter(lambda x: type(x) == socket._socketobject, gc.get_objects()):
    fd = sock.fileno()
    old_flags = fcntl.fcntl(fd, fcntl.F_GETFD)
    fcntl.fcntl(fd, fcntl.F_SETFD, old_flags | fcntl.FD_CLOEXEC)
{% endhighlight %}

And our fd inheritance bug was solved.

Happy hacking.
