# Buzz! controller lights for PCSX2 on Windows

These unofficial source patches forward the emulated Buzz! controller's four red-light commands to a physical USB set. ChatGPT generated the implementation. A human tester verified the lights during gameplay on v2.8.2 and v2.9.181 with a wired Sony Buzz! set identified as USB `054C:0002`.

| PCSX2 release | Patch | Validation |
| --- | --- | --- |
| [v2.8.2](https://github.com/PCSX2/pcsx2/releases/tag/v2.8.2), stable | [pcsx2-2.8.2-luces-buzz.patch](pcsx2-2.8.2-luces-buzz.patch) | Windows x64 build and in-game lighting confirmed |
| [v2.9.181](https://github.com/PCSX2/pcsx2/releases/tag/v2.9.181), nightly | [pcsx2-2.9.181-luces-buzz.patch](pcsx2-2.9.181-luces-buzz.patch) | Windows x64 build and in-game lighting confirmed |

Both tags contain the same upstream Buzz! source files, so the version-labeled patches contain the same diff.

The Windows build targets one attached set with USB ID `054C:0002`. Keep the normal PCSX2 button mappings and enable the optional **Physical Buzz Lights** setting for the virtual Buzz! device. Use the Windows HID driver for the controller. If Windows reports a USB hub error, follow the official [Buzz Controller setup guide](https://pcsx2.net/docs/configuration/controllers/#windows-setup).

## Build on Windows

1. Download the patch for your exact release. Run the following from the folder containing the patch and set `$version` to either `2.8.2` or `2.9.181`.

   ```powershell
   $version = '2.8.2'
   $patch = (Resolve-Path "pcsx2-$version-luces-buzz.patch").Path
   git clone --depth 1 --branch "v$version" https://github.com/PCSX2/pcsx2.git "pcsx2-$version"
   Set-Location "pcsx2-$version"
   git apply --check $patch
   if ($LASTEXITCODE -ne 0) { throw 'The patch does not match this checkout.' }
   git apply $patch
   ```

2. Install Visual Studio 2022 17.14 or newer with the C++ x64 build tools and Windows SDK. Extract the PCSX2 Windows dependencies beside the solution so that `deps` is at the source root. The tested dependency archive contained Qt 6.12.0 and had SHA-256 `E1EC4FB9F201D22A64F3DA3390A39E65089DF7030C2C5951FE735180823667C5`. The published archive is refreshed over time. [PCSX2 build guide](https://pcsx2.net/docs/advanced/building/#building-on-windows) · [Windows dependency releases](https://github.com/PCSX2/pcsx2-windows-dependencies/releases)

3. For **v2.8.2** with that dependency archive, move these five folders out of the source tree's `deps/include` to let the Visual Studio project use the compatible FFmpeg 8.0 headers bundled in `3rdparty/ffmpeg/include`: `libavcodec`, `libavformat`, `libavutil`, `libswresample`, and `libswscale`.

4. From the source root in a Visual Studio Developer PowerShell, build **Release | x64**. Set the same value of `$version` if you opened a new shell.

   ```powershell
   $solution = if ($version -eq '2.8.2') { 'PCSX2_qt.sln' } else { 'PCSX2_qt.slnx' }
   msbuild $solution /m /t:pcsx2-qt /p:Configuration=Release /p:Platform=x64
   ```

5. Extract the matching official PCSX2 portable archive into a separate folder. Copy the build's `bin/pcsx2-qtx64.exe` into it together with the top-level runtime DLLs from `bin` and the matching `bin/QtPlugins` directory. Keep the FFmpeg runtime DLLs shipped with the corresponding official portable archive. Run `pcsx2-qtx64.exe -portable` so the test copy stores its settings separately. Configure it with your own PS2 BIOS and Buzz! game.

## Test the lights

1. Connect one compatible Buzz! set and close any browser-based buzzer tester.
2. In **Settings → Controllers → USB Port 1**, select **Buzz Controller**, map the buttons, and enable **Physical Buzz Lights** in the device settings.
3. Start a round that prompts players to buzz in. Check that the red lights follow the game's prompts.

An official update installed over the patched executable will replace it. Keep the test copy separate and apply the patch for the exact version you build.

## Source and license

The patches modify the Buzz! implementation from [PCSX2](https://github.com/PCSX2/pcsx2). The upstream copyright and license notices remain in the source. See [COPYING.GPLv3](COPYING.GPLv3).
