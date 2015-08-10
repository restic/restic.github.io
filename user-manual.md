---
layout: manual
submenu:
  - { anchor: "create-repository",       title: "Create a repository" }
  - { anchor: "backup-directories-files", title: "Backup directories and files" }
  - { anchor: "list-snapshots", title: "List all snapshots" }
  - { anchor: "restore-snapshots", title: "Restore a snapshot" }
  - { anchor: "mount-repository", title: "Mount a repository" }
  - { anchor: "backup-stdin", title: "Backup from stdin" }
  - { anchor: "restore-stdout", title: "Restore to stdout" }
  - { anchor: "sftp-repository", title: "SFTP repository" }
  - { anchor: "s3-repository", title: "S3 repository" }
  - { anchor: "http-repository", title: "HTTP repository" }

---

User manual
===========

## <a name="create-repository"></a>Create a repository

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

## <a name="backup-directories-files"></a>Backup directories and files

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

## <a name="backup-stdin"></a>Backup from stdin

Sometimes you need to backup the output of a command line. It is typically the case when you want to backup a mysql database.

```
here come the command
```

## <a name="mount-repository"></a>Restore to stdout

After backuping data from stdin, it sounds obvious to restore to stdout.

```
here come the command
```

## <a name="sftp-repository"></a>Create an SFTP Repository

Restic can save data on any SFTP server

```
here come the command
```

## <a name="s3-repository"></a>Create an Amazon S3 Repository

Restic can save data on any Amazon S3 Bucket

```
here come the command
```

## <a name="http-repository"></a>Create an REST Repository

Restic can save data on any REST server that respect restic the restic API.

```
here come the command
```