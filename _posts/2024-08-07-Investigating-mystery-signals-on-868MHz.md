---
layout: post
title: Mystery signals on 868MHz
date: 2024-08-07 21:01:00
description: My investigation of mystery signals I captured with RTL-SDR
tags: SDR C
categories: SDR
giscus_comments: true
thumbnail: 
---

[Github repository with supporting files and data](https://github.com/simonjonsson87/mystery-signals)

# Background and intro
This project started when I plugged in my RTL-SDR, tuned it to 868MHz, and saw some mystery signals. This article is about my attempt at finding out what these signals are and writing a decoder for them. But first a little intro about RTL-SDR:

SDR stands for Software Defined Radio, and is exactly what it sounds like: a radio that can be controlled by software. Much like a good old-fashioned radio that you can manually tune to a frequency between 88MHz and 108MHz listen to a radio programme on, an SDR can be tuned by software to frequencies between 24MHz and 1.7GHz. 

You can listen to radio programmes on an RTL-SDR as well, but you need a software on your laptop to do so. You can also listen to radio traffic between airplanes and control towers, receive weather balloon transmissions, weather satelite images, and much more.

In general, different use cases of radio communications use certain frequency bands. Airplane to control tower communication is often on 118-137MHz, weather balloons usually transmit on 400-406MHz, and so on. Frequency 433MHz and 868MHz are used for low power communications such as car keys, RFID, weather stations, home automation, etc.

To decode signals sent on 433MHz and 868MHz on your RTL-SDR, you need a software. One such software is [rtl_433](https://github.com/merbanan/rtl_433) which is an open-sourced project which aims to provide decoders for devices that use these two, and other similar, frequences. 

The first time I used rtl_433 I tuned it to 433MHz and found out that the air in my house was full on information from sensors that I did not own. There were several temperature/humidity sensors, air pressure sensors from my neighbors car, and from remote controls. 

When I tuned rtl_433 to 868MHz, there was really only one kind of signals, and rtl_433 didn't have a decoder for them. Hence, I set out to write one. 

# Finding the right modulation
I first used this command to capture the signals in rtl_433:
```
rtl_433 -f 868M  -S all -A
```
The -A enables rtl_433's built in analysis tool which provided the output in the dropdown below.

<br>
<details>
<summary>Click for full rtl_433 capture output</summary>
<pre>
<code>
Detected FSK package	2024-07-05 19:18:16
Analyzing pulses...
Total count:  100,  width: 78.33 ms		(78329 S)
Pulse width distribution:
 [ 0] count:    1,  width:    0 us [0;0]	(   0 S)
 [ 1] count:    5,  width:    5 us [5;6]	(   5 S)
 [ 2] count:   14,  width:    2 us [2;2]	(   2 S)
 [ 3] count:    6,  width:    3 us [3;3]	(   3 S)
 [ 4] count:    7,  width:    1 us [1;1]	(   1 S)
 [ 5] count:    3,  width:   10 us [9;12]	(  10 S)
 [ 6] count:    7,  width:    4 us [4;4]	(   4 S)
 [ 7] count:    4,  width:    7 us [7;8]	(   7 S)
 [ 8] count:    1,  width:   17 us [17;17]	(  17 S)
 [ 9] count:   41,  width:  500 us [500;504]	( 500 S)
 [10] count:    9,  width: 1000 us [1000;1002]	(1000 S)
 [11] count:    2,  width: 1501 us [1500;1502]	(1501 S)
Gap width distribution:
 [ 0] count:    9,  width:    3 us [3;3]	(   3 S)
 [ 1] count:   10,  width:    2 us [2;2]	(   2 S)
 [ 2] count:   16,  width:    1 us [1;1]	(   1 S)
 [ 3] count:    6,  width:    5 us [5;6]	(   5 S)
 [ 4] count:    4,  width:    4 us [4;4]	(   4 S)
 [ 5] count:    2,  width:    7 us [7;8]	(   7 S)
 [ 6] count:    1,  width:   11 us [11;11]	(  11 S)
 [ 7] count:   38,  width:  499 us [469;501]	( 499 S)
 [ 8] count:    4,  width: 1501 us [1501;1501]	(1501 S)
 [ 9] count:    7,  width: 1000 us [999;1001]	(1000 S)
 [10] count:    1,  width: 4000 us [4000;4000]	(4000 S)
 [11] count:    1,  width: 9504 us [9504;9504]	(9504 S)
Pulse period distribution:
 [ 0] count:    8,  width:    3 us [3;3]	(   3 S)
 [ 1] count:   10,  width:    9 us [8;11]	(   9 S)
 [ 2] count:    6,  width:    4 us [4;4]	(   4 S)
 [ 3] count:   10,  width:    5 us [5;7]	(   5 S)
 [ 4] count:    5,  width:   14 us [12;18]	(  14 S)
 [ 5] count:    2,  width:    2 us [2;2]	(   2 S)
 [ 6] count:    7,  width:    7 us [7;7]	(   7 S)
 [ 7] count:   35,  width: 1000 us [1000;1004]	(1000 S)
 [ 8] count:   11,  width: 2092 us [2001;2502]	(2092 S)
 [ 9] count:    3,  width: 1490 us [1469;1501]	(1490 S)
 [10] count:    1,  width: 4501 us [4501;4501]	(4501 S)
 [11] count:    1,  width: 10004 us [10004;10004]	(10004 S)
Pulse timing distribution:
 [ 0] count:    1,  width:    0 us [0;0]	(   0 S)
 [ 1] count:   11,  width:    5 us [5;6]	(   5 S)
 [ 2] count:   24,  width:    2 us [2;2]	(   2 S)
 [ 3] count:   15,  width:    3 us [3;3]	(   3 S)
 [ 4] count:   23,  width:    1 us [1;1]	(   1 S)
 [ 5] count:    4,  width:   10 us [9;12]	(  10 S)
 [ 6] count:   11,  width:    4 us [4;4]	(   4 S)
 [ 7] count:    6,  width:    7 us [7;8]	(   7 S)
 [ 8] count:    1,  width:   17 us [17;17]	(  17 S)
 [ 9] count:   80,  width:  499 us [469;504]	( 499 S)
 [10] count:   16,  width: 1000 us [999;1002]	(1000 S)
 [11] count:    6,  width: 1501 us [1500;1502]	(1501 S)
 [12] count:    1,  width: 4000 us [4000;4000]	(4000 S)
 [13] count:    1,  width: 9504 us [9504;9504]	(9504 S)
Level estimates [high, low]:  11909,     48
RSSI: -1.4 dB SNR: 23.9 dB Noise: -25.3 dB
Frequency offsets [F1, F2]:   23428,  25369	(+357.5 kHz, +387.1 kHz)
Guessing modulation: Non Return to Zero coding (Pulse Code)
Attempting demodulation... short_width: 1, long_width: 1, reset_limit: 1024, sync_width: 0
Use a flex decoder with -X 'n=name,m=FSK_PCM,s=1,l=1,r=1024'
[pulse_slicer_pcm] Analyzer Device
codes     : {32863}1f8cec1c8fff1f679ffb7fbfffeff8def839b030c3fc8fe706b78fef3c78f01743001206f8070c1f3fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {14999}fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {4526}ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {3526}ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {3525}ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {3525}ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
[pulse_slicer_pcm] Analyzer Device
codes     : {2501}fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
</code>
</pre>
</details>

<br>

rtl_433 guessed that the modulation was 'short_width: 1, long_width: 1, reset_limit: 1024, sync_width: 0' and used it to produce the output shown above.

This output puzzled me, mostly because I could not make out a packet structure. I thought that looking at the raw signal in a graphical tool would help. I found the fantastic software [Universal Radio Hacker](https://github.com/jopohl/urh) (URH) which is something like a Swiss army knife for SDR analysis. I used it to open one of the .cu8 files I had captured in rtl_433 (using the ```-S``` option). The output looked like this:

{% include figure.liquid loading="eager" path="assets/img/URH_whole_pulse.png" class="img-fluid rounded z-depth-1" caption="URH whole pulse" zoomable=true %}

Clearly, URH had guessed the modulation of the signal differently to rtl_433. When I looked at several .cu8 files a package structure started to emerge and I concluded that URH was more right than rtl_433. 

Looking at the signal in URH, I found that the signal had tapered ends. Each transmission began with a gradual increase in signal amplitude over approximately 300 μs. Then comes periods of 500 μs of what appears to be different frequencies. Presumably, one frequency means 1 and another means 0. 

{% include figure.liquid loading="eager" path="assets/img/URH_pulse_start.png" class="img-fluid rounded z-depth-1" caption="URH pulse start" zoomable=true %}

# Building the decoder to capture data

## Capturing the signal with rtl_433
I figured that it would be best to try to get rtl_433 to capture the signal. rtl_433 is great for collecting data since you can get it to output the results in csv format. 

In rtl_433 you can use the option '-X' to dictate what modulation will be used. It has the parameters 'modulation', 'short', 'long', 'reset', and 'gap'. I don't really understand what any of those mean in the context of my sort of modulation so I set both short and long to 500. With reset and gap, I tried different parameters. I found that some parameters make rtl_433 chop the package up into several rows, and for other the packages would get sections with the same data repeated. I experimented with different sets until I found parameters that minimized these issues.

At this stage I also cloned the rtl_433 repository and started getting into how the code works. To create a new decoder, rtl_433 provides a template C file. You define modulation and package structure, and rtl_433 will then use your decoder C file to decode transmissions that match. I started with the template and wrote a decoder (see [GitHub link](https://github.com/simonjonsson87/rtl_433/blob/unknown_device/src/devices/unknown1.c)) that would output the mystery packages as both hex code and binary. This was so that I would get the raw data from the packages so I could analyse them in R. 

Now that I had the decoder I could start rtl_433 and have it capture data automatically using this command:

```
rtl_433 -f 868M -X 'n=name,m=FSK_PCM,s=500,l=500,r=10000,g=7000' -M level -F csv:capture_withX_livingroom_14.csv
```

The '-X' does largely the same as my decoder. However, I found that many packages are caught by the '-X' option but not the decoder. I still don't know why that is. 

The '-M level' generates the Received Signal Strength Indicator (rssi) is is also part of the csv output. This is useful because I could place my laptop in various places to get an idea where the signals are coming from.


# Package analysis
It's finally time to start looking at the packages. It seems that every package starts with 8 bytes of 0xAA followed by 0x2DD4. After that follows a number of bytes. All unique packages are given below (in 'Click for data'). The seven first columns are the 8 first bytes, and the 'rest' column contains whatever is left of the package. The 'count' column indicates how many instances there are of the particular package.  

All R scripts used for the analysis below as well as data can be found in [this repository](https://github.com/simonjonsson87/mystery-signals).

<br>
<details>
<summary>Click for data</summary>

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; text-align:left; ">byte 11 to 12</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; ">byte 13</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; ">byte 14</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; ">byte 15</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; ">byte 16</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; ">byte 17</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; col7">byte 18</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; col8">byte 19 to end</th>
<th style="border-top: double; text-align:left; font-style:italic; font-weight:normal; padding:0.2cm; border-bottom:1px solid black; col9">count</th>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">4A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">FFFFF8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">4C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">8F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">005B8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">95</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">011C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">95</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">98</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00B70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">50</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0458</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">38</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">36</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">95</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0088</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">95</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0208</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">36</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">13</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">28</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">131</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">76</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">62</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">7</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">48</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">40</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0234</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">33</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">71</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0238</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">124</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023B00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0238</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">82</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0238</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">222</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02390</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">88</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">83</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">7</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023F00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">172</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">39</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">202</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">110</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">25</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">285</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022300</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">309</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">399</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02210</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022100</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">289</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">54</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">377</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0226</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02270</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0450</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">631</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02260</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0224</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">247</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02250</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">128</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02240</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">102</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">109</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">290</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">333</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">67</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02280</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">7</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022800</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">171</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">113</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">99</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">70</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">25</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">24</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D700</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">45</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">41</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">153</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0224</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0224</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">64</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">128</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02240</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022400</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">568</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022B00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">330</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">285</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02290</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">170</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">41</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">108</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">33</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022F00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">46</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">89</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022D00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">102</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">37</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">45</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D700</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">109</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">128</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D50</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">67</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">25</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DB0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">78</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DA00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">60</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">20</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">14</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DFC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DFC0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">18</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">21</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C300</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">06</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">07</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">09</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">39</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C50</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">109</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0D</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">72</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0D</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02CB0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">14</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">12</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">12</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">008B00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00458</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0088</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0110</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">011600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0448</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0238</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0238</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">87</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">54</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">023C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">19</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">29</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">81</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0221</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02210</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">64</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">21</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">044</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0440</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">83</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0227</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02270</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">126</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02260</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0224</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">129</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02250</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022500</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">119</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02240</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">33</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">60</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022A00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">81</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">275</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0228</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02280</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022800</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">178</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">37</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">165</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">19</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">129</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022D</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">088</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0110</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0210</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">91</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">14</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">20</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">044</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0440</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">7F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">19</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0168</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">16</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">34</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">37</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">61</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">137</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">111</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">81</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">158</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">218</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">05B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">27</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">24</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">133</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">21</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">16</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0458</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">7E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">016E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">7E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">016</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">7</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">179</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">38</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D800</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">60</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">375</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">87</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DE0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DE00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">058</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">614</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DD0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DD00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">333</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">93</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02DC0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">118</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">149</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">229</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">255</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">235</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">88</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">50</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">87</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">06</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">76</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">058</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">07</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">022</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">54</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0458</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">05B8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">07</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">9B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">075B</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">2A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">4F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">16</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0024</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">107</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">121</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">97</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">17</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">122</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">213</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">35</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">209</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">185</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">220</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">577</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">82</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">670</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">523</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">24</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">96</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">121</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">98</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">161</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">145</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">13</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DF0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">264</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">63</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">7</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">48</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">41</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00EB0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00EB00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">20</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">39</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0090</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">A1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">A1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">19</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">59</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">14</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">48</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">56</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">32</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00CC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">55</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">46</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">29</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">66</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">64</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">161</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">130</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">58</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">42</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">43</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">22</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FF0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">14</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FE0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">40</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">23</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FC0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FC00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">76</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">41</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">27</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">02E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">63</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">9</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">18</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">17</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">47</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">63</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FE0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FE00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">55</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00FD0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">71</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">96</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">234</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">144</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">360</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">493</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">307</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">157</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">46</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">31</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">72</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">468</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">200</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">41</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">50</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">27</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0090</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">48</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">9F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">FF0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">058</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6018</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">31</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00B4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">016</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0168</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">76</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">016</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0160</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">9F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">98</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">758</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">93</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2430</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">580</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0058</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00590</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">272</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">7F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DCCC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">42</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00348</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">036</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">36</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00040</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">71</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0014</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">74</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0016</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">75</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00698</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">9C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">9E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000AC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">7A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00348</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">36</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00088</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">3A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">9E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0034C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">71</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">75</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00690</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0050</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">71</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0014</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">75</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00698</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">BC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0040</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0050</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">375</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003D</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003D00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">154</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">30</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">302</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00230</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00108</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">444</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">375</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">746</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">256</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DD</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">169</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00270</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002700</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">25</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00260</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002F80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0024</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">31</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00250</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00158</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">21</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0014</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">70</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002B00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0050</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">7C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0014</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">53</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0028</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">67</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">185</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">53</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00280</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002800</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0016</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">123</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">23</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">39</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">584</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002D00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">237</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">77</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">002C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">124</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">5</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D300</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">553</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D200</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">530</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">139</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">15</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">ED</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D000</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">182</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">306</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D600</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">111</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D50</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">79</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">FFFF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">3C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0014</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">402</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DB0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">71</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F3</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">67</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">008</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">82</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">13</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00D80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">81</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">10</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DF0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">38</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F7</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">193</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DD0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">152</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">52</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">F9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00DC0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">111</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">DB</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00220</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">28</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">99</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">008</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00580</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D6</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01A0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FF</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">003C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">06</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">E4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01B0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">89</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">E0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00AC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C4</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0050</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">D2</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">F0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">32</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">4A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">63</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">FE</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01300</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">41</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0024</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0130</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C1</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">64</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">4855</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">12</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0020</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0040</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">004C</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">008</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">26</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0080</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">8</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">009</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0090</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0098</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">6</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00980</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">11</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">009800</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">010</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C9</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">01300</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">2</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">18</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; "></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8"></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">530</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">3</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">46</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">1E</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">005A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">6A</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">008</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">CA</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">91</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">93</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col8">0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; col9">1</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">EEA0</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">E5</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; ">C8</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; col7">00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; col8">008</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; border-bottom: double; col9">1</td>
</tr>
</table>

</details>
<br>

## Byte 1 to 10 - Header and 2DDF
First of all, each package appear to start with eight bytes of 0xAA. This is folowed by the two bytes: 0x2DDF. The 2DDF start is not the case for all packages, but for 99.86% of them. I've concluded this is most likely some set bytes that all packages start with. The few exceptions could be corrupted data. 
```
AAAAAAAAAAAAAAAA2DD4...
```

## Byte 11 to 12 - Different devices?
In the next 4 bytes things start to get more complex. I suspect that the next two bytes are ids for different devices. Most packages look something like this:
```
AAAAAAAAAAAAAAAA2DD4075B...
```
If we presume that the next four bytes after 2DDF are an id for a device, the id's 'DCCC', '075B', and 'EEA0' make up 39.03%, 32.74%, and 27.85% of all packages, respectively. 

Looking at the rest if the ids, they usually have lower signal strength and there is also a tendeny for them to come from times when I have captured signals from unusual places. This is consistent with the less common ids being other devices further away.

## Byte 13 - Two package types?
It seems like the first nibble of byte 13 indicates package type. As can be seen in the data, there seems to be two common types of packages. The first has a 0 in the first nibble of byte 13, and there are many variations on the data that comes in this package type. The second starts with a 6, and it's a static package where there is a sequence of 0xC8C8C8 towards the end. When looking at timing of packages these two types are sent together, but more on that later.

## Bytes 14 and 15 - ???
I don't know what these are. As can be seen in the data, they don't vary much. In the tables below any values with under 10 instances have been truncated. The fact that there is a small number of values that these bytes assume makes me think that they are not some sort of continues data, such as a temperature reading. Rather, they might be flags of some kind.

#### Byte 14

| value |     count |
|:--|-----:|
|00 | 11050|
|01 | 33446|
|02 |    14|
|10 |  1297|
|20 |  3014|
|30 |   272|
|80 |    15|

<br>

#### Byte 15

|  value | count |
| :-- | -----: |
|0A | 12534|
|14 |    10|
|16 |   201|
|1A |  1428|
|1E |  9212|
|28 |  3248|
|2A |  3268|
|30 |  3533|
|46 |  1568|
|85 |    13|
|91 |    16|
|C8 | 14033|

<br>


## Bytes 16 and 17 - Measures
When we look at packages where the first nibble of byte 13 is 0, byte 16 is '00' 93.96% of the time, '01' 0.01%  (n=33428). Byte 17, on the other hand, assumes 89 different values, fairly evenly spread. I think that these signals are giving some sort of sensor reading which is 2 bytes long. Usually the sensor reading is less than 255 and only one byte is needed, making byte 16 0x00. But occasionally, the value goes over 255 and byte 16 becomes 01. 


# Data analysis

If we assume for a minute that bytes 16 and 17 are sensor values; we actually have some data to analyse. When plotting the values for these bytes, they did indeed appear to be some sort of sensor reading. One possibility is that they are temperature readings. If so, perhaps the value 260 actually means 26.0 degrees celsius? To investigate this further, I setup a Raspberry Pi to collect data from a few temperature sensors that I knew to be temperature sensors as well as my mystery signals. The shell script used to collect the data can be found [here](https://github.com/simonjonsson87/mystery-signals/blob/6adc45f7f7687b94d73573ef9b305783f4101112/auto-capture.sh).

The known temperature sensors were Acurite Tower and Nexus and they were both transmitting on 433MHz and rtl_433 had decoders for them. Presumably they are both outside, and probably in the sun, because they peaked at 36 degrees. I checked a weather station in the area and they had recorded peak temperatures in the high 20s during the same period. 


{% include figure.liquid loading="eager" path="assets/img/mystery_and_known_temperature_all.png" class="img-fluid rounded z-depth-1" caption="Mystery signals and known temperature - all data" zoomable=true %}

It's clear that the mystery signals are approximately following the know temperature sensors. We see that both in the 24 hour cycle and as the weather gets warmer and colder over several days. The mystery signals do not fluctuate as much, though, which may be because they could be indoor temperature sensors. 

When we look closely at mystery-DCCC, we can see that there is often a small nudge in the curve late in the evening, around 23:00. In the graph below, we see that this nudge is around 01:00 on Saturday night. Could this be someones bedtime?

{% include figure.liquid loading="eager" path="assets/img/mystery_and_known_temperature_4_days.png" class="img-fluid rounded z-depth-1" caption="Mystery signals and known temperature - all 4 days" zoomable=true %}

# Conclusion
Much remains unknown about these signals. They are still mysterious. One could only speculate what the different package types might be, and most of the packages are unknown. However, it appears probably that byte 16 and 17 are in fact temparature readings from an indoor thermometer. 

As for the decoder I wanted to build for rtl_433, it will have to wait. 