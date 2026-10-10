# SUV Music — Android Music Player

A native Android music player built with **Kotlin**, **Media3/ExoPlayer**, **Room** and
**MediaStore**. Dark automotive-style UI with a big orange transport control — made for
car dashboards and long drives.

## Features

### Core playback
- 📂 **Scans local storage** via `MediaStore` — every audio file on the device
- ▶️ **Real playback engine** — ExoPlayer (Media3), handles all common formats
- 🎧 **Background playback** — foreground `MediaSessionService` keeps audio alive
- 🔔 **Lock-screen / notification controls** — automatic, and **Bluetooth car controls work out of the box**
- ⏯️ **Play / pause / next / prev** + seek bar with live time
- 🔀 **Shuffle** — toggles ExoPlayer's shuffle mode
- 🔁 **Repeat** — cycles off → all → one
- 🔇 **Audio focus** handling — pauses for calls, ducks, resumes
- 🎛️ **Headphone unplug** auto-pause (`handleAudioBecomingNoisy`)

### Library management
- 🖼️ **Album art** — loaded per-row and in the now-playing card via Glide
- ⭐ **Favourites** — one-tap star, persisted in Room
- 📁 **Folder browsing** — browse your music by actual directory
- 📝 **Playlists** — create playlists, add songs via long-press
- 🔍 **Search** — live filter across title / artist / album / folder
- 🗂️ **Tabbed views** — All songs · Folders · Favourites · Playlists
- ▶️ **Now-playing highlight** — the current track is tinted orange in the list

### Audio processing
- 🎚️ **Full equalizer** — per-band sliders built dynamically (5 or 10 bands depending on device), plus hardware presets
- 🔊 **Bass boost**, **Virtualizer (3D)**, **Loudness enhancer**
- 🔄 **EQ reset**
- 😴 **Sleep timer** — 5/15/30/45/60/90 min, counts down live in the UI then pauses

## Project structure

```
suvmusic/
├── settings.gradle
├── build.gradle                    # AGP + Kotlin + KSP plugin versions
├── gradle.properties
└── app/
    ├── build.gradle                # dependencies
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/suvmusic/player/
        │   ├── MainActivity.kt             # UI, tabs, search, controller wiring
        │   ├── PlaybackService.kt          # foreground ExoPlayer service
        │   ├── MusicRepository.kt          # MediaStore scanner + Song model + folder grouping
        │   ├── SongAdapter.kt              # SongAdapter + FolderAdapter
        │   ├── audio/
        │   │   ├── AudioEffects.kt         # EQ, bass, virtualizer, loudness
        │   │   └── SleepTimer.kt           # countdown + auto-pause
        │   ├── data/
        │   │   └── MusicStore.kt           # Room entities, DAO, database, repository
        │   └── ui/
        │       ├── EqualizerDialog.kt      # dynamic band sliders
        │       ├── SleepTimerDialog.kt     # duration picker
        │       └── SongActionDialog.kt     # long-press menu
        └── res/
            ├── layout/                     # activity_main, item_song, item_folder,
            │                               #   dialog_equalizer, item_eq_band, item_playlist
            ├── drawable/                   # 17 vector icons + shape backgrounds
            ├── values/                     # colors, strings, themes + ChipTab styles
            └── mipmap-anydpi-v26/          # adaptive launcher icons
```

**~2,150 lines** across 10 Kotlin files, 6 layouts, 17 drawables.

## Build it

Requires **JDK 17** and **Android SDK 34**.

```bash
cd suvmusic
echo "sdk.dir=/path/to/Android/Sdk" > local.properties
gradle wrapper --gradle-version 8.7
./gradlew assembleDebug
# → app/build/outputs/apk/debug/app-debug.apk

adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Or open the folder in **Android Studio** → *Run*.

## Permissions

| Android | Permission | Why |
|---|---|---|
| 13+ | `READ_MEDIA_AUDIO` | read local music |
| ≤12 | `READ_EXTERNAL_STORAGE` | read local music |
| all | `FOREGROUND_SERVICE_MEDIA_PLAYBACK` | keep playing in background |
| all | `WAKE_LOCK` | playback with screen off |
| 13+ | `POST_NOTIFICATIONS` | media notification |

## How the pieces fit together

```
MainActivity ──binds──▶ MediaController ──▶ PlaybackService (owns ExoPlayer)
     │                                              │
     │  audioSessionId                              │ MediaSession
     ▼                                              ▼
 AudioEffects ◀──────────────────────────  lock-screen / Bluetooth controls
     │
 MusicStore (Room)  ── favourites, playlists, playlist_songs
```

The **service** owns the one true `ExoPlayer`; the **activity** only holds a lightweight
`MediaController` bound in `onStart()` and released in `onStop()`, so rotation and
backgrounding never leak the player.

## Extending it further

- **Crossfade / gapless** → `ExoPlayer.setAudioAttributes` + `DefaultLoadControl` tuning
- **Music visualiser** → `Visualizer` API bound to the same `audioSessionId`
- **Lyrics** → parse `.lrc` sidecar files in `Song.path`'s folder
- **Widgets** → `MediaSession` already provides the token for a `MediaControlView`
- **Car app** → add `androidx.car.app` and a `CarAppService` reusing the same session
- **Download from URL** → `MediaItem.fromUri(httpUrl)` — ExoPlayer streams directly

## Notes

- No `INTERNET` permission is requested — this player is fully offline by design.
- Audio effects are device-dependent; every call is wrapped in `runCatching`, and the
  EQ dialog shows a friendly message when a device exposes no effects at all.
- `KSP` is used for both Room and Glide annotation processing (faster than KAPT).