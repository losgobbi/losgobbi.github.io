---
title: "Tracking Android 14 Wifi problem at RPI4"
date: 2025-01-24T19:17:33-03:00
categories: ["android"]
---

#### Android
I was doing some experiments with my Raspberry Pi 4 + “the portable display 7 IPS raspberry pi” over the last months like building, installing and testing Android OS (Android 14) from the following repository: [android-rpi](https://github.com/android-rpi/) . Besides that repository, there is also the  [raspberry-vanilla](https://github.com/raspberry-vanilla), and in both cases, they “support” Android for rpi (not just the for pi4). The first one, is not considered a vanilla repository (as we can see in the name), meaning, that is not a pure AOSP fork, it may have some or a set of changes customized to rpi hardware.

On the other hand, at vanilla repo, it’s considered a pure fork, with just opensource stuff, like using at graphics stack the [Mesa3d](https://www.mesa3d.org/) project as a implementation of OpenGL.

Well, at my tests I noticed two issues from “unrelated areas”, Wifi and BT, that kept me very intrigued:

- Wifi was not connecting to either Wifi 2.4GHz and 5Ghz, and the following error was identified (there are other strange messages within wifi services and wpa_supplicant process):

```jsx
11-28 23:14:22.228  2666  2666 I wpa_supplicant: wlan0: CTRL-EVENT-SSID-TEMP-DISABLED id=0 ssid="<ssid>" auth_failures=1 duration=10 reason=WRONG_KEY
```

- And when I’ve tried to use bluetooth, pairing and connecting with a headset, a disconnection happened during the “connection handshake”:

```jsx
11-29 00:03:34.149  3541  3568 I btm_acl : packages/modules/Bluetooth/system/stack/acl/btm_acl.cc:205 hci_btsnd_hcic_disconnect: Disconnecting peer:xx:xx:xx:xx:65:1f reason:HCI_ERR_PEER_USER comment:stack::l2cap::l2c_link::l2c_link_timeout All channels closed
```

There were other strange errors and not necessary the two logs that I’ve quoted are the culprit of Wifi/BT to not be working. Later, I’ve identified a very similar report at this [issue](https://github.com/android-rpi/device_arpi_rpi4/issues/128), for BT case, in which I was trying to help. The android-rpi is less active than vanilla one, at least for now, so in that time I was wondering, in order to pursuit those problems and compare the behavior: *“Maybe I can test this using the vanilla repo?”*. And that, is what I did!!!

#### Tracking

….after spending some time to change my workspace from android-rpi to vanilla one, I was capable of repeating the same tests and compare the logs, in order to see if both “state machines” were equivalent, since they were in a kind of similar Android version, but not exactly using the same infrastructure of wpa, wpa configuration and so on.

At that point, I was driven to discover why android-rpi was in that state? In which scope the issue was? AOSP, “device folder/product configuration”, firmware, Linux Kernel? Well, I did some experiments in order to isolate those cases mixing different things:

- creating the android-rpi device folder inside the vanilla tree;

The idea was to identify if the problem was:

- lack of configurations inside the device structure;
- something wrong with the specific AOSP tag;
- or at another scope;

In this particular mixed case, blobs for Linux Kernel and firmware were copied from vanilla repo.

#### Results:

After some building issues, I’ve faced a loop of different problems over pi4. Some of them were created by me during the lack of attention when mixing these repositories. One of them was a crash at SurfaceFlinger, one process responsible for managing things related to the graphics stack:

```jsx
01-01 00:18:23.959 17253 17273 F libc    : Fatal signal 6 (SIGABRT), code -1 (SI_QUEUE) in tid 17273 (surfaceflinger), pid 17253 (surfaceflinger)
01-01 00:18:24.111 17276 17276 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
01-01 00:18:24.111 17276 17276 F DEBUG   : Build fingerprint: 'arpi/rpi4/rpi4:14/AP2A.240905.003/eng.gobbi.20250103.104810:eng/test-keys'
01-01 00:18:24.111 17276 17276 F DEBUG   : Revision: '0'
01-01 00:18:24.111 17276 17276 F DEBUG   : ABI: 'arm64'
01-01 00:18:24.111 17276 17276 F DEBUG   : Timestamp: 1970-01-01 00:18:23.997155267+0000
01-01 00:18:24.111 17276 17276 F DEBUG   : Process uptime: 1s
01-01 00:18:24.111 17276 17276 F DEBUG   : Cmdline: /system/bin/surfaceflinger
01-01 00:18:24.111 17276 17276 F DEBUG   : pid: 17253, tid: 17273, name: surfaceflinger  >>> /system/bin/surfaceflinger <<<
01-01 00:18:24.111 17276 17276 F DEBUG   : uid: 1000
01-01 00:18:24.111 17276 17276 F DEBUG   : tagged_addr_ctrl: 0000000000000001 (PR_TAGGED_ADDR_ENABLE)
01-01 00:18:24.111 17276 17276 F DEBUG   : signal 6 (SIGABRT), code -1 (SI_QUEUE), fault addr --------
01-01 00:18:24.111 17276 17276 F DEBUG   : Abort message: 'no suitable EGLConfig found, giving up (format: 1, vendor: Android, version: 1.4 Android META-EGL, extensions: EGL_ANDROID_front_buffer_auto_refresh EGL_ANDROID_get_native_client_buffer EGL_ANDROID_presentation_ti
me EGL_EXT_surface_CTA861_3_metadata EGL_EXT_surface_SMPTE2086_metadata EGL_KHR_get_all_proc_addresses EGL_KHR_swap_buffers_with_damage EGL_ANDROID_get_frame_timestamps , Client API: OpenGL_ES)'
...
```

The “no suitable EGLConfig found” was a indication that something related to the graphics/OpenGL structure was not right after mixing those repositories. After some time I’ve realized that I’ve missed a .dtb overlay file from Linux Kernel when I copied the device folder that was preventing some DRI cards to be populated at /dev, like the original vanilla image, triggering a wrong execution of Mesa3d initialization, crashing the SurfaceFlinger:

```c
// there were no devices entries like the original vanilla tests
console:/ # ls -l  /dev/dri/*
crw-rw-rw- 1 root graphics 226,   0 1970-01-01 00:00 /dev/dri/card0
crw-rw-rw- 1 root graphics 226,   1 1970-01-01 00:00 /dev/dri/card1
crw-rw-rw- 1 root graphics 226, 128 1970-01-01 00:00 /dev/dri/renderD128
```

This was only one of the issues, after fixing that a lot of other instabilities were detected during the “mixing test”. In the end of all of this, I couldn’t stabilize that test and reach the root cause of why both problems were occurring at android-rpi repo. 

Anyway, mixing different devices folders from “different” versions can be tricky and create a lot of instabilities. Considering that the history of changes, in this case, for android-rpi, is not “public known”, it’s hard to point if from the first A14 support Wifi/BT was working or not.
Android for RPI4 doesn’t officially supports LK bootloader/fastboot mode since it uses the sdcard, making every test very expensive.