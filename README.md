# MSP-OSD - Full OSD / Displayport OSD

This package gives you support for full flight controller driven OSD in line with those on analog + other HD systems.

Technically - it takes MSP DisplayPort (so-called "canvas" although this is a misnomer as there is another Betaflight "canvas" mode for Pixel OSDs) messages through UDP and renders them to a framebuffer overlaid under the DJI `dji_glasses` menu system.

A custom `font.bin` package may be placed on the root of the goggles SD card, at which point it will override the font in `/blackbox/font.bin`.

SFML (PC/Mac development) and DJI Goggles viewports are available, as well as a *mux* for the Air Unit / Vista, which creates a *pty* and provides filtered MSP access, and reroutes DisplayPort messages to UDP.

# Setup and Installation

## Easy Installation

* Install WTFOS from https://fpv.wtf. WTFOS must be installed on both the goggles and each AU/Vista.
* Install the msp-osd package on each device using WTFOS.
* Reboot.

## Flight Controller Setup

* Ensure that the correct UART is set to use MSP
* Enable MSP DisplayPort

### Betaflight - Since 4.4

Betaflight 4.4 has added native support for OSD in HD aspect ratio.

#### Configure With Preset

There is a preset available: "OSD for Fpv.wtf, DJI O3, Avatar HD", once applied you can then skip ahead to configuring in the OSD tab.

#### Configure Manually

Or to configure manually, first enter the following commands in the CLI tab:

```
set osd_displayport_device = MSP
set vcd_video_system = HD
save
```

And then in the Ports tab, select the peripheral "VTX (MSP + DisplayPort)" for the UART your Vista/Air unit is connected to.

![Ports Tab Setting](/docs/img/ports-vtx.png)

Afterwards, you can configure the OSD elements as normal in the OSD tab.

#### Troubleshooting wrong grid size in BF 4.4 Configurator

