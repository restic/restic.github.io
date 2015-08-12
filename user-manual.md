---
layout: page
title: User Manual
sitenav:
  - { anchor: "building-restic", title: "Building restic" }
  - { anchor: "initialize-repository", title: "Initialize a repository" }
  - { anchor: "create-snapshot", title: "Create a snapshot" }
  - { anchor: "list-snapshots", title: "List all snapshots" }
  - { anchor: "restore-snapshots", title: "Restore a snapshot" }
  - { anchor: "mount-repository", title: "Mount a repository" }
  - { anchor: "browse-repository-objects", title: "Browse repository objects" }
  - { anchor: "manage-repository-keys", title: "Manage repository keys" }
  - { anchor: "check-integrity-consistency", title: "Check integrity and consistency" }
  - { anchor: "sftp-repository", title: "SFTP repository" }
  - { anchor: "s3-repository", title: "S3 repository" }
---

## <a name="building-restic"></a>Building restic

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

## <a name="initialize-repository"></a>Initialize a repository

First, we need to create a "repository". This is the place where your backups will be saved at.

In order to create a repository at `/tmp/backup`, run the following command and enter the same password twice:

{% highlight console %}
$ restic init --repo /tmp/backup
enter password for new backend:
enter password again:
created restic backend 085b3c76b9 at /tmp/backup
Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.
{% endhighlight %}

Remembering your password is important! If you lose it, you won't be able to access data stored in the repository.

## <a name="create-snapshot"></a>Create a snapshot

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

As you can see, restic created a backup of the directory and was pretty fast! If you run the command again, restic will create another snapshot of your data, but this time it's even faster. This is deduplication at work! 

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

You can even backup individual files in the same repository. 

```
$restic -r /tmp/backup backup ~/work.txt
scan [/tmp/backup backup ~/work.txt]
scanned 0 directories, 1 files in 0:00
[0:00] 100.00%  0B/s  220B / 220B  1 / 1 items  0 errors  ETA 0:00
duration: 0:00, 0.03MiB/s
snapshot 31f7bd63 saved
```

In fact several hosts may use the same repository to backup directories and files leading to a greater deduplication.

## <a name="list-snapshots"></a>List all snapshots

Now list the all the snapshots in the repository with the `snapshots` command:

{% highlight console %}
$ restic -r /tmp/backup snapshots
enter password for repository:
ID        Date                 Source      Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir     /home/user/work
79766175  2015-05-08 21:40:19  kasimir     /home/user/work
{% endhighlight %}

## <a name="restore-snapshot"></a>Restore a snapshot

Restoring a snapshot is as easy as it sounds, just use the following command to restore the contents of the latest snapshot to `/tmp/restore-work`:

{% highlight console %}
$ restic -r /tmp/backup restore 79766175 --target ~/tmp/restore-work
enter password for repository: 
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
{% endhighlight %}

## <a name="mount-repository"></a>Mount a repository

Browsing your backup as a regular filesystem is just as easy, create a mount point and use the following command to serve the repository with FUSE.

{% highlight console %}
$ mkdir /mnt/restic
$ restic -r /tmp/backup mount /mnt/restic
enter password for repository:
Now serving /tmp/backup at /tmp/restic
Don't forget to umount after quitting!
{% endhighlight %}

## <a name="browse-repository-objects"></a>Browse repository objects

Internaly, a repository is able to store data of several different types described in the [design documentation](https://github.com/restic/restic/blob/master/doc/Design.md). 

The structure of the repository can be explore with the ```list``` command:

{% highlight console %}
$restic -r /tmp/backup list snapshots
d369ccc7d126594950bf74f0a348d5d98d9e99f3215082eb69bf02dc9b3e464c
{% endhighlight %}

The ```cat``` command allows you to display the json representation of objects such as snapshots or trees and the content of the blobs.

{% highlight console %}
$restic -r /tmp/backup cat snapshot d369ccc7d126594950bf74f0a348d5d98d9e99f3215082eb69bf02dc9b3e464c
enter password for repository:
{
  "time": "2015-08-12T12:52:44.091448856+02:00",
  "tree": "05cec17e8d3349f402576d02576a2971fc0d9f9776ce2f441c7010849c4ff5af",
  "paths": [
    "/home/user/work"
  ],
  "hostname": "localhost",
  "username": "username",
  "uid": 501,
  "gid": 20
}
{% endhighlight %}

## <a name="manage-repository-keys"></a>Manage repository keys

Restic allows to set multiple access keys or passwords per repository. In fact you can list, add, remove and change these keys:

{% highlight console %}
$restic -r /tmp/backup key list
enter password for repository:
 ID          User        Host        Created
----------------------------------------------------------------------
*eb78040b    username    localhost   2015-08-12 13:29:57

$ restic -r /tmp/backup key add 
enter password for repository:
enter password for new key:
enter password again:
saved new key as <Key of username@localhost, created on 2015-08-12 13:35:05.316831933 +0200 CEST>

$restic -r backup key list
enter password for repository:
 ID          User        Host        Created
----------------------------------------------------------------------
 5c657874    username    localhost   2015-08-12 13:35:05
*eb78040b    username    localhost   2015-08-12 13:29:57
{% endhighlight %}

## <a name="check-integrity-consistency"></a>Check integrity and consistency

Imagine a malveillant user get a privileged access to your repository and modify your backup with the intention to make you restore malicious data.

{% highlight console %}
sudo echo "boom!" >> backup/index/d795ffa99a8ab8f8e42cec1f814df4e48b8f49129360fb57613df93739faee97
{% endhighlight %}

Restic allows you to check the integrity and consistency of your backup:

{% highlight console %}
$restic -r /tmp/backup check
Load indexes
ciphertext verification failed
{% endhighlight %}

Furthermore, when trying to restore your snapshot, the same verification will occur.

{% highlight console %}
$restic -r /tmp/backup restore 79766175 --target ~/tmp/restore-work
Load indexes
ciphertext verification failed
{% endhighlight %}

## <a name="sftp-repository"></a>Create an SFTP repository

In order to backup data via SFTP, you must first setup a server with SSH and let it know your public key. Password less login is important since restic fail to connect if it is prompted for credentials.

Then the setup of the SFTP repository can be achieved with the init command:

{% highlight console %}
$restic -r sftp://user@host/tmp/backup init
enter password for new backend:
enter password again:
created restic backend f1c6108821 at sftp://user@host/tmp/backup
Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.
{% endhighlight %}


## <a name="s3-repository"></a>Create an Amazon S3 repository

Restic can backup data on any Amazon S3 bucket. However, changing the url scheme is not enough since Amazon uses special security credentials to sign requests. By consequence, you must first setup the following environment variables with the credentials you obtained while creating the bucket.

{% highlight console %}
$export AWS_ACCESS_KEY_ID=<MY_ACCESS_KEY>
$export AWS_SECRET_ACCESS_KEY=<MY_SECRET_ACCESS_KEY>
{% endhighlight %}

You can then easily initialize a repository that uses your Amazon S3 as a backend.

{% highlight console %}
$restic -r s3://region_name/bucket_name init
enter password for new backend:
enter password again:
created restic backend eefee03bbd at s3://region_name/bucket_name
Please note that knowledge of your password is required to access the repository.
Losing your password means that your data is irrecoverably lost.
{% endhighlight %}
