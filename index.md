---
layout: default
---

restic
======

restic is a program that does backups right. The design goals are:

 * *Easy:* Doing backups should be a frictionless process, otherwise you are tempted to skip it.  Restic should be easy to configure and use, so that in the unlikely event of a data loss you can just restore it. Likewise, restoring data should not be complicated.

 * *Fast:* Backing up your data with restic should only be limited by your network or hard disk bandwidth so that you can backup your files every day. Nobody does backups if it takes too much time. Restoring backups should only transfer data that is needed for the files that are to be restored, so that this process is also fast.

 * *Verifiable:* Much more important than backup is restore, so restic enables you to easily verify that all data can be restored.

 * *Secure:* Restic uses cryptography to guarantee confidentiality and integrity of your data. The location the backup data is stored is assumed not to be a trusted environment (e.g. a shared space where others like system administrators are able to access your backups). Restic is built to secure your data against such attackers.

 * *Efficient:* With the growth of data, additional snapshots should only take the storage of the actual increment. Even more, duplicate data should be de-duplicated before it is actually written to the storage backend to save precious backup space.

 * *Free:* restic is free software and licensed under the [BSD 2-Clause License](https://github.com/restic/restic/blob/master/LICENSE) and actively developed on [GitHub](https://github.com/restic/restic/).

Building
========

restic is written in the Go programming language. At the moment, the only way to install restic on your system is to compile it from source. You need at least Go version 1.3.

In order to build restic, execute the following steps:

{% highlight console %}
$ git clone https://github.com/restic/restic
[...]

$ cd restic

$ go run build.go
[...]

$ ./restic --help
Usage:
  restic [OPTIONS] <command>

Application Options:
  -r, --repo= Repository directory to backup to/restore from

Help Options:
  -h, --help  Show this help message

Available commands:
  backup     save file/directory
  cache      manage cache
  cat        dump something
  find       find a file/directory
  fsck       check the repository
  init       create repository
  key        manage keys
  list       lists data
  ls         list files
  restore    restore a snapshot
  snapshots  show snapshots
  version    display version
{% endhighlight %}

Using restic
============

First, we need to create a "repository". This is the place where your backups will be saved at.

In order to create a repository at `/tmp/backup`, run the following command and enter the same password twice:

{% highlight console %}
$ restic init --repository /tmp/backup init
enter password for new backend:
enter password again:
created restic backend 085b3c76b9 at /tmp/backup
Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.
{% endhighlight %}

Remembering your password is important! If you lose it, you won't be able to access data stored in the repository.

Now we're ready to backup some data. Run the following command and enter the repository password you chose above again:

{% highlight console %}
$ restic -r /tmp/backup backup ~/work
enter password for repository:
scan [/home/user/work]
scanned 764 directories, 1816 files in 0:00
[0:29] 100.00%  54.732 MiB/s  1.582 GiB / 1.582 GiB  2580 / 2580 items  0 errors  ETA 0:00
duration: 0:29, 54.47MiB/s
snapshot 40dc1520 saved
{% endhighlight %}

As you can see, restic created a backup of the directory and was pretty fast! If you run the command again, restic will create another snapshot of your data, but this time it's even faster:

{% highlight console %}
$ restic -r /tmp/backup backup ~/shared/work/web
enter password for repository:
using parent snapshot 40dc1520aa6a07b7b3ae561786770a01951245d2367241e71e9485f18ae8228c
scan [/home/user/work]
scanned 764 directories, 1816 files in 0:00
[0:00] 100.00%  0B/s  1.582 GiB / 1.582 GiB  2580 / 2580 items  0 errors  ETA 0:00
duration: 0:00, 6572.38MiB/s
snapshot 79766175 saved
{% endhighlight %}

This is deduplication at work! Now list the all the snapshots in the repository with the `snapshots` command:

{% highlight console %}
$ restic -r /tmp/backup snapshots
enter password for repository:
ID        Date                 Source      Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir     /home/user/work
79766175  2015-05-08 21:40:19  kasimir     /home/user/work
{% endhighlight %}

Restoring a snapshot is as easy as it sounds, just use the following command to restore the contents of the latest snapshot to `/tmp/restore-work`:

{% highlight console %}
$ restic -r /tmp/backup restore 79766175 ~/tmp/restore-work
enter password for repository: 
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
{% endhighlight %}

Contribute and Documentation
============================

Contributions are welcome! More information can be found in [the restic contribution guidelines](https://github.com/restic/restic/blob/master/CONTRIBUTING.md). A document describing the design of restic and the data structures stored on disc is contained in [the design document](https://github.com/restic/restic/blob/master/doc/Design.md).

Contact
=======

If you discover a bug or find something surprising, please feel free to [open a github issue](https://github.com/restic/restic/issues/new). If you would like to chat about restic, there is also the IRC channel `#restic` on `irc.freenode.net`. Or just write an email :)

License
=======

Restic is licensed under "BSD 2-Clause License". You can find the complete text in the file `LICENSE`.
