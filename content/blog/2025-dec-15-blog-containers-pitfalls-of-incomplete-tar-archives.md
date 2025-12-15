+++
title = "Containers: pitfalls of incomplete tar archives"
date = 2025-12-15
slug = "2025-dec-15-blog-containers-pitfalls-of-incomplete-tar-archives"
+++

# Containers: pitfalls of incomplete tar archives

As [Colin Walters](https://blog.verbum.org/) likes to say, containers
are just tarballs wrapped in JSON.  We'll mostly ignore the JSON part
here, but there's a lot to unpack (pun and foreshadowing intended)
with regards to tar and how it interacts with the various pieces that
work in conjunction to make containers a reality.

## A refresher on `tar`

Let's explore a key quirk of tar from which everything else we'll
discuss follows.

First, let's create a basic tar archive, which contains a directory
with a file inside of the directory:

```
⬢ [jeckersb@toolbx tmp]$ mkdir foo
⬢ [jeckersb@toolbx tmp]$ touch foo/bar
⬢ [jeckersb@toolbx tmp]$ tar cf foo.tar foo/
```

Now let's list the contents of the tar archive:

```
⬢ [jeckersb@toolbx tmp]$ tar tvf foo.tar
drwxr-xr-x jeckersb/jeckersb 0 2025-12-10 11:09 foo/
-rw-r--r-- jeckersb/jeckersb 0 2025-12-10 11:09 foo/bar
```

Ok, that makes sense.  We see the directory and the file we added to
the archive.

But here's the key bit:  What happens if you add *only* the file to the archive?

```
⬢ [jeckersb@toolbx tmp]$ tar cf incomplete.tar foo/bar
⬢ [jeckersb@toolbx tmp]$ tar tvf incomplete.tar
-rw-r--r-- jeckersb/jeckersb 0 2025-12-10 11:09 foo/bar
```

Now the archive has metadata *only* for the file.  You can deduce from
the path `foo/bar` that there is a directory `foo` involved, but you
can't definitively know how `foo` is supposed to look.  What are the
permissions, owner, group, mtime?  We don't know.  So what happens if
you extract this tar archive?

```
⬢ [jeckersb@toolbx tmp]$ mkdir incomplete-unpacked && tar xf incomplete.tar -C incomplete-unpacked
⬢ [jeckersb@toolbx tmp]$ ls -l incomplete-unpacked/
total 0
drwxr-xr-x. 2 jeckersb jeckersb 60 Dec 10 11:19 foo
⬢ [jeckersb@toolbx tmp]$ ls -l incomplete-unpacked/foo/
total 0
-rw-r--r--. 1 jeckersb jeckersb 0 Dec 10 11:09 bar
```

It implicitly creates the directory `foo` in order to put `bar` at the
correct path.  This makes sense.  But notice `foo` gets created with
metadata that is dependent on the environment in which the unpacking
of the archive happened.  In the above example, you can see that the
timestamp on the directory is 10 minutes later than the timestamp of
the `bar` file inside.  This is because it took me 10 minutes of
writing and editing this post before I did the unpack operation.  I
can set my umask to something else and extract it a second time, and
now we'll see a different timestamp *and* different permissions:

```
⬢ [jeckersb@toolbx tmp]$ umask 0077
⬢ [jeckersb@toolbx tmp]$ mkdir incomplete-unpacked-with-umask && tar xf incomplete.tar -C incomplete-unpacked-with-umask
⬢ [jeckersb@toolbx tmp]$ ls -l incomplete-unpacked-with-umask/
total 0
drwx------. 2 jeckersb jeckersb 60 Dec 10 11:25 foo
⬢ [jeckersb@toolbx tmp]$ ls -l incomplete-unpacked-with-umask/foo/
total 0
-rw-------. 1 jeckersb jeckersb 0 Dec 10 11:09 bar
```

## OCI image spec

OCI container images are serialized as a [series of
tarballs](https://github.com/opencontainers/image-spec/blob/main/layer.md)
that are applied on top of each other to create the complete
filesystem.  We'll get back to the specifics of that operation a bit
later.

The entire layer specification is worth reading, but we'll focus on
two parts.  First, how to determine changes:

```
When two directories are compared, the relative root is the top-level directory.
The directories are compared, looking for files that have been [added, modified, or removed](#change-types).

For this example, `rootfs-c9d-v1/` and `rootfs-c9d-v1.s1/` are recursively compared, each as relative root path.

The following changeset is found:

Added:      /etc/my-app.d/
Added:      /etc/my-app.d/default.cfg
Modified:   /bin/my-app-tools
Deleted:    /etc/my-app-config

This reflects the removal of `/etc/my-app-config` and creation of a file and directory at `/etc/my-app.d/default.cfg`.
`/bin/my-app-tools` has also been replaced with an updated version.
```

And then the following section on representing changes:

```
A [tar archive][tar-archive] is then created which contains _only_ this changeset:

- Added and modified files and directories in their entirety
- Deleted files or directories marked with a [whiteout file](#whiteouts)

The resulting tar archive for `rootfs-c9d-v1.s1` has the following entries:

./etc/my-app.d/
./etc/my-app.d/default.cfg
./bin/my-app-tools
./etc/.wh.my-app-config

To signify that the resource `./etc/my-app-config` MUST be removed when the changeset is applied, the basename of the entry is prefixed with `.wh.`.
```

But note that the changeset says nothing about the directories higher
in the filesystem.  So in the above example, it would be perfectly
valid (and more correct) to generate changeset tar archives which are
incomplete in the same manner we described above.  Most notably, these
changesets would *not* contain tar entries for the directories `/etc`
and `/bin`.

This oversight in the specification is not new or surprising.  There
is an [open issue from
2017](https://github.com/opencontainers/image-spec/issues/737) which
describes exactly this problem.  There is a linked [pull
request](https://github.com/opencontainers/image-spec/pull/970) which
has been open since late 2022 with intermittent discussion, but no
consensus or resolution has been reached yet.

This has led to some projects implementing their own workarounds to
deal with this behavior.  The `containerd` project as an example
[noted this
issue](https://github.com/containerd/containerd/issues/1723) in 2017
and [worked around
it](https://github.com/containerd/containerd/pull/1925) by creating
explicit archive directory entries for parents.

## Rechunking in bootc / rpm-ostree

Base images used via `bootc` are typically post-processed and
finalized via the [rpm-ostree
rechunker](https://coreos.github.io/rpm-ostree/build-chunked-oci/).
The general idea (oversimplifying a bit for brevity) is that the tool
can inspect the RPM database and map each file in the container to its
associated RPM.  It then "rechunks" the container by creating a new
layer for each RPM, where the layer contains the files for that RPM
and only that RPM.  This ensures that when a new version of an RPM is
released in the future, when it is integrated into a new base image,
only the layer associated with that RPM will need to be re-downloaded
by the end user.

However, consider a directory like `/usr`.  This is owned by the
`filesystem` package which contains all of the shared base directories
in the system.  This directory will be included in the layer
associated with the `filesystem` package with all of its related
metadata.

Now, almost every other package in the system is going to have some
amount of content that is shipped under the `/usr` hierarchy.
Originally, the `rpm-ostree` rechunker would omit the archive entry
for `/usr` in each package layer.

This however would cause the behavior of indeterminate tar archives
discussed above to be manifest in the final container.  For example,
it was noted in [this composefs-rs
issue](https://github.com/containers/composefs-rs/issues/132).

We worked around this issue by [creating a new
format-version](https://github.com/coreos/rpm-ostree/pull/5421) for
the `rpm-ostree` rechunker, which creates all of the parent
directories in each tar layer.  This is similar to the fix implemented
as noted above by the `containerd` project.  This works, but it's not
ideal.  It means that all of the parent directories and their
respective metadata have to be duplicated across every layer in the
image.  This is an unnecessary waste of disk space and bandwidth.

## Why isn't this a more widespread problem?

When I first realized what was going on here, I thought I could
trigger this behavior with a `Containerfile` like this:

```
FROM quay.io/fedora/fedora:43

RUN <<EORUN

# Create two levels of directories, because adding a file to a
# directory alters the directory mtime.  This means adding a file
# under /foo/bar in the next layer will cause the mtime for /foo/bar
# to change, but /foo will remain unchanged.
mkdir -p /foo/bar

# zero the mtime on foo
touch -d @0 /foo
EORUN

# Add a file in a derived layer
#
# In theory, this layer would not contain an entry for /foo, since it
# is unchanged.
#
# In practice... not so much.
RUN touch /foo/bar/baz
```

But alas:

```
⬢ [jeckersb@toolbx example]$ podman build -f Containerfile -t localhost/example
STEP 1/3: FROM quay.io/fedora/fedora:43
STEP 2/3: RUN <<EORUN (# Create two levels of directories, because adding a file to a...)
--> d3f72d4d7b90
STEP 3/3: RUN touch /foo/bar/baz
COMMIT localhost/example
--> b730a7409a5a
Successfully tagged localhost/example:latest
b730a7409a5a5b5450f4f1d142415043aeac48f6ac0d1ac2a632d75071668a60
⬢ [jeckersb@toolbx example]$ podman run --rm -it localhost/example stat /foo
  File: /foo
  Size: 6               Blocks: 0          IO Block: 4096   directory
Device: 0,93    Inode: 64851509    Links: 1
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-12-10 21:31:11.973931592 +0000
Modify: 1970-01-01 00:00:00.000000000 +0000
Change: 2025-12-10 21:31:11.973678196 +0000
 Birth: 2025-12-10 21:31:11.973031361 +0000
```

But wait... the directory mtime is (correctly) still zeroed from the
first layer.  So... everything up until now is a lie?

No!  If we pull apart the last tar layer, we'll see that it actually
*does* contain a directory entry for /foo, even though it's not
modified in any way during the last `RUN` invocation.  I'll skip the
part where I re-exported the image to an oci-dir just to make it
easier to look through the layers.  Just trust me when I say this is
the tarball that gets generated for the last layer:

```
⬢ [jeckersb@toolbx sha256]$ tar tvf 9402e1d4a1daea10ff057116f7fc6858be340fd82204eb17db566bfec547a911.tar
drwxr-xr-x 0/0               0 1969-12-31 19:00 foo/
drwxr-xr-x 0/0               0 2025-12-10 16:31 foo/bar/
-rw-r--r-- 0/0               0 2025-12-10 16:31 foo/bar/baz
```

Ok, so `/foo` is there.  But... why?  We didn't modify it.  Deeper
down the rabbit hole we go.

## Overlayfs

Spoiler: the answer is
[overlayfs](https://docs.kernel.org/filesystems/overlayfs.html).

The OCI spec defines the image format, but notably it does *not*
specify any particular implementation details on how to store, manage,
and stitch together the final container image from its individual
layers.

In reality, container runtimes such as podman (via
[containers-libs](https://github.com/containers/container-libs/tree/main/storage),
formerly containers-storage) use the kernel filesystem overlayfs to
stitch together the layers into the final realized view of the world.

A `RUN` invocation in a `Containerfile` is in a nutshell this series
of steps:

1.  Mount the previous layer (and recursively, layers prior to that)
    as a `lowerdir` in an overlayfs filesystem.
2.  Perform the `RUN` invocation in the mounted filesystem.
3.  Collect the contents of the resulting `upperdir` directory.  This
    becomes the tarball for that layer.

This process is how we end up with our `/foo` directory duplicated
into the final layer, even though we did not modify it.  We can mock
this out just using an overlay mount and basic commands.

First, mock our lower layer, and set the mtime:

```
[root@burd tmp]# mkdir lower upper work merged
[root@burd tmp]# mkdir -p lower/foo/bar
[root@burd tmp]# touch -d @0 lower/foo
[root@burd tmp]# stat lower/foo
  File: lower/foo
  Size: 60        	Blocks: 0          IO Block: 4096   directory
Device: 0,45	Inode: 46858       Links: 3
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:user_tmp_t:s0
Access: 1969-12-31 19:00:00.000000000 -0500
Modify: 1969-12-31 19:00:00.000000000 -0500
Change: 2025-12-10 16:53:27.174443520 -0500
 Birth: 2025-12-10 16:53:20.544455919 -0500
```

Next, mount it via overlayfs:

```
[root@burd tmp]# mount -t overlay overlay -o lowerdir=lower,upperdir=upper,workdir=work merged
[root@burd tmp]# cd merged
[root@burd merged]# stat foo
  File: foo
  Size: 60        	Blocks: 0          IO Block: 4096   directory
Device: 0,90	Inode: 46858       Links: 3
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:user_tmp_t:s0
Access: 1969-12-31 19:00:00.000000000 -0500
Modify: 1969-12-31 19:00:00.000000000 -0500
Change: 2025-12-10 16:53:27.174443520 -0500
 Birth: 2025-12-10 16:53:20.544455919 -0500
```

Now we simulate creating the new layer:

```
[root@burd merged]# touch foo/bar/baz
```

Finally we can unmount the overlayfs mount and observe the contents of
the `upper` dir:

```
[root@burd merged]# cd ..
[root@burd tmp]# umount merged
[root@burd tmp]# find upper/
upper/
upper/foo
upper/foo/bar
upper/foo/bar/baz
[root@burd tmp]# stat upper/foo
  File: upper/foo
  Size: 60        	Blocks: 0          IO Block: 4096   directory
Device: 0,45	Inode: 46870       Links: 3
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:user_tmp_t:s0
Access: 2025-12-10 16:57:20.828001765 -0500
Modify: 1969-12-31 19:00:00.000000000 -0500
Change: 2025-12-10 16:56:16.746593313 -0500
 Birth: 2025-12-10 16:56:16.745126092 -0500
```

So that's where our `/foo` entry comes from.  This behavior is
explained in the [overlayfs
documentation](https://docs.kernel.org/filesystems/overlayfs.html#non-directories):

```
Objects that are not directories (files, symlinks, device-special
files etc.) are presented either from the upper or lower filesystem as
appropriate. When a file in the lower filesystem is accessed in a way
that requires write-access, such as opening for write access, changing
some metadata etc., the file is first copied from the lower filesystem
to the upper filesystem (copy_up). Note that creating a hard-link also
requires copy_up, though of course creation of a symlink does not.

The copy_up may turn out to be unnecessary, for example if the file is
opened for read-write but the data is not modified.

The copy_up process first makes sure that the containing directory
exists in the upper filesystem - creating it and any parents as
necessary. It then creates the object with the same metadata (owner,
mode, mtime, symlink-target etc.) and then if the object is a file,
the data is copied from the lower to the upper filesystem. Finally any
extended attributes are copied up.
```

Since the content of the layer tarball is taken directly from the
`upper`, we end up getting a copy of `/foo` via this process.

## Where do we go from here?

Hopefully I have explained the situation adequately and this all makes
sense.  I have achieved some amount of inner peace that comes with
reaching understanding.

However... I still don't like it.  I think we can do better.

Here's the thing for `bootc`: in our rechunked images, I don't want to
include the same directories over and over again in every single
layer.  It shouldn't be necessary, and by strict reading of the OCI
spec we *shouldn't* do that if the directories are unchanged (which
they aren't).

But in the current state of the world, if we *don't* duplicate those
entries into every layer, we end up with non-deterministic directory
metadata when our OCI image is mounted and viewed through the lens of
overlayfs.  This is fundamentally broken as we move bootc into a
future where
[composefs](https://github.com/bootc-dev/bootc/issues/1190) and
[fsverity](https://docs.kernel.org/filesystems/fsverity.html) are used
to harden and validate images.  Ideally a container image should be
able to measure its own fsverity hash in a reproducible manner.  Today
that is not possible.

So the following is what I've done as a proof-of-concept (a huge
motivation for writing this post in the first place was to
exhaustively explain my justification for why this is needed).  I've
written and tested a patch (still needs refining before submitting)
for overlayfs which does the following:

- Adds a new feature `passthrough` (naming bikeshedable) which
  functions similarly to the `metacopy` feature.
- Uses the `trusted.overlay.passthrough` to flag directories as
  "structural-only".
- When encountering a directory with the xattr, searches through the
  lowerdirs until it finds a directory which does not have the xattr
  set.  The metadata for this layer is used.  (If no unflagged
  lowerdir is found, it falls back to the current behavior of using
  non-deterministic local metadata)


Here's an example.  This is similar to our manual overlayfs example
above, except this uses two lower layers to show how unpacking a tar
layer with incomplete directory information might look.

First, create our base layer again:

```
[root@fedora ~]# mkdir upper merged work
[root@fedora ~]# mkdir -p base_layer/foo/bar
[root@fedora ~]# touch -d @0 base_layer/foo
```

Then, in our second layer we create the `baz` file.  This time we set
the `passthrough` xattr on `/foo`, which is how I envision this all
working when a container storage engine unpacks a `tar` archive with
incomplete parent directory information.  Ideally this layer should
include a tar entry for `foo/bar/baz` and (arguably) `foo/bar` since
the directory metadata for `bar` should be updated to reflect the
addition of `bar`.  But it definitely should *not* include an entry
for `foo/`:

```
[root@fedora ~]# mkdir -p second_layer/foo/bar
[root@fedora ~]# touch second_layer/foo/bar/baz
[root@fedora ~]# setfattr -n trusted.overlay.passthrough second_layer/foo
```

Now let's mount it with the patched overlayfs.  First, we'll specify
`passthrough=off` to show the existing behavior:

```
[root@fedora ~]# mount -t overlay overlay -o upperdir=upper,lowerdir=second_layer:base_layer,workdir=work,passthrough=off merged
[root@fedora ~]# stat merged/foo
  File: merged/foo
  Size: 6               Blocks: 0          IO Block: 4096   directory
Device: 0,59    Inode: 365132      Links: 1
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2025-12-10 17:33:32.787147889 -0500
Modify: 2025-12-10 17:33:32.787147889 -0500
Change: 2025-12-10 17:34:01.701054010 -0500
 Birth: 2025-12-10 17:33:32.787147889 -0500
```

Here the `foo` directory has whatever metadata we created for it when
we "unpacked" it with the command `mkdir -p second_layer/foo/bar`
above.  This is what we're trying to avoid.

Now let's unmount and remount with `passthrough=on`:

```
[root@fedora ~]# umount merged
[root@fedora ~]# mount -t overlay overlay -o upperdir=upper,lowerdir=second_layer:base_layer,workdir=work,passthrough=on merged
[root@fedora ~]# stat merged/foo
  File: merged/foo
  Size: 6               Blocks: 0          IO Block: 4096   directory
Device: 0,59    Inode: 365129      Links: 1
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 1969-12-31 19:00:00.000000000 -0500
Modify: 1969-12-31 19:00:00.000000000 -0500
Change: 2025-12-10 17:28:12.268740241 -0500
 Birth: 2025-12-10 17:27:39.158847877 -0500
```

Now we have the metadata for `foo` being passed through the second
layer and retrieved from the base layer, which is the behavior we
want.

## Conclusion

Ensuring that OCI image layers are able to only ship content that
they've modified will aid reproducibility and help unlock
optimizations by avoiding having derived layers always need to inherit
metadata from their parent. At the current time, however, tooling
creating OCI images should include their parent information in order
to maximize compatibility. We are looking forward to a future where
that's not necessary though!
