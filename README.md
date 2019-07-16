# delta-hpkg

Binary delta updates for Haiku hpkg

## Situation

Nightly builds of e.g., `haiku.hpkg` or `qt.hpkg` are rather large (many megabytes) yet probably contain much redundancy between builds. Especially for developers/testers who like to be on "master", this means a lot of redundant downloading, which may not only be time-consuming but even expensive (think of users using slow, expensive, metered connections)

## The idea

* Only download what has changed between the "old" and the "new" `.hpkg`
* This can be achieved by checksumming chunks of the file, and only download the chunks that have different checksums (there are existing mechanisms like [zsync](http://zsync.moria.org.uk/) that do exactly this)

## Related areas

* Peer-to-peer software distribution ("filesharing built into the package manager"). User story: "When 500 machines in a school network are updated, wouldn't it be neat if not all 500 would hit Haiku's server, but download from each other instead?"

* On-disk block deduplication. User story: "I don't my hard disk to be filled up by all those versions of everything"

Possibly they should be thought through together, because they may use similar (the same?) techniques for checksumming chunks of the file. The mechanism for checksumming chunks of the file can be most efficient when it knows about the inner workings of the files to be checksummed. An initial implementation could do without knowing the innards, and then it could later be made more effcient by adding this knowledge to the innards.

hpkg currently uses https://www.zlib.net/ compression although Haiku wants to move to zstandard.

## Proof of concept

Work in progress. Currently Linux is used because the tools are not on Haiku yet.

I have prepared some files for testing and stored them on GitHub Releases:

```
#
# Prepare files
#

wget https://github.com/AppImage/zsync2/releases/download/continuous/zsyncmake2-137-bacc238-x86_64.AppImage
chmod +x *.AppImage

# GitHub renames files when uploaded, this breaks zsync; hence workaround
mv haiku-r1~beta1_hrev53233-1-x86_64.hpkg haiku-r1.beta1_hrev53233-1-x86_64.hpkg
mv haiku-r1~beta1_hrev53259_3-1-x86_64.hpkg haiku-r1.beta1_hrev53259_3-1-x86_64.hpkg

zsyncmake haiku-r1.beta1_hrev53233-1-x86_64.hpkg 
zsyncmake haiku-r1.beta1_hrev53259_3-1-x86_64.hpkg

```

These can now be used for testing:

```
#
# To test
#

wget https://github.com/AppImage/zsync2/releases/download/continuous/zsync2-137-bacc238-x86_64.AppImage
chmod +x *.AppImage

rm haiku-r1.beta1_hrev53259_3-1-x86_64.hpkg

./zsync2-*.AppImage https://github.com/probonopd/delta-hpkg/releases/download/testing/haiku-r1.beta1_hrev53259_3-1-x86_64.hpkg.zsync -i ./haiku-r1.beta1_hrev53233-1-x86_64.hpkg
```

Compared to what we have seen with AppImage, (squashfs), `Usable data from seed files: 1.185383%` is an __extremely__ bad ratio. Why?

Maybe we need to play with some zsyncmake parameters, or (worst case) we must implement knowledge about the inner workings of the files to be checksummed in `zsyncmake2` and `zsync2`.

Someone who understands the inner workings of `.hpkg` might need to have a deep look at http://zsync.moria.org.uk/paper/. Strangely, we did not have to go through those hoops for AppImage, which uses squashfs.
