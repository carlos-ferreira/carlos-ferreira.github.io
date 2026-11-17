---
layout: post
title: "Turning an old Android phone into a dedicated music player"
date: 2026-11-17 00:00:00 +0000
categories: android projects
description: "How I turned a Xiaomi Mi A2 Lite into an offline music player for my son, with Bluetooth headphones, a camera and an FM tuner."
---

My son's MP3 player kept losing his music. With Codex's help, I turned an old Xiaomi Mi A2 Lite into an offline media player for him. He can listen through Bluetooth headphones, take pictures and browse his photos, just as he could on his original player.

<link rel="stylesheet" href="{{ '/assets/css/media-device-post.css' | relative_url }}">

<figure class="media-device-photo">
  <a href="{{ '/assets/images/android-media-device/devices-side-by-side.jpg' | relative_url }}"><img src="{{ '/assets/images/android-media-device/devices-side-by-side.jpg' | relative_url }}" alt="The original MP3 player on the left beside the Xiaomi Mi A2 Lite running Media Device on the right" width="1488" height="1984" loading="lazy" decoding="async"></a>
  <figcaption>The original MP3 player (left) and the Xiaomi Mi A2 Lite running Media Device (right): the inspiration and the device I built around it.</figcaption>
</figure>

## Could I fix his MP3 player?

Every so often, all the songs would disappear from the player's SD card. Its FAT32 filesystem was getting corrupted, leaving the player unusable. We found [people on Reddit describing similar problems](https://www.reddit.com/r/mp3players/comments/1eyssyd/comment/ll90tor/), and I couldn't find a clear solution.

