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


### Open Beta (updated for 2.5.6.50979)

First, some variables to avoid repetition:

- `$USERNAME`: refers to the wine user. On standalone, this is your normal
  username and on steam it is `steamuser`
- `$INSTALL_DIR`: the location in program files where the game is installed.
  On standalone: `drive_c/Program Files/Eagle Dynamics/DCS`. or `DCS World OpenBeta` On steam, it's
  `/home/frans/.local/share/Steam/steamapps/common/DCSWorld`
- `$CONFIG_DIR`: the place where user config stuff is stored
  `drive_c/users/$USERNAME/Saved Games/DCS<possibly openbeta>`
- `$LOG`: the game log file `$CONFIG_DIR/Logs/dcs.log`

If the game crashes on startup, check `$LOG`. If it says something about
`VoiceChat` or `webrtc_plugin.dll`, the built in voice chat system (which
barely anyone is using) fails to load.

There are a few workarounds for this. If you only want to play single player,
try one of the following:

- Replace `VoiceChat/bin/VoiceChat.dll` with `CaptoGlove/bin/CaptoGlove.dll` in
  `$INSTALL_DIR/CoreMods/services`
- Edit the LUA to stop loading voice chat
  https://github.com/ValveSoftware/Proton/issues/1722#issuecomment-601839315

If you want multi player, the above will make the integrity check fail.
Luckily, it does not test the dll which causes the crash
`$INSTALL_DIR/bin/webrtc_plugin.dll` so we can modify it to stop running.

See https://github.com/ValveSoftware/Proton/issues/1722#issuecomment-606780304 for instructions

*Standalone should now start*

If the steam version crashes when done loading a game, check `$LOG`. If it says
something about the arial font missing, you need to replace it with a working
version. Arial can not be distributed see
https://law.stackexchange.com/a/14834. However, as linked in that stackexchange
post, you can replace it with [Arimo](https://www.fontsquirrel.com/fonts/arimo)
in `drive_c/windows/Fonts`


### Multiplayer

As of a few versions ago, the server browser does not work, and neither does
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





## Known issues and fixes

If things go wrong, the primary thing to look for is the game log
`drive_c/users/$USERNAME/Saved Games/DCS<possibly openbeta>/Logs/dcs.log`.
After crashes, the crash reporter will spam a bit about various DLLs being used
recently, and just before that, the cause of the crash should be visible.

Sometimes crashes happen before the game gets far enough to create a log file.
Then your best bet is to read the proton output. In stea, you can easily get
this by starting steam from a terminal.

If you can't find an issue, or found a solution for one, please discuss it in
the [proton issue](https://github.com/ValveSoftware/Proton/issues/1722)

### Crash on F10

If you press F10, by default the binding to bring up the map, the game will
crash ("permanently" on steam, see fixing steam permanent crashing for a fix).
Luckily, the problem is with the F10 key itself, not the map, so rebind it to
something else you see fit. The same applies for the communication menu


### White smoke renders weirdly

This is a long standing issue, most likely related to texture loading. Luckily
it is just a visual artefact that can be (largely) ignored


### F16 RWR shows a opaque contact on the RWR

This is likely caused by the symbol indicating lock-on being broken. No fix is
known

### Running SRS

[SRS](http://dcssimpleradio.com/) is used by a lot of multiplayer servers. It
too works with some tweaks

Install the game plugin by following the instructions in the SRS readme.

*Note* As of SRS 19.0.1, this method no longer works. As a replacement, I have
a custom SRS client that *kind of* works here https://gitlab.com/TheZoq2/srsrs.

It's easiest to run SRS in its own prefix. Create one, and then run `winetricks
dotnet452 win10` in that prefix. Now you can start `SR-ClientRadio.exe` from
the downloaded files.

Credit: https://github.com/ciribob/DCS-SimpleRadioStandalone/issues/409.

### Module disabled by user

You probably won't run into this, but if you do, there is a fix.

One of your modules is missing, it is not shown in the list at the bottom of
the main menu, and you can't use it. On standalone, check if it is enabled in
the module manager. On steam however, things are a bit more tricky. If you
copied your configs between standalone and steam, module manager disabled mods
will be disabled in steam too. This information is stored in
`$CONFIG_DIR/enabled.lua` or something similar. Remove it to fix the issues


