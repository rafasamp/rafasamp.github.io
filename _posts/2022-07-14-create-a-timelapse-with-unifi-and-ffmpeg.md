---
title: "Create A Timelapse With Unifi and Python"
categories:
  - Tech
tags:
  - python
  - unifi
  - ffmpeg
excerpt: "An easy way to capture and generate a timelapse from any Unifi security camera"
header:
  overlay_image: /assets/images/header/create-a-timelapse-with-unifi-and-python.png
  overlay_filter: 0.5
  caption: "Python script to retrieve a snapshot from Unifi"
---

I find timelapses fascinating. Although Unifi has a great software suite it pains me that they do not make their snapshot API available allowing users to export a timelapse straight from the Protect portal. This leaves us with the only option to hacking a solution for ourselves which is what I go through in this post.

## Enabling Anonymous Snapshots

**Be warned!** Enabling anonymous snapshots allows anyone with access to your security network to retrieve a JPEG of the live feed of the camera anonymously. **Do not do this if your camera network is not properly secured** 
{: .notice--danger}

In order for our Python script to retrieve a live snapshot from the Unifi camera we need to enable anonymous snapshots. You may do so using the following steps [provided by JJJ](https://jjj.blog/2019/12/get-snap-jpeg-from-unifi-protect-cameras/):

1. Get into Protect UI https://protect.ui.com/
2. Select NVR you’d like to use
3. On left bottom corner click the Settings gear
4. Click Advanced
5. On the right side, you’ll see Device password, click REVEAL
6. Copy the password
7. Connect to the Camera IP through https, e.g. https://your.camera.ip.address
8. Login with username ubnt and password from above
9. Skip device setup, keep it in Unifi Video mode
10. Enable the “Anonymous Snapshot” on the camera
11. From there you should be able to access http://your.camera.ip.address/snap.jpeg

## Capturing Periodic Snapshots with Python

The next step in the process is to periodically access the camera and retrieve a live snapshot. I have created a simple Python script to aid in the process:

<script src="https://gist.github.com/rafasamp/1514cd289bfc0c12025034da06bde80a.js"></script>

Every time the script is run, it will place a snapshot in a subfolder with the camera name and current time in ISO8601 format [(hi /r/8601!)](https://www.reddit.com/r/ISO8601/)

I have this currently running on a Raspberry Pi 2 via a cron job every 30 minutes. This means that at 60FPS each day cycle will yield one second of smooth timelapse goodness. If you've never created a cron job before, I use the excellent tool at [crontab.guru](https://crontab.guru/)

![image-center](/assets/images/crontab-guru.png){: .align-center}

Next step is to add the job to crontab. This is assuming you'll be running this on Linux of course, for Windows just go ahead and use the Task Scheduler wizard.

First, from the directory you'll be hosting the script and snapshots, run `pwd` and `which python3` to retrieve the full path of the directory and of Python.

Once you have these handy, enter `crontab -e` command to edit the cron file, or to create one if it does not exist.

This will open the VIM or NANO editor depending on your system, hit the `i` key on your keyboard for VIM and specify the scheduling pattern, full path to Python, and the full path to the script. Here is what it looks like on my setup (I am using NANO because reasons):

![image-center](/assets/images/crontab-unifi-snapshot.png){: .align-center}

To verify, enter `crontab -l` and you should see your scheduled job show up. Soon enough (precisely every hour and every 30 minutes on the hour in this case) a screenshot will be written to the folder. Great! Now all that's needed is to leave this running for a good long while and we're one step closer to making a proper timelapse.

## Generating a Timelapse with FFMPEG

This is arguably the easiest step in the process since all you need is to navigate to the directory where your screenshots are and entering the following line:

```
ffmpeg -r 60 -pattern_type glob -i "*.jpeg" -s 1920x1080 -vcodec libx264 output.mp4
```

![image-center](/assets/images/ffmpeg-timelapse-snapshot.png){: .align-center}

Here is what each part of the command does:

* `-r 60` The framerate of the video, in this case 60 frames per second
* `-pattern_type glob -i "*.jpeg"` This tells FFMPEG to grab all the JPEG files in the folder
* `-s 1920x1080` Outputs the video in 1080p, you may change this to your camera resolution or lower if you prefer
* `-vcodec libx264` The codec to be used, libx265 can be used but depending on your hardware it'll take much longer to render
* `output.mp4` The filename of the timelapse

Once the process if finished, your timelapse is ready to be viewed.

### Making a GIF

FFMPEG is one of the most amazing tools out there. With two simple commands we can take our timelapse and make a GIF out of it.

```
ffmpeg -i output.mp4 -filter_complex "[0:v] palettegen" palette.png
ffmpeg -i output.mp4 -i palette.png -r 12 -s 320x180 -filter_complex "[0:v][1:v] paletteuse" output.gif
```

![image-center](/assets/images/timelapse-output.gif){: .align-center}

Credit to [HomeHack](https://homehack.nl/create-animated-gifs-from-mp4-with-ffmpeg/) for the GIF snippet!
