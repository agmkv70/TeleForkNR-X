# TeleForkNR (Telegram X)

TeleForkNR changes on top of Telegram X. They live on the `teleforknr` branch, and `main` stays as upstream.

## What differs from upstream

- **Branding:** the app name is TeleForkNR and the package is `org.teleforknr.x`, using our own API credentials and Firebase project (`teleforknr`). The launcher icon has a small Ukrainian flag. Language packs can't rename the app back.
- **Chat list:** avatars are 30% smaller, and the text and separators follow them. A protected chat's preview doesn't block screenshots of the list. The chat itself still blocks them.
- **Messages (classic layout):** the body starts below the avatar at the left margin and wraps close to the right edge. The name row stays beside the avatar.
- **Reactions:** hidden by default (Settings → Stickers & Emoji → Hide reactions). Each chat's ⋮ menu has **Show reactions** to allow them in that chat.
- **Channels:** the ⋮ menu has **Hide comments**, per channel. The Follow, Mute/Unmute and Discuss bottom bars are hidden by default, with their actions in the ⋮ menu. **Show bottom bar** brings them back.

Search the code for `TeleForkNR` to find each change.

## Build setup (Windows)

1. Get the submodules: `git submodule update --init --recursive --depth 1`
2. **Line endings:** the system Git here uses `core.autocrlf=true`, and CRLF breaks the native builds (libvpx assembly, for example). Set `core.autocrlf=false` in this repo and in every submodule, then check the files out again:
   ```sh
   git config core.autocrlf false
   git submodule foreach --recursive 'git config core.autocrlf false'
   ```
3. **Tools:**
   - [MSYS2](https://www.msys2.org) installed at `C:/msys64`, with `pacman -S --needed git mingw-w64-ucrt-x86_64-git-lfs perl make diffutils`
   - Android SDK with NDK `27.3.13750724`, build-tools `37.0.0` and platform `android-37.2`
   - **JDK 25**, needed by the build itself, not just as a toolchain. If `JAVA_HOME` points somewhere else, set it for the build command.
4. **`local.properties`** is gitignored and holds:
   ```properties
   sdk.dir=C:/Users/nik/AndroidDev/Sdk
   msys2.dir=C:/msys64
   app.id=org.teleforknr.x
   app.name=TeleForkNR
   telegram.api_id=...
   telegram.api_hash=...
   keystore.file=C:/Users/nik/AndroidDev/keys/teleforknr-keystore.properties
   tgx.extension=none
   ```
   `app/google-services.json` has to contain a client for the package in `app.id`, or the build fails.

## Building

```powershell
$env:JAVA_HOME = "C:\Users\nik\AndroidDev\jdk\jdk-25.0.4.1+1"
.\gradlew.bat assembleLatestUniversalDebug   # debug, all CPU types
.\gradlew.bat assembleLatestArm64Release     # signed release, arm64
```

APKs are written to `app/build/outputs/apk/<variant>/<buildType>/`. The first full build takes about 10 minutes because of the native code. Later builds take seconds.

## Release signing

The key and its password file are outside the repo, in `C:/Users/nik/AndroidDev/keys/`:

- `teleforknr.jks`, the signing key
- `teleforknr-keystore.properties`, which holds `keystore.file`, `keystore.password`, `key.alias` and `key.password`

**Never commit these files, and keep a backup somewhere else.** Without this key, no future build can update installed copies. Users would have to uninstall and log in again.

A debug build and a release build are signed with different keys, so neither can install as an update over the other. Uninstall one before installing the other. Uninstalling logs you out and clears TeleForkNR's local settings.

## Installing on a phone

`adb install -r <apk>`. On Xiaomi/MIUI, turn on Developer options → **USB debugging** and **Install via USB**. Then accept the prompts on the phone.
