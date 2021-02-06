# DCS on Linux

DCS world can run on linux through wine and proton, though it does take some
work to get running. The game has two distribution methods: standalone and
steam. Both have worked successfully, though often one will be broken and the
other work, If one fails it can be a good idea to try the other

The game also has two versions: stable and open beta. Open beta is what is used
by most MP servers, and stable is updated less frequently (the advantage of
stable for us is that we don't have to work around linux bugs every 2 weeks
when a new OB is released)

Thanks to everyone who has helped getting the game running and debugging issues
in the [proton issue
tracker](https://github.com/ValveSoftware/Proton/issues/1722). Unfortunately,
workarounds easily get burried there, so I decided to create this document with
known up to date methods for getting things to work.

## Getting it working through lutris

An easy way to get started is to use Lutris. Standalone has two install scripts
on lutris which may just work out of the box:
https://lutris.net/games/dcs-world/

## Getting it working manually

Both versions need some winetricks applied. For standalone, use `winetricks`,
for proton use `protontricks 223750`

Start the game once first to create the prefix, then run
```
<tricks command> vcrun2017 corefonts xact d3dcompiler_43
```

This should be all it takes to get standalone working

### Fixing Steam version permanent crashing

If your game crashes in the steam version, it will permanently fail to start
after that. To fix that: remove `drive_c/windows/system32/lsteamclient.dll`
which was created in the crash, and the game should start back up fine.


### Open Beta (updated for 2.5.6.59398)

For now, this guide assumes you use the standalone version. The steam version
may also work, but I have not tested it in a while. Currently, wine 6.0 rc1 or
the lutris version of that release are what work best but other wine versions
may also work.

First, some variables to avoid repetition:

- `$USERNAME`: refers to the wine user. On standalone, this is your normal
  username and on steam it is `steamuser`
- `$INSTALL_DIR`: the location in program files where the game is installed.
  On standalone: `drive_c/Program Files/Eagle Dynamics/DCS`. or `DCS World OpenBeta` On steam, it's
  `/home/frans/.local/share/Steam/steamapps/common/DCSWorld`
- `$CONFIG_DIR`: the place where user config stuff is stored
  `drive_c/users/$USERNAME/Saved Games/DCS<possibly openbeta>`
- `$LOG`: the game log file `$CONFIG_DIR/Logs/dcs.log`

For standalone, if the game crashes before showing the login screen. You need
to add a "dll override" for `wbemprox`. In lutris, you can do so under "runner
options". For wine and steam proton, you can do so using the `WINEDLLOVERRIDES`
flag https://wiki.winehq.org/Wine_User's_Guide#WINEDLLOVERRIDES.3DDLL_Overrides

With that change, you should be able to log in but once the game starts you
will see a black screen. To fix this, create a symlink from
`$INSTALL_DIR/bin/webrtc_plugin.dll` to `$INSTALL_dir/webrtc_plugin.dll`

The game should now start

You may also see a crash when loading a mission. This might be caused by a
Arial missing font which can not be distributed with wine.

## Known issues and fixes

If things go wrong, the primary thing to look for is the game log
`drive_c/users/$USERNAME/Saved Games/DCS<possibly openbeta>/Logs/dcs.log`.
After crashes, the crash reporter will spam a bit about various DLLs being used
recently, and just before that, the cause of the crash should be visible.

Sometimes crashes happen before the game gets far enough to create a log file.
Then your best bet is to read the proton output. In both lutris and steam, you can easily get
this by starting lutris or steam from a terminal.

If you can't find an issue, or found a solution for one, please discuss it in
the [proton issue](https://github.com/ValveSoftware/Proton/issues/1722)

### White smoke and some other particles renders weirdly

This is a long standing issue, most likely related to texture loading. Luckily
it is just a visual artefact that can be (largely) ignored


### F16 RWR shows a opaque square on the RWR over the priority contact

This issue occurs because some textures fail to load for an unknown reason. The
fix is simple: open the file
`${INSTALL_DIR}/Mods/aircraft/F-16C/Cockpit/IndicationResources/RWR/indication_RWR.tga`
with an image editor (gimp or krita have been used successfully), then just
re-export the file. Now the RWR should render correctly


### Missing multiplayer server list

For a few 2.5.6 versions, the server browser did not work, and neither did
directly connecting to servers using connect by IP. However, there is a
workaround.

Edit `$INSTALL_DIR/MissionEditor/modules/mul_password.lua`. Find the function `onChange_btnOk` and add the
line `onlyConnect = true` to the start of the function like so.

```lua
function onChange_btnOk()  
    onlyConnect = true -- This line was added
	if onlyConnect == true then
	-- ...
end
```

Now you should be able to use the connect by IP button to join servers, but the
server list is still broken. Luckily, a server list is available if you log in
on https://www.digitalcombatsimulator.com/, and from there you can get the IP
of servers.

### Crash on F10

For many DCS versoins and or wine versions, if you press F10, by default the
binding to bring up the map, the game will crash ("permanently" on steam, see
fixing steam permanent crashing for a fix).  Luckily, the problem is with the
F10 key itself, not the map, so rebind it to something else you see fit. The
same applies for the communication menu

### Module disabled by user

You probably won't run into this, but if you do, there is a fix.

One of your modules is missing, it is not shown in the list at the bottom of
the main menu, and you can't use it. On standalone, check if it is enabled in
the module manager. On steam however, things are a bit more tricky. If you
copied your configs between standalone and steam, module manager disabled mods
will be disabled in steam too. This information is stored in
`$CONFIG_DIR/enabled.lua` or something similar. Remove it to fix the issues



## Other software

While not included in DCS, here are some resources for getting external
software often used by the game running.

### SRS

[SRS](http://dcssimpleradio.com/) is used by a lot of multiplayer servers. It
too works with some tweaks

Install the game plugin by following the instructions in the SRS readme.

*Note* As of SRS 19.0.1, this method no longer works. As a replacement, I have
a custom SRS client that *kind of* works here https://gitlab.com/TheZoq2/srsrs.

It's easiest to run SRS in its own prefix. Create one, and then run `winetricks
dotnet452 win10` in that prefix. Now you can start `SR-ClientRadio.exe` from
the downloaded files.

Credit: https://github.com/ciribob/DCS-SimpleRadioStandalone/issues/409.


### Headtracking via opentrack

Opentrack can emulate a gamepad which is read and can be mapped to the
corresponding controls in the game. This should work out of the box, simply
select `lubudev joystick receiver` as the output in opentrack.

It is also possible to use opentrack using the freetrack protocol. Credit to @akp  for writing this.
This Doesn't
require you to bind headtracking for every aircraft and works well with other
games that do not support binding axes to head movement.

https://github.com/opentrack/opentrack Opentrack works out of the box with
libevdev joystick output, however this requires you to bind headtracking for
every aircraft (and doesn't play well with Il-2 BoX or Falcon BMS).

A better option is to enable wine (freetrack and npclient) output instead of
joystick axis output.

https://aur.archlinux.org/packages/opentrack/ If you are building opentrack
from the AUR, you can modify the PKGBUILD.  Replace line 34
`-DSDK_WINE_PREFIX=/ \` with `-DSDK_WINE=ON/ \`

Otherwise, download the source code from the github and follow these
instructions: https://github.com/opentrack/opentrack/wiki/Building-on-Linux
**After you cd into the directory, run `ccmake .`, press c to configure, turn
ON SDK_WINE, c to configure and g to generate.**

DCS still requires `HeadTracker.dll` in the bin directory for opentrack to
function.  Download Eagle Dynamics API interface DLL (64-bit) from
http://facetracknoir.sourceforge.net/information_links/download.htm

You must open opentrack and start tracking before you launch DCS. Be sure to
point the output to the correct wine/proton prefix. In addition, you'll need to
launch DCS with WINEESYNC=1 or WINEFSYNC=1 if you enable those in the wine
output settings.

![Screenshot_20201222_001754](https://user-images.githubusercontent.com/10890625/102798194-b5b75a80-43eb-11eb-843c-90ef83a1c170.png)

Context: https://github.com/ValveSoftware/Proton/issues/1722#issuecomment-749061952

### Headtracking via Linuxtrack

In the case the Opentrack fails to work (@bradley-r observed that running Opentrack's
Wine plugin prevented DCS from launching - likely due to the sharing of the same 
prefix) or you wish to try an alternative, Linuxtrack (https://github.com/uglyDwarf/linuxtrack/) 
offers similar functionality. 

Begin by installing the universal Linux package (https://github.com/uglyDwarf/linuxtrack/wiki/universal-Linuxtrack-package).
Once complete, run `ltr-gui` and under the 'Misc' tab, select (re)install TrackIR firmware.) Linuxtrack
will attempt to complete this task for you, but, at time of writing, the TrackIR download links have changed, so
you may need to do this manually. Download the latest TrackIR firmware, install it to your default (or
temporary) prefix, then select 'Extract from unpacked'.

![img1](https://user-images.githubusercontent.com/43189454/107122801-ccc5e500-6891-11eb-9a71-6c2a89fdf3f1.png)

Navigate to the prefix you used, and select the TrackIR 5 folder under `/drive_c/Program Files (x86)/NaturalPoint/`. 
Once done, you will be prompted to install the Wine-side components; select the prefix DCS is installed under
(only standalone has been tested.) `ltr-gui` can now be closed, and provided Linuxtrack is running
(and has been configured), use the `FreeTrackTester.exe` present in the second prefix `/drive_c/Program Files (x86)/Linuxtrack/`. You should see the values changing, and thus controlling the view in-game.

![img2](https://user-images.githubusercontent.com/43189454/107122784-b029ad00-6891-11eb-8e0b-41d06e706e6d.png)

Note that `HeadTracker.dll` need not be present as Linuxtrack replicates TrackIR directly (in the case of DCS, at least.)
