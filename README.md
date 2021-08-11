# xorg-xf86-video-sis
### a X.org/XFree86 video driver for various SIS display adapters.
This driver currently supports these display adapters:
-  Legacy series: SiS 5597/5598, 6326, 6236/AGP/DVD, 530/620
-  300 series: SiS 300/305, 540, 630/730
-  315 series: SiS 315/E/PRO, 55x, 650, 651, M650, 740, 661FX, 661MX, 661GX, M661FX, M661MX, M661GX, 741, 741GX, M741, M741GX
-  330 series: SiS 330/Xabre, 760, 760GX, M760, M760GX, 761, 761GX, M761, M761GX
-  340 series: SiS 34x, XGI Volari Z7, V3XT, V5, V8i

This driver currently does not support these display adapters:  
-  SiS 6201
-  SiS 6202
-  SiS 6205
-  SiS 6215
-  SiS 6225
-  SiS 6306
-  SiS 6306

This driver currently supports these features:  
-  2D acceleration for all chipsets
-  8/15/16/24 bit color depth
-  15 bit color depth support (Legacy series only)
-  Hardware cursor
-  Color hardware cursor (315/330/340 series only)
-  XAA, EXA, XVideo, XRender and other extensions (The XVideo extension is not currently supported on the Volari Z7, XRender acceleration is only supported on the 315/330/340 series chipsets)
-  TV output (6326 only)
-  TV output on adapters that use Chrontel TV encoders or SiS video bridges (300/315/330/340 series only, other encoder chips and video bridges may not be supported)
-  LCD support via LVDS transmitters or SiS video bridges (300/315/330/340 series only, other bridges may not be supported)
-  TurboQueue support (Legacy series only, the 530/620 chipsets are not currently supported)
-  Dual head mode (300/315/330/340 series only)
-  Xinerama (300/315/330/340 series only)
-  Merged framebuffer mode (300/315/330/340 series only)
-  SiSCtrl interface (300/315/330/340 series only)
-  Basic 3D acceleration and OpenGL support (300 series only)  

# Installation 
1. Clone the git repoistory: `git clone https://github.com/ROBLCX/xorg-xf86-video-sis.git`
2. Switch to the latest release: `git checkout for-xf86-video-sis-0.12.0`
3. Compile the driver: 
`
autoreconf 
automake 
./configure --prefix=/usr --disable-static 
make 
make install
`

4. Add these lines to your X configuration file (/etc/X11/xorg.conf):
`Section "Device"
    Identifier      "Configured Video Device"
    Driver          "sis"
EndSection
Section "Monitor"
    Identifier      "Configured Monitor"
EndSection
Section "Screen"
    Identifier      "Default Screen"
    Monitor         "Configured Monitor"
    Device          "Configured Video Device"
EndSection`

# X Configuration Options
The following options are of particular interest for the SiS driver. Each of them must be specified in the Device section of the X configuration file for this card.  
In the list below, the options' arguments are described by type. For "boolean", the keywords "on", "true" and "yes", as well as "off", "false" and "no" respectively have the same meaning.  
### For all chipsets
- These options select whether the software (SW) or hardware (HW) cursor should be used. The default is using the hardware cursor.
- Disables 2D acceleration. By default, 2D acceleration is enabled.
- Determines which acceleration architecture should be used. Possible arguments are "XAA" or "EXA". As of this writing, EXA is still experimental and it is not recommended to be used on production machines. By default, XAA will be used.
- This option enables clockwise ("CW") or counter-clockwise("CCW") rotation of the display. Enabling either CW or CCW rotation disables the RandR extension as well as all 2D acceleration and Xv support. Default: no rotation.
- This option enables reflecting the display horizontally ("X"), vertically ("Y") or in both directions ("XY"). Enablingreflection disables the RandR extension as well as all 2D acceleration and Xv support. Default: no rotation.
- This option enables the shadow framebuffer layer. By default, it is disabled.
- Disables XVideo (Xv) support. By default, XVideo support is enabled .
- Enables or disables gamma correction. Default: gamma correction is enabled.

### Legacy series specific options
- Enables 1 cycle memory access for read and write operations. The default depends on the chipset used.
- SiS chipsets have the ability to extend the engine command queue in video RAM. This concept is called "TurboQueue" and gives some  performance improvement. Due to hardware bugs, the TurboQueue is disabled on the 530/620, otherwise enabled by default.
- (5597/5598) This option, if set, disables the CPU to VGA host bus.  Disabling the host bus will result in a severe performance regression.
- Due to hardware bugs, XVideo may display a corrupt image when decoding YV12 encoded material. This option, if set, disables support for YV12 and hence forces the Xv-aware application to use either YUV2 or XShm for video output.
- (6326 only) Selects the TV output standard. May be PAL or NTSC. By default, this is selected by a jumper on the card.
- (6326 only) The SiS 6326 can only directly address 4096K bytes of video RAM. However, there are some cards out there featuring 8192K (8MB) of video RAM. This RAM is not addressable by the engines. Therefore, by default, the driver will only use 4096K. This behavior can be overridden by specifying the amount of video RAM using the VideoRAM keyword. If more than 4096K is specified, the driver will disable 2D acceleration, Xv and the HW cursor. On all other chipsets, this keyword is ignored. The size argument is expected in KB, but without "KB".

### 300/315/330/340 series specific options
- This option enables/disables the driver's interface for the SiSCtrl utility.
- Enables or disables CRT1 (= the external VGA monitor). By default, the driver will use CRT1 if a monitor is detected during server start. Some older monitors can't be detected, so they may require setting this option to true. To disable CRT1 output, set this option to false.
- (For SiS 650, M650, 651, 661, 741, 760 with either SiS 301LV, 302LV or SiS 301C video bridge only) The argument may be "VGA", "LCD" or "OFF".  Specifying LCD will force the driver to use the VGA controller's CRT1 channel for driving the LCD while CRT2 is free for TV usage. "OFF" is the same as setting the option ForceCRT1 to "false". Default is VGA
- Selects the CRT2 output device type. Valid parameters are "LCD", "TV", "SVIDEO", "COMPOSITE", "SVIDEO+COMPOSITE", "SCART", "VGA", "YPBPR480I", "YPBPR480P", "YPBPR720P", "YPBPR1080I" or "NONE". NONE disables CRT2.  SVIDEO, COMPOSITE, SVIDEO+COMPOSITE, SCART and all the YPBPR alternatives are only for systems with a SiS video bridge and select the desired plug or TV standard type. For Chrontel systems, TV should be used instead. VGA means secondary VGA and is only available on some SiS video bridges (301, 301B, 301C).
- (For SiS video bridges only) This option enables or disables gamma correction for CRT2. Default: gamma correction for CRT2 is enabled.
- Although this option is accepted for all chipsets, it currently only makes sense on the 300 series; DRI is only supported on these chipsets. This option enables/disables DRI.
- Selects the TV output standard. May be PAL or NTSC, on some machines (depending on the hardware) also PALM and PALN. Default: BIOS setting.
- These options allow relocating the image on your TV. Both options take an integer within the range of -16 to 16. Default: 0. Not supported for Chrontel 7019.
- (For Chrontel TV encoders only) Selects whether TV output should be overscan or underscan.
- (For Chrontel 7005 TV encoders in PAL mode only) Selects whether TV output should be super-overscan (slightly larger than the viewable area) or not.
- (For SiS video bridges only) These options allow zooming the image on your TV. SISTVXScale takes an integer within the rangeb of -16 to 16.  SISTVYScale accepts -4 to 3. Default: 0. Not all modes can be scaled.

### 300 series specific options
- This option might only be needed if you are running X on a Linux 2.4 series kernel. This option is not needed and should be omitted on Linux 2.6 and *BSD.

### 315/330/340 series specific options
- Enables or disables RENDER acceleration. This feature, for instance, accelerates output of anti-aliased text. By default, RENDER acceleration is enabled. RENDER acceleration is currently only supported for XAA, not EXA.
- (For 315, 650, 740, 330, 340 and XGI chips only) This option selects whether the XVideo (Xv) overlay should be displayed on CRT1 or CRT2. Setting this option means CRT2. The other CRT will only display the (by default: blue) color key or a black/red pattern.
      

