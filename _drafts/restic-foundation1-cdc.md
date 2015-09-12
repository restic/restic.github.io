---
layout: post
title: Foundation - Introducing Content Defined Chunking (CDC)
---

This post will explain Content Defined Chunking (CDC) and how it is used by restic.

Backup programs need to deal with large amounts of data, saving each file again
to the backup location when a subsequent (usually called "incremental") backup
is created is not efficient. Over time, different strategies have emerged to
handle data in such a case.

The most basic strategy is to only save files that have changed since the last
backup, this is where the term "incremental" backup comes from. This way,
unmodified files are not stored again on subsequent backups. But what happens if
just a small portion of a large file is modified? Using this strategy, the
modified will be saved again, although most data is unchanged.

A better idea is splitting a file into smaller fixed-size pieces (called
"chunks" in the following) of e.g. 1MiB in size. When the backup program saves a
file to the backup location, it is sufficient to save all chunks and the list of
chunks the file consists of. These chunks can be identified for example by the
SHA-256 hash of the content, so duplicate chunks can be detected and saved only
once. For a file consisting of a large number of null bytes, only one chunk of
null bytes needs to be stored.

On a subsequent backup, unmodified files are not saved again because all chunks
have already been saved before. Modified files on the other hand are split into
chunks again, and new chunks are saved to the backup location.
