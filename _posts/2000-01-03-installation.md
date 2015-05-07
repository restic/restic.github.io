---
title: "Installation"
bg: '#63BD2F'
color: white
fa-icon: download
---

restic is written using the Go programming language. At the moment, the only way
to install restic on your system is to compile it from source. Please read
the [Getting Started](https://golang.org/doc/install) guide of Go if you need help
installing Go.

In order to install restic, run:

{% highlight text linenos=table %}
export GOPATH=~/src/go
go get github.com/restic/restic/cmd/restic
$GOPATH/bin/restic --help
{% endhighlight %}