It is recommended to enable [compressed transmission](#compressed-transmission) with BF 4.4; it will soon become the default. It removes/avoids ordering issues between FC/AU/Goggles bootup - the AU has to tell the FC the grid size it supports.

If you don't want to / can't do this - try rebooting your goggles, then reboot your AU.

### Betaflight - 4.3 or Before

We have a configurator preset available - "FPV.WTF MSP-OSD", just be sure to pick the UART your Vista/Air unit is connected to.

#### Or to configure manually

On *Betaflight*, this is done using the following commands in the CLI tab:

```
set osd_displayport_device = MSP
set displayport_msp_serial = <ConfiguratorUART - 1>
set vcd_video_system = PAL
save
```

Eg.: If the Betaflight Configurator says your DJI VTx is attached to UART2, the value for **<ConfiguratorUART - 1>** is **1** - so you would use ```set displayport_msp_serial = 1```.
Test if the value is correct by typing `save` and after the reboot `get displayport_msp_serial` This command should return the value you set it to. If it returns -1 (and that was not the value you set) then the value was not correct.

For Betaflight - ensure you set the Video Format to PAL or Auto in the OSD tab - otherwise you don't get access to the whole OSD area. Note that currently BF Configurator hides these options once you switch to MSP for OSD; the last command above should have done this for you.

#### Softserial

We don't recommend connecting via soft serial; results have been poor - it gives slow/laggy/inconsistent behaviour. But some users have reported it being usable, so if for whatever reason this is your only option, read on.

If you have connected the Vista/Airunit to a softserial port run the `serial` command to list serial ports
Use the value after _serial_ with set `displayport_msp_serial` but do **not** subtract 1 from the value. E.g.:
```
# serial
serial 20 1 115200 57600 0 115200
serial 0 64 115200 57600 0 115200
serial 1 0 115200 57600 0 115200
serial 30 1 115200 57600 0 115200
# set displayport_msp_serial = 30
```

#### FakeHD

Betaflight's (before 4.4) OSD supports a 30 * 16 Grid, which looks large/blocky when displayed in the DJI Goggles.

For versions of Betaflight before 4.4 (or other FC firmwares without HD support), as a workaround, we have a mode called "FakeHD". It chops up the OSD into sections and positions them evenly around an HD grid (with gaps between) - the way this is done is configurable, explained below. This then allows the use of smaller fonts - so it looks nicer/more in keeping with the built in Goggles OSD (but you still only have 30 columns / 16 rows to configure.... and you have the gaps to contend with).

A diagram to help...

| Before (in Configurator) | After (in Goggles) |
| -------|-------|
|![FakeHD Before](/docs/img/fakehd_before.png "Before")|![FakeHD After](/docs/img/fakehd_after.png "After")|

##### To configure/enable:

Visit https://fpv.wtf/package/fpv-wtf/msp-osd with your goggles connected, and check "Fake HD"

Optionally, place custom fonts in the root of your sd card, using the names `font_bf_hd.bin` / `font_bf_hd_2.bin` (NB: FakeHD no longer uses font_hd.bin / font_hd_2.bin)

Configuration of the grid is also possible; see below.

No air unit/vista config is required.

##### Menu Switching - Getting rid of gaps when displaying Menu / Post Flight Stats + displaying centered:

In order to have menus (accessible in Betaflight using stick commands) and post-flight stats appear in the center of the screen while using FakeHD, rather than having gaps + looking broken, you should set up menu switching.

FakeHD can use the presence/absence of a character in the OSD as a switch to indicate when you are in regular OSD mode or in the menu/stats and switch to centering temporarily when needed.

By default, the `Throttle Position` icon is used (character 4) - but you can set any character you want. It needs to be something that doesn't flash or change in the regular OSD, and ideally (but not essential) something that is never in the menu/post flight stats. The icons next to various elements are obvious choices here. You can set this using the `fakehd_menu_switch` configuration parameter.

Betaflight has a list here: https://github.com/betaflight/betaflight/blob/master/docs/osd.md


If you want to use FakeHD with some other Flight Controller, you will need to find an appropriate icon. (Let us know - we can include the information here).

Finally, if you don't have anything in your OSD that works for menu switching, you can hide the menu switching character and the subsequent 5 characters, allowing you to add the `Throttle Position` element but not have to see it on screen. This is enabled by setting `fakehd_hide_menu_switch` to true.

Notes:

 - Because of this switching feature, if you change to a different quad or OSD config (specifically the switch element is in a different place), FakeHD will center - you will need to reboot your Goggles to get it to recognise the switch element in a different location.

 - Also because of this switching, if you are editing OSD in the configurator with the goggles on to preview and you move the switching element around, it will cause the gaps to be disabled and everything to center. The new location of the switching element will be found next time you reboot the goggles and it'll work as normal.

##### I don't want gaps at all...

Set config `fakehd_lock_center` to true and the center locking used for the menu / post flight stats will be enabled permanently (ie: you fly with it as well) - basically it places your 30 x 16 SD grid into the middle of an HD grid, there's never any gaps - see diagram below + compare to diagrams above.

| After/Centered (in Goggles) `fakehd_lock_center` |
|-------|
|<img src="/docs/img/fakehd_centered.png" alt="After / Centered"  height=200 /> |

##### Customising the default FakeHD grid.

By default, FakeHD positions your SD grid into the HD grid as per the before/after diagram above.

If this doesn't work for you for whatever reason, some customisation is available. It's necessarily a little complicated.

Each row can be set to one of:

| Code | Description |
|---|----|
| L | Left aligned, the SD row occupies cell 1-30, no gaps |
| C | Center aligned, the SD row occupies cell 16-45, no gaps |
| R | Right aligned, , the SD row occupies cell 31-60, no gaps |
| W | Split A - Row is split in 3, the FakeHD default, filling cells 1-10, 26-35, 51-60 |
| T | Split B - Row is split in 2, touching the sides - filling cells 1-15 + 46-60 |
| F | Split C - Row is split in 2 and away from the sides - filling cells 11-25 + 36-50 |
| D | DJI Special - Row is centered but pushed a little left; used to posiution the bottom row between the existing DJI OSD elements |

<img src="/docs/img/fakehd_rows.png" alt="Columns"  height=200 />

And then the columns as a whole can be set to one of:

| Code | Description |
|---|----|
| T | Top aligned, OSD occupies rows 1-16  |
| M | Center aligned, OSD occupies cells 4-19, no gaps |
| B | Bottom aligned, , the OSD occupies rows 7-22 |
| S | Split - FakeHD default - split in 3, OSD occupies rows 1 - 5, 9 - 13, 17-22 |

Using the default row config; here's what they all look like:

| T | M | B | S |
| - | - | - | - |
|<img src="/docs/img/fakehd_columns_t.png" alt="T" />|<img src="/docs/img/fakehd_columns_m.png" alt="M" />| <img src="/docs/img/fakehd_columns_b.png" alt="B" />| <img src="/docs/img/fakehd_after.png" alt="S" />|

###### To configure rows

Rows config accepts a 16 character long string; each character configuring it's corresponding row. The default FakeHD config would be set like this:

`fakehd_rows = WWWWWWCCWWWWWWWD`

The characters are case sensitive, but the configurator will reject invalid characters.

###### To configure columns

Columns accepts a single character configuring how the whole grid is aligned. The default would be set like this:

`fakehd_columns = S`

The characters are case sensitive, but the configurator will reject invalid characters.

### INAV

Select "HDZero VTx" or "MSP Display Port" (on newer INAV versions) as the Peripheral. Next, select "HD" in the OSD tab if you'd like to use the HD Canvas.

If the iNav OSD appears garbled at first, try entering the iNav menus using the RC sticks, and then exiting the menus. This will force INAV to switch into HD mode a second time.

It is recommended to enable [compressed transmission](#compressed-transmission) with INAV to avoid issues with the display corrupting - the artifical horizon is the most common element to show this.

### Ardupilot

Please install an Ardupilot Custom or Nightly build from after 8/29/2022 for full functionality. There was one critical bug fix for MSP telemetry not passing through a DisplayPort serial port. Additionally, there were several feature additions including HD support after the last 4.2 release.

Settings:

```
SERIALx_PROTOCOL = 42
OSD_TYPE = 5
```
If you wish to use a Betaflight font instead of an Ardupilot font, you can also set ``MSP_OPTIONS = 4` to allow the use of a Betaflight font.

More info: https://ardupilot.org/plane/docs/common-msp-osd-overview-4.2.html#dji-goggles-with-wtf-osd-firmware

### KISS Ultra

Select MSP on serial and select DJI WTF as canvas dialect. That's it.

### QUICKSILVER

Configure the UART under Digital VTX - see https://docs.bosshobby.com/Configuring-Quicksilver/#setup

## Fonts

We bundle in default fonts for the flight controller variants we support. [Preview images are available here](docs/fonts). Or you can use a custom one...

* Download a font package. See below for known community fonts.
* Rename the files for your desired font to `font_<fc variant>` - see table below for examples or take a look at the `fonts` directory for a template for how the file names should look. (If your FC firmware is not listed below, use the generic filenames)
* Place these four files on the root of your Goggles SD card.
* Reboot.

### FC Specific Font File Names

| Flight controller | SD | HD |
| ----------------- | -- | -- |
| Betaflight       | `font_bf.bin`, `font_bf_2.bin` | `font_bf_hd.bin`, `font_bf_hd_2.bin` |
| INAV       | `font_inav.bin`, `font_inav_2.bin` | `font_inav_hd.bin`, `font_inav_hd_2.bin`|
| Ardupilot       | `font_ardu.bin`, `font_ardu_2.bin` | `font_ardu_hd.bin`, `font_ardu_hd_2.bin`|
| KISS Ultra       | `font_ultra.bin`, `font_ultra_2.bin` | `font_ultra_hd.bin`, `font_ultra_hd_2.bin`|
| QUICKSILVER       | `font_quic.bin`, `font_quic_2.bin` | `font_quic_hd.bin`, `font_quic_hd_2.bin`|
| Generic/Fallback       | `font.bin`, `font_2.bin` | `font_hd.bin`, `font_hd_2.bin`|

VTx (AU/Vista) which have not had their msp-osd upgraded, as well as flight controllers which do not respond to the Variant request, like old Ardupilot versions, will fall back to the Generic/Fallback font.

### Suggested Third Party Fonts

 - [KNIFA's Material](https://github.com/Knifa/material-osd/releases)
 - [SNEAKY_FPV's colour fonts for INAV, ARDU and BF](https://sites.google.com/view/sneaky-fpv/home)
 - [VICEWIZE Italic](https://github.com/vicewize/vicewizeosdfontset)
 - [Kw0ngk4n's Neue OSD](https://github.com/Kw0ngk4n/WTF-Neue-OSD)
 - [EVilm1's OSD Font](https://github.com/EVilm1/EVilm1-OSD-Font)

### Generate your own Font from an analog font (advanced)

* Download [mcm2img](https://github.com/bri3d/mcm2img) and set up a working Python environment to run it.

* Locate the font you'd like to install - it will be a `.mcm` file, in the source code repository or configurator for your Flight Controller.

* For Betaflight: https://github.com/betaflight/betaflight-configurator/tree/master/resources/osd/2
* For INAV: https://github.com/iNavFlight/inav-configurator/blob/master/resources/osd/
* For Ardupilot: https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_OSD/fonts

* Run `python3 mcm2img.py mcmfile.mcm font RGBA 255 255 255`

* Place the 4 files this makes (font.bin, font_2.bin, font_hd.bin, font_hd_2.bin) on the root of the SD card in the goggles.

* Reboot

You can customize the font color by changing the 255 255 255 RGB values.

Useful tool for working with fonts: https://github.com/shellixyz/hd_fpv_osd_font_tool

## Modify / Move original DJI OSD elements

You can now modify the elements present in the original DJI OSD. These include for example : transmission speed, latency, channel used, googles battery, sd card icon and default timer.

Elements position, visibility, font and icons can be modified by editing the internal googles files.
This is possible by connecting to the googles using ADB. You can even preview changes using a Python script!

This is not a trivial thing for everyone to do, the full tutorial can be found [here](https://github.com/EVilm1/WIKI-HACK-DJI-OSD#6-advanced-setup-modify-the-dji-hud-elements).

## Compressed Transmission

As of 0.7.0, a new option, `compress_osd`, has been added to the air side process.

If this is set to "true", then the entire character buffer will be sent using LZ4 compression at the rate defined in `osd_update_rate_hz`, instead of sending raw MSP messages over the air.

When enabled, this should fix INAV delta update related issues as well as provide better link stability.

To enable:

Visit https://fpv.wtf/package/fpv-wtf/msp-osd with your Air Unit / Vista plugged in to edit package options.

This option is enabled by default as of 0.10.0, however, if you upgraded from an older version, your configuration will need to be updated using the configurator.

If you continue to have issues with especially INAV character corruption, it is likely your serial link is saturated. Check that the "Custom OSD" option in your DJI goggles menus is set to _disabled_ , and also try out the cache_serial option.

## Configuration options

Configuration options can be set using the WTFOS Configurator.

Visit https://fpv.wtf/package/fpv-wtf/msp-osd with your Goggles or Air Unit plugged in to edit options.

### Current available options (Goggles):

| Option | Description | Type | Default|
| ------ | ----------- | ---- |--------|
|`show_waiting`| enables or disables WAITING FOR OSD message | true/false | true |
|`show_au_data`| enables AU data (temp/voltage) overlay on the right | true/false | false |
|`rec_enabled`| enable OSD recording to .msp files alongside video | true/false | true |
|`rec_pb_enabled`| enable OSD playback if .msp file is stored alongside video | true/false | true |
|`hide_diagnostics`| hide the diagnostic information in the bottom right | true/false | false |
|`fakehd_enable`| enables [FakeHD](#FakeHD); the other FakeHD options don't do anything if this is disabled. FakeHD is force disabled if the Flight Controller supports proper HD / RealHD | true/false| false |
|`fakehd_lock_center`| Lock FakeHD in centered mode all the time; no gaps/spreading out even when you are flying. | true/false | false |
|`fakehd_menu_switch`| FakeHD will use this character as the menu switch to detect when you are in menus/postflight and triggger centering. | integer/number | 4 (Betaflight Throttle) |
|`fakehd_hide_menu_switch`| FakeHD will hide the menu switch set above; and the next 5 characters | true / false | false |
| `fakehd_columns` | FakeHD column alignment config | Single character, one of T M B S | S |
| `fakehd_rows` | FakeHD row alignment config, each character configures the alignment for one row | 16 characters, each one of L C R W T F D | WWWWWWCCWWWWWWWD |



### Current available options (Air Unit/Vista):

| Option | Description | Type | Default|
| ------ | ----------- | ---- |--------|
|`compress_osd`| Enable sending full frames of compressed data. Disable to send raw MSP data [Read more](#Compressed-Transmission) | true/false| true |
| `osd_update_rate_hz` | Configure the update rate in hz for the OSD when using compressed transmission | integer | 10 |
| `disable_betaflight_hd` | Disable HD Mode, which is otherwise set by default in Betaflight 4.4 | true/false | false |
| `fast_serial` | Change serial baud rate to 230400 baud, which can improve OSD performance in some situations - FC UART config must be changed to match. | true/false | false |
| `cache_serial` | Cache unimportant MSP messages for seldom-used features (like PID tuning in the DJI Goggles Settings Menu) to reduce serial pressure | true/false | false |

## FAQ / Suggestions

### How do I create a new font (for INAV, Ardupilot, etc.)?

Use `mcm2img` , specifically Knifa's branch to allow you to draw using a PNG template.

https://github.com/Knifa/mcm2img/tree/templates

### Why is everything so big / can I make the text smaller (betaflight)?

For Betaflight prior to 4.4, look into FakeHD.
For Betaflight after 4.4, you should see "HD" fonts by default. Make sure your VTx (AU/Vista) is powered up and visit the Betaflight Configurator to move OSD items to the edge of the screen.

### How can I get my INAV/ArduPilot/Kiss Ultra OSD closer to the edge of the screen / Why is FakeHD closer to the edges?

 - The goggles need 60 characters to go edge to edge - so the 50 in the hd grid doesn't quite fill it
 - So, depending on the Flight Controller's setup, the RealHD grid is displayed centered in the goggles - gaps on both edges.
 - FakeHD had no compatibility constraints like this so we were able to use the full width of the screens.
 - Consequently, FakeHD can get nearer the edges.
 - Currently no solution to get RealHD closer to the edges.

### What is RealHD

Sometimes we refer to the proper MSP OSD HD grid supported by ArduPilot / Kiss Ultra / INAV / Betaflight (from 4.4) + others as RealHD, to distinguish from FakeHD.

# Compiling (development and debugging)

To build for DJI, install the [Android NDK](https://developer.android.com/ndk/downloads) and add the NDK toolchain to your PATH, then use `ndk-build` to build the targets.

To build for UNIXes, install CSFML and run:

```
make -f Makefile.unix
```

Provided targets and tools are:

* `msp_displayport_mux` - takes MSP DisplayPort messages, bundles each frame (all DisplayPort messages between Draw commands) into a single UDP Datagram, and then blasts it over UDP. Also creates a PTY that passes through all _other_ MSP messages, for `dji_hdvt_uav` to connect to.
* `libdisplayport_osd_shim.so` - Patches the `dji_glasses` process to listen for these MSP DisplayPort messages over UDP, and blits them to a DJI framebuffer screen using the DJI framebuffer HAL `libduml_hal` access library, and a converted Betaflight font stored in `font.bin`.
* `osd_sfml` - The same thing as `osd_dji`, but for a desktop PC using SFML and `bold.png`.

Additional debugging can be enabled using `-DDEBUG` as a CFLAG.

## Custom Build Installation (Goggles)

There's a slightly different process for V1 vs V2 Goggles, they renamed some bits between the two.

### FPV Goggles V1

```
ndk-build
adb shell setprop dji.dji_glasses_service 0
adb push libs/armeabi-v7a/libdisplayport_osd_shim.so /tmp
adb shell LD_PRELOAD=/tmp/libdisplayport_osd_shim.so dji_glasses_original -g
```

### FPV Goggles V2

```
ndk-build
adb shell setprop dji.glasses_wm150_service 0
adb push libs/armeabi-v7a/libdisplayport_osd_shim.so /tmp
adb shell LD_PRELOAD=/tmp/libdisplayport_osd_shim.so dji_gls_wm150_original -g
```

### Air Unit / Air Unit Lite (Vista)

```
ndk-build
adb push msp_displayport_mux /blackbox
adb shell setprop dji.hdvt_uav_service 0
adb shell mv /dev/ttyS1 /dev/ttyS1_moved
adb shell nohup /blackbox/msp_displayport_mux 192.168.41.2 /dev/ttyS1_moved /dev/ttyS1 &
adb shell setprop dji.hdvt_uav_service 1
```

This tells the displayport mux to send data from /dev/ttyS1_moved to 192.168.41.2 (goggles) and to create a fake serial port at /dev/ttyS1 with the displayport messages filtered out.

Optionally, you can add `-f`, like `nohup /blackbox/msp_displayport_mux -f 192.168.41.2 /dev/ttyS1_moved /dev/ttyS1` to put the serial port in a faster 230400 baud mode, and set the MSP serial port in your flight controller to 230400 to try to improve the framerate.

You can also omit `setprop dji.hdvt_uav_service 1` , which will improve your OSD framerate at the expense of disabling all Air Unit / Vista side coordination functionality (AU recording, channel changes, some RC features, etc.).

Enjoy.

## Additional Reading / Learning

https://github.com/fpv-wtf/margerine/wiki

## Shoutouts / Thank You

* http://github.com/fpv-wtf team, for making this all possible and very helpful random bits of advice
