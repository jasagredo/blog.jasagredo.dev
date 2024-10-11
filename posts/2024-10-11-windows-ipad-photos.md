---
title: 'Transfering photos from iPad to Windows'
author: Javier
---

TL;DR: See bottom of the post for the steps.

Recently my iPad storage was reaching its limit and most of the contents were
photos. Remembering the times I have attempted to transfer photos to my computer
gives me nightmares still. Every time I had to give up on some photos because
they computer would directly say that it was not able to transfer some items, or
they would just disappear on the destination folder despite the progress bar
getting all the way to 100%.

This time I was determined to get those photos out, and not lose a single one of
them. Recall that my computer is a Windows machine and that the bridge
Microsoft-Apple is not as good as it seems.

## First attemp: Windows photos

The Windows photos app comes with a feature to import photos from external
sources, and that is the method [recommended by
Apple](https://support.apple.com/en-us/120267).

Once you connect the iPad and it shows up in the device manager, it will appear
as an option in the photos app. Notice it doesn't show up in the "Safely remove
hardware and eject media" tray icon.

The Photos app will let you select the "new" photos or all of them. I am unsure
of how it decides which photos are new. Just for safety I usually untrack all
the photos folders in the Photos app and create a new one for each photo dump,
but still it reports that some photos are not new. Anyways, I select all the
photos and tell Photos to dump all of them into a folder.

The painful process starts. More than 1000 images being transferred at a
painfully slow rate, despite the iPad being connected directly to a USB-C port
on my machine. After almost an hour, the process completes, but when I go to my
folder to see the pictures, this is what I get:

![](../images/posts/2024-10-11-windows-ipad-photos/grey1.PNG)

An unbelievable amount of photos have grey/white areas on them. For the other
photos, which don't have these empty areas, they are corrupted and Photos can't
even open them (all the ones that don't have a thumbnail preview are broken):

![](../images/posts/2024-10-11-windows-ipad-photos/folder.PNG)

I just screenshotted a big chunk to show that the error is not occasional. The
vast majority of the photos are gone. Notice that each `JPG` has a `MOV` next to
it with the same name. For the times this has worked it turns out the `MOV` is
the live photo video for each photo.

Good thing that I didn't trust the Photos app, which said everything was
imported just nice.

## Second attempt: apply suggestions from Apple

Turns out that Apple suggests installing the [Apple Devices
app](https://support.apple.com/en-us/118290) first. I never heard of such an
app, and I have been using Apple products for quite a while already.

Installing it had no effect on the result. Most photos ended up broken again.

Another suggestion I saw online was to go to Settings > Apps > Photos and at the
bottom there is an option to "Automatically convert" the photos when
transferring to a Mac or PC. Actually I had this enabled, that's why the photos
were `JPG`s. Otherwise the format would have been `HEIC`, with the same result.

## Third attempt: Online transfer services

Okay so I was not able to transfer the photos via cable. In the past I have used
services like [WeTransfer](https://wetransfer.com/) to send big files to other
people.

The iPad browser is able to transfer files via WeTransfer but WeTransfer only
allows for 2GB of data on each link, and I had like 25GBs of photos. On top of
that, when selecting multiple photos from the library, the iPad loads them one
by one, therefore making it painfully slow to upload all +1000 photos.

At this point I started considering getting only a handful of pictures out via
email (or if they were just a handful maybe WeTransfer could do it).

## Fourth attempt: `magic-wormhole`

In my iPad I have [`iSH`](https://ish.app/) installed. I don't use it
intensively because I don't really use the iPad that much, but I have it around
in case I need a terminal quickly (for example to connecting to my RPi server).

`iSH` being an alpine linux inside an app, I guessed there had to be a
`magic-wormhole` binary for that one. Turns out [there
is](https://pkgs.alpinelinux.org/package/edge/testing/x86/magic-wormhole-rs),
but only in the alpine/edge/testing repository. Modifying the
`/etc/apk/repositories` file with this repository let me install it.

Now if I was able to access the photos library on `iSH` I would be able to send
its contents via `wormhole` directly into my machine. Turns out this is
[not yet implemented](https://github.com/ish-app/ish/issues/89) in `iSH`.

However there was still some hope! From the photos app on the iPad, one can
select a bunch of photos (or all) and tap "Save in Files". I created a folder
and stored all the photos there (duplicating the used space by them sadly). Then
using the command `mount -t ios . /mnt` you get a nice file selector to choose a
folder to mount on `/mnt` inside `iSH`.

I was [full of hope](https://www.youtube.com/watch?v=HA3Ks8NLS-Y) when typing
the command to send the pictures via wormhole when `iSH` answered back with the
following message:

```shell
localhost:~# wormhole-rs send /mnt
Illegal instruction
```

Wryyy!

## Final attempt: SCP

Being about to give up I thought of a different solution, but didn't even know
if it was possible: what if I `scp`'ed that directory? Would I be able to `ssh`
into the iPad if there was an `sshd` server running in `iSH`?

Turns out that you can! Here is [the
link](https://github.com/ish-app/ish/wiki/Running-an-SSH-server) from the
official wiki.

Following that guide I was able to ssh into the iPad from my machine, then run
`scp` to get all the files:

```shell
➜ scp -r root@<ipad's IP>:/mnt ./ipad
root@192.168.1.162's password:
1798DDEA-4526-4C30-8178-CA9053ADA477.JPG                 100% 2024KB   3.0MB/s   00:00
79DCD689-5A3A-4312-B419-CFB91E4D87A0.JPG                 100% 1966KB   4.5MB/s   00:00
IMG_8971.HEIC                                            100% 2316KB   4.0MB/s   00:00
IMG_9663.HEIC                                            100% 2924KB   6.3MB/s   00:00
...
```

Finally after exactly 52 minutes and 49 seconds of tension, all the photos were
downloaded into my computer! Sadly most of them were `HEIC` files, so now I have
to use [`XnView MP`](https://www.xnview.com/en/xnviewmp/) to batch-convert all
those into something I can easily manipulate and more widely supported.

Perhaps I should mention that the Photos app on Windows doesn't open `HEIC`
files, and you are told to download the `HEVC Video Extensions` from the
Microsoft Store, which costs a whole 0,99 eur.

Anyways, this was a painful process, but now I know how to do it, without
getting corrupted photos. I just thought I would dump the investigation here in
case it helps someone and also myself in the future.

## How to safely transfer photos from iPad to a Windows PC

To transfer photos from iPad to PC, the following should work:

1. Install [`iSH`](https://ish.app/) app.
2. Follow the quick step by step guide at the top of [this wiki
   page](https://github.com/ish-app/ish/wiki/Running-an-SSH-server).
3. Go to Settings > Wifi > tap the `i` on the right side of the connected
   network and look for the IP address.
4. Go to your Photos app on the iPad.
5. Select the Photos you want to transfer, tap share and tap "Save on Files".
6. Create a dedicated folder in your iPad's internal storage, and store the
   photos there.
7. Go back to `iSH` and mount that folder into `/mnt` by running `mount -t ios
   . /mnt`.
8. In your computer run the following `scp` command (replace the things between
   angle brackets with real values):
   ```shell
   scp -r root@<ipad's IP>:/mnt <destination folder>
   ```
9. Introduce the password you set in step 2, and the pictures will start being
   transferred.
10. If you can't open the `HEIC` files, consider using [`XnView
    MP`](https://www.xnview.com/en/xnviewmp/) and batch-converting them to
    something else (`Tools > Batch convert...`).