I extracted the firmware with [smartlink_flash](https://github.com/ilyakurdyukov/smartlink_flash) and used ChatGPT to help reverse-engineer it. We found a few things worth investigating, but I still couldn't say what had corrupted the card. Testing a change would eventually mean flashing his player, and a bad image could leave it unusable for good. I stopped after the analysis. I had another device that already had the hardware my son needed.

## What I wanted to keep from the original

The Xiaomi was running LineageOS 18.1, based on Android 11. What I liked about my son's player was that he could pick it up, choose a song and listen. I wanted to keep that simplicity, including keeping it offline. Bluetooth was for his headphones. Wi-Fi, cellular data, app installation and general Settings had no place in everyday use. I could load music and look after the device from my computer.

I tried two versions of an iPod-style launcher in the browser. Both took their colorful function icons from the little player. One put a circular control wheel under the icons. The other gave each function a tile and playback its own buttons. On a touchscreen I preferred the second layout.

<div class="media-device-gallery">
  <figure>
    <a href="{{ '/assets/images/android-media-device/prototype-classic-wheel.jpg' | relative_url }}"><img src="{{ '/assets/images/android-media-device/prototype-classic-wheel.jpg' | relative_url }}" alt="Original browser prototype with six colorful feature tiles and an iPod-style circular control wheel" width="402" height="823" loading="lazy" decoding="async"></a>
    <figcaption>Browser prototype: Classic wheel, exploring the dedicated-player idea.</figcaption>
  </figure>
  <figure>
    <a href="{{ '/assets/images/android-media-device/prototype-touch-grid.jpg' | relative_url }}"><img src="{{ '/assets/images/android-media-device/prototype-touch-grid.jpg' | relative_url }}" alt="Original Touch grid browser prototype with six feature tiles and separate playback buttons" width="402" height="846" loading="lazy" decoding="async"></a>
    <figcaption>Browser prototype: Touch grid, the direction selected for the first app shell.</figcaption>
  </figure>
</div>

With that design in hand, I built **Media Device** in [Kotlin](https://developer.android.com/kotlin) with [Jetpack Compose](https://developer.android.com/compose). The first version on the phone still had placeholders. The home screen now has Music, Playlists and Radio across the top, with Camera, Photos and Bluetooth underneath. A shared Now Playing strip brings you back to the current song or station.

<div class="media-device-gallery">
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-early-home.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-early-home.png' | relative_url }}" alt="Early Daisy Android app home screen with six feature tiles, time, battery and playback controls" width="1080" height="2280" loading="lazy" decoding="async"></a>
    <figcaption>The first Touch grid running on the Mi A2 Lite. The coding agents named this early build “daisy” after the phone’s codename. Camera and Photos were still placeholders. I added Radio and rearranged the tiles later.</figcaption>
  </figure>
</div>

## Turning the phone on and finding the player

Before building out those tiles, I wanted to know what my son would see if he pressed Home, restarted the phone or the app crashed. I started with a small policy prototype on the existing ROM. I provisioned it as Android's [Device Owner](https://developer.android.com/reference/android/app/admin/DevicePolicyManager) and enabled managed [lock task mode](https://developer.android.com/work/dpc/dedicated-devices/lock-task-mode). Home, Recents, Back and notification gestures all left the prototype in front. Android also refused to open general Settings.

Then I deliberately crashed the app. Kiosk mode ended, and the Settings screen I'd used earlier was sitting underneath it. Setting the player as the default launcher still didn't reliably bring it back. I gave it a [persistent Home entry](https://developer.android.com/work/dpc/dedicated-devices/cookbook) and put its recovery service in a separate process. When I repeated the crash, the player reopened and lock task mode became active again.

Restarting the phone exposed a different gap. The player came back, but Android showed its swipe lockscreen first, with notifications still within reach. I disabled the nonsecure lockscreen and status bar in daily use and rebooted again. This time the phone reached the player with lock task mode active before I touched the screen or sent a command from my computer.

I kept maintenance on my authorized computer through [USB debugging](https://developer.android.com/tools/adb). I can update the app there, keep its data and return it to kiosk mode. These checks cover the failures I reproduced on this installation. The phone still has an unlocked bootloader and the ROM reported a May 2022 security patch.

## Getting music into his headphones

### Pairing the headphones

The phone found my Sony WH-1000XM4 headphones straight away, then complained about an incorrect PIN. The logs explained the delay. Android had tried to show a pairing confirmation dialog owned by Settings. Kiosk mode had blocked it, so there was nothing on screen for me to confirm.

Trying [companion device pairing](https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing) hit the same obstacle. What worked was temporarily allowing Settings when someone tapped Pair. The app withdraws permission when pairing finishes, fails or times out, or when the user leaves the screen. A separate alarm provides a fallback. The exception covers the whole Settings package. I could open general Settings from my computer during that window, which is why I'd still like to narrow it.

Once paired, I could disconnect while keeping the bond, reconnect the [audio profiles](https://developer.android.com/develop/connectivity/bluetooth/profiles), and hear the song through the headphones. Connect, Disconnect and Forget use [non-public Android APIs](https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces), so those controls are specific to the Android 11 setup I tested.

<div class="media-device-gallery">
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-bluetooth.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-bluetooth.png' | relative_url }}" alt="Installed Bluetooth screen showing Sony WH-1000XM4 connected for music with Disconnect and Forget controls" width="1080" height="2280" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: the integrated Bluetooth screen reports the observed music connection. Audible playback was checked separately.</figcaption>
  </figure>
</div>

### Loading and browsing his music

I set USB file transfer (MTP) as the default from my computer. Windows could then see the phone again after I unplugged and reconnected it. That let me copy music over without putting a USB Settings shortcut on the player.

The app reads the local library through [MediaStore](https://developer.android.com/training/data-storage/shared/media) and plays it through a [Media3 background service](https://developer.android.com/media/media3/session/background-playback). One test library held 498 songs across 37 albums and 21 artists. Scrolling through those albums exposed something I'd missed with a handful of tracks. The app kept extracting the same [embedded cover art](https://developer.android.com/reference/android/media/MediaMetadataRetriever#getEmbeddedPicture()) as covers came back into view. I moved that work off the main thread and cached one thumbnail per album, with limits on memory and disk use. Scrolling felt smoother in the [optimized release build](https://developer.android.com/develop/ui/compose/performance), although the runs had different cache and warm-up conditions.

<div class="media-device-gallery">
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-albums.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-albums.png' | relative_url }}" alt="Installed music library showing the Albums tab, local artwork, album names and a paused mini-player" width="1080" height="2280" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: local album browsing, with artwork read from the music files.</figcaption>
  </figure>
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-now-playing.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-now-playing.png' | relative_url }}" alt="Installed Now Playing screen showing The Gift album artwork, playback, shuffle, repeat and volume controls" width="1080" height="2280" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: an earlier Now Playing layout during the shuffle/repeat checks. Later work adjusted the cover size and added Browse songs.</figcaption>
  </figure>
</div>

Shuffle gave me a way to test the playback controls one tap at a time. Turning it on could put the current song last in the shuffled order, leaving the Next button disabled. I made manual navigation wrap around while keeping the shuffled order intact. On a twelve-song album, I tapped Next until I came back to the song I'd started on. I’d visited every track once. Repeat still controls what happens when a song ends on its own.

Saved playlists survived stopping and reopening the app. The live queue and playback position still need that same treatment after a restart. I also want to try removing an SD card mid-session and see how the player handles a playlist whose files have disappeared.

## Taking a photo and finding it again

I wanted the Camera and Photos tiles to feel like two parts of the same player. I used [CameraX](https://developer.android.com/media/camera/camerax) to save pictures under `DCIM/Daisy/` and built a viewer that opens the original image at a size suited to the screen. Both run inside the restricted app.

The revealing test was to turn off Android's auto-rotate, hold the phone sideways and take a picture. The first version used the display rotation at shutter time and treated the picture as portrait. I changed [capture rotation](https://developer.android.com/media/camera/camerax/orientation-rotation) to follow the orientation sensor. The next landscape photo saved at 4000 × 3000 and opened the right way up in Photos, all while lock task mode stayed active.

<div class="media-device-gallery media-device-gallery-wide">
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-camera-landscape.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-camera-landscape.png' | relative_url }}" alt="Installed camera in landscape with a plain test scene, overlaid shutter and front/rear camera switch" width="2280" height="1080" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: landscape Camera with overlaid controls and a plain test scene.</figcaption>
  </figure>
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-photo-viewer-landscape.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-photo-viewer-landscape.png' | relative_url }}" alt="Installed landscape photo viewer showing a test capture, previous and next controls, and a delete button" width="2280" height="1080" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: the captured test image in the in-app viewer, with edge navigation and Delete.</figcaption>
  </figure>
</div>

One gallery error was especially odd. I could see my pictures underneath a message saying they couldn't be loaded. A refresh had cancelled the previous query, and my error handler had treated [coroutine cancellation](https://developer.android.com/kotlin/coroutines/coroutines-best-practices#watch-out-for-exceptions) as a failed load. Letting cancellation propagate and clearing old errors on success fixed the message in my refresh and sleep/wake tests.

## Adding FM radio

FM was originally something I hoped to explore later. Plugging in wired headphones gave the Xiaomi's tuner an antenna, and I could hear a broadcast using the phone's existing FM software. Its Saved recordings screen could also lead into other apps, so I took the native tuner off the list of apps the player could open.

I kept the ROM's FM engine and built a Radio tile around it. The controls live inside Media Device and talk to a separate service through authenticated [Android Interface Definition Language (AIDL) calls](https://developer.android.com/develop/background-work/services/aidl). I could tune, seek and save favorites. Starting FM pauses music, and starting music stops FM. A separately signed test client was refused access to all nine control methods. The service depends on native libraries from this phone, which I kept out of the repository.

<div class="media-device-gallery">
  <figure>
    <a href="{{ '/assets/images/android-media-device/prototype-fm-tuner.jpg' | relative_url }}"><img src="{{ '/assets/images/android-media-device/prototype-fm-tuner.jpg' | relative_url }}" alt="Browser prototype of the FM Tuner design with a frequency ruler and compact favorite cards" width="402" height="782" loading="lazy" decoding="async"></a>
    <figcaption>Browser prototype: the selected Tuner concept, rendered in its light appearance. Playing and the station labels are sample state.</figcaption>
  </figure>
  <figure>
    <a href="{{ '/assets/images/android-media-device/app-fm-tuner.png' | relative_url }}"><img src="{{ '/assets/images/android-media-device/app-fm-tuner.png' | relative_url }}" alt="Installed dark FM Tuner screen at 102.2 MHz, stopped, showing a saved Comercial favorite and wired-headphone antenna reminder" width="1080" height="2280" loading="lazy" decoding="async"></a>
    <figcaption>On the phone: the implemented Tuner in the app theme. Playback is stopped in this capture; Comercial is a saved label, not received RDS text.</figcaption>
  </figure>
</div>

The tuner is a working prototype. The station name in the screenshot is a label I saved myself. Although the driver reports RDS support, I've yet to receive a decoded station name or RadioText. I also want to listen through each output to check the speaker and headphone switching I saw in the routing logs.

## Where the project stands

I set out to give my son an offline media device like the one he already had, and that's what the Xiaomi is now. He can copy an album from the computer to the player on his own, find it in his music library and listen through his headphones. When he wants to take a photo, he can switch straight to the camera.

The polish I'd prioritize is the kind he'd notice, like picking up a song where he left off after a restart, or reconnecting his headphones after a long listening session. On my side, I'll keep checking the pairing exception and what happens if the player and its recovery service both stop. Beyond that, I'd like his requests to guide what I add next.

## References and resources

The Android documentation below covers the APIs and techniques discussed in this post across multiple Android versions; the device results above apply to the tested Android 11 / LineageOS installation. The community report links back to the problem that prompted the project.

### Language and interface

- [Kotlin and Android](https://developer.android.com/kotlin)
- [Jetpack Compose](https://developer.android.com/compose)
- [Compose performance](https://developer.android.com/develop/ui/compose/performance)

### Device control and Bluetooth

- [DevicePolicyManager API reference](https://developer.android.com/reference/android/app/admin/DevicePolicyManager)
- [Lock task mode](https://developer.android.com/work/dpc/dedicated-devices/lock-task-mode)
- [Dedicated devices cookbook](https://developer.android.com/work/dpc/dedicated-devices/cookbook)
- [Android Debug Bridge (adb)](https://developer.android.com/tools/adb)
- [Companion device pairing](https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing)
- [Bluetooth profiles](https://developer.android.com/develop/connectivity/bluetooth/profiles)
- [Restrictions on non-SDK interfaces](https://developer.android.com/guide/app-compatibility/restrictions-non-sdk-interfaces)

### Music, camera and storage

- [Access media files from shared storage](https://developer.android.com/training/data-storage/shared/media)
- [Background playback with MediaSessionService](https://developer.android.com/media/media3/session/background-playback)
- [MediaMetadataRetriever.getEmbeddedPicture()](https://developer.android.com/reference/android/media/MediaMetadataRetriever#getEmbeddedPicture())
- [CameraX overview](https://developer.android.com/media/camera/camerax)
- [CameraX use case rotations](https://developer.android.com/media/camera/camerax/orientation-rotation)
- [Coroutine best practices: exceptions and cancellation](https://developer.android.com/kotlin/coroutines/coroutines-best-practices#watch-out-for-exceptions)

### Radio backend communication

- [Android Interface Definition Language (AIDL)](https://developer.android.com/develop/background-work/services/aidl)

### Community report

- [Reddit: MP3-player corruption report](https://www.reddit.com/r/mp3players/comments/1eyssyd/comment/ll90tor/)

### Firmware investigation

- [smartlink_flash](https://github.com/ilyakurdyukov/smartlink_flash) — the tool I used to extract the original MP3 player's firmware.
