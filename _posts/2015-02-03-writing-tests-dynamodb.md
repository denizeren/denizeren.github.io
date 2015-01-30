---
layout: post
title: Writing Tests for Dynamodb Application
---

While developing my side project I needed to write tests for my backend which was using Dynamodb. So connecting to Amazon in my test code was not the solution (obviously), but what was the right way to write tests? I looked at [goamz](https://github.com/AdRoll/goamz/tree/master/dynamodb) library, which I used for dynamodb interaction in my project.

{% highlight bash %}
DYNAMODB_LOCAL_VERSION = 2014-10-07

launch: DynamoDBLocal.jar
    cd dynamodb_local_$(DYNAMODB_LOCAL_VERSION) && java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar

DynamoDBLocal.jar: dynamodb_local_$(DYNAMODB_LOCAL_VERSION).tar.gz
    mkdir -p dynamodb_local_$(DYNAMODB_LOCAL_VERSION)
    [ -f dynamodb_local_$(DYNAMODB_LOCAL_VERSION)/DynamoDBLocal.jar ] || tar -C dynamodb_local_$(DYNAMODB_LOCAL_VERSION) -zxf dynamodb_local_$(DYNAMODB_LOCAL_VERSION).tar.gz

dynamodb_local_$(DYNAMODB_LOCAL_VERSION).tar.gz:
    curl -O https://s3-us-west-2.amazonaws.com/dynamodb-local/dynamodb_local_$(DYNAMODB_LOCAL_VERSION).tar.gz

clean:
    rm -rf dynamodb_local_$(DYNAMODB_LOCAL_VERSION)*
{% endhighlight %}

Voila! This was the trick. Guys at Amazon build a small client-side version of [dynamodb](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html) using Java so that people like me can test their applications without connecting to Amazon.

Using this local dynamodb application I was able to write a small [dynamodb](https://github.com/denizeren/dynamostore) backend for gorilla sessions and write tests for my side project.

Happy hacking.
