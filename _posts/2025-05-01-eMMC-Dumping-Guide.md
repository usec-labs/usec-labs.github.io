---
# layout: post_docstyle
# title: "Dumping eMMC Like a PRO"
# date: 2025-05-01
# image: /assets/images/emmc-hero.jpg
# image_alt: "eMMC chip with probe connections"

title: "Dumping eMMC Like a PRO"
date: 2025-05-08
categories: [Tutorial, eMMC]
tags: [emmc, hydrabus, easyjtag, hacking, beaglebone, easyjtag]     # TAG names should always be lowercase
description: Step-by-step guide to dumping eMMC memory using EasyJTAG, BeagleBone Black, and HydraBus — including access to BOOT, USER, and RPMB partitions.
toc: true
comments: true

---

## Introduction

Many device manufacturers choose eMMC as external memory because it is simple and inexpensive. This memory is primarily used to store code and data. Like any external memory, eMMC contect can be extracted by attackers for the following reverse engineering and analysis.

In this blog post, we discuss three techniques for dumping eMMC data. The first two methods are fairly common; the third - based on HydraBus - is aimed at readers who want command-level control of eMMC communication.

We also assume that the eMMC chip has been removed from the board. While in-circuit dumping is possible, it requires supplying the eMMC with the correct voltage and attaching additional wires to the CMD, CLK, and DATA lines. This task is generally more complicated (than the eMMC unsoldering), because the power you inject into the eMMC also reaches other components on the board, and you must keep the original eMMC host (the main application processor) powered down. Access to the eMMC pins is not always possible either, especially when their traces are buried in the inner layers of the PCB. Overall, in-circuit dumping is more complicated and prone to errors.

> Although dumping the chip in-circuit is possible, we assume that the eMMC chip has been removed from the board.
{: .prompt-tip }

> To keep the blog post concise, we’ve placed the eMMC description at a different post. Nonetheless, we want to emphasize that our goal is to read all three hardware partitions - BOOT, USER, and RPMB.
{: .prompt-info }

### Tools Used
- AOYUE hot air station  
- Binocular microscope  
- Reballing toolset and low-temperature soldering paste  
- Easy JTAG Plus  
- ICFriend universal eMMC socket  
- Consumables such as desoldering wire, pliers, acetone, etc.  
- Breakout PCB for eMMC  
- Dell Precision 3470 laptop  
- BeagleBone Black single-board computer

### Useful External links
- [JEDEC eMMC 5.1 Specification](https://www.jedec.org/document_search?search_api_views_fulltext=jesd84-b51){:target="_blank" rel="noopener"}
- [HydraBus Binary MMC Mode Guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-MMC-mode-guide){:target="_blank" rel="noopener"}
- [Replay Protected Memory Block](https://en.wikipedia.org/wiki/Replay_Protected_Memory_Block){:target="_blank" rel="noopener"}
- [eMMC WFBGA153 to microSD card adapter PCB](https://github.com/voltlog/emmc-wfbga153-microsd){:target="_blank" rel="noopener"}
- [Reballing eMMC](https://www.youtube.com/shorts/s1ZNlKvwHKE){:target="_blank" rel="noopener"}

## Dumping eMMC Like a Repair Expert

If you need to dump eMMC chips regularly, we recommend investing in a commercial tool designed for this purpose. These tools typically include a socket where the eMMC chip can be inserted. In most cases, you can dump all the hardware partitions - BOOT, USER, and RPMB - with a single click.

### Examples of the commercially available tools

| Tool        | URL                                                 |
|-------------|-----------------------------------------------------|
| EasyJTAG    | [z3x-team.com](https://z3x-team.com){:target="_blank" rel="noopener"} |
| UFI Box     | [ufi-box.com](https://ufi-box.com){:target="_blank" rel="noopener"}   |
| Medusa Pro  | [medusabox.com](https://medusabox.com){:target="_blank" rel="noopener"} |
| RIFF Box    | [riffbox.org](https://riffbox.org){:target="_blank" rel="noopener"}   |
| BeeProg2    | [elnec.com](https://www.elnec.com/en/){:target="_blank" rel="noopener"} |

### What You Need to Do

1. Unsolder the eMMC chip. Hot air and patience will help you.  
   ![Unsoldering photo](/assets/img/blog/2025-05-01-emmc-dumping-guide/Unsoldering.jpg)
   _Figure 1. Unsoldering the eMMC chip from an Amazon Echo Spot_

2. Clean the eMMC pads.  
   ![Cleaning pads](/assets/img/blog/2025-05-01-emmc-dumping-guide/eMMC_cleaning.png)
   _Figure 2. Cleaning the eMMC pads_

3. Reball the eMMC or at least apply a small amount of solder to each pad to ensure good contact with the socket pins. If you are not familiar with reballing techniques, please, check YouTube tutorials ([Reballing eMMC](https://www.youtube.com/shorts/s1ZNlKvwHKE){:target="_blank" rel="noopener"}). 
   ![Reballing](/assets/img/blog/2025-05-01-emmc-dumping-guide/eMMC_reballed.png)
   _Figure 3. Reballing result on an eMMC chip_

4. Insert the eMMC into the socket (and connect the socket to the tool).  
   ![Reballing](/assets/img/blog/2025-05-01-emmc-dumping-guide/Socket.jpg)
   _Figure 4. Universal eMMC socket compatible with multiple readers_

5. Use the tool’s software to dump the contents.  
   ![Reballing](/assets/img/blog/2025-05-01-emmc-dumping-guide/Easy_jtag.jpg)
   _Figure 5. Dumping eMMC partitions using EasyJTAG software_

Let’s look at the beginning of the dumped BOOT and RPMB partitions. Although the first 64 bytes don’t contain much information, we can see that the BOOT partition uses big-endian byte order, whereas the RPMB partition uses little-endian.
![eMMC_dumps_easyjtag](/assets/img/blog/2025-05-01-emmc-dumping-guide/Linux_console_easyjtag.png)
_Figure 6. BOOT and RPMB partition data (dumped using EasyJTAG)_

---

## Dumping eMMC Like a Software Hacker

If you only need to dump eMMC content occasionally, professional tools might feel too expensive. Fortunately, Linux supports eMMC natively, however, we need to find a way to connect the eMMC to a computer.

> The SD and eMMC signal sets are physically compatible - and their command sets are similar too. 
{: .prompt-info }

You can read the eMMC using SD slots. So you just need to find an interface between the eMMC and SD connector. One possibility is to use a [breakout PCB](https://github.com/voltlog/emmc-wfbga153-microsd){:target="_blank" rel="noopener"}. 
![Breakout](/assets/img/blog/2025-05-01-emmc-dumping-guide/PCB.jpg)
_Figure 7. Breakout board for connecting eMMC to an SD card interface_

You can solder your eMMC to a breakout PCB and plug it into the SD slot of a computer to read the USER partition. Let's do that. Reball your eMMC and then solder the eMMC to the breakout board as shown below.
1. Unsolder the eMMC chip.
2. Clean the eMMC pads.
3. Reball the eMMC (this time this step is mandatory).
4. Solder the eMMC onto the breakout board. 
   ![Reballing](/assets/img/blog/2025-05-01-emmc-dumping-guide/Breakout_soldering_2.jpg)
   _Figure 8. Soldering the eMMC chip onto the breakout board using hot air_
5. Plug the socket into the SD compartment of your laptop/computer.  
   ![SDReading](/assets/img/blog/2025-05-01-emmc-dumping-guide/SD_reading.jpg)
   _Figure 9. Accessing eMMC partitions with a USB SD card reader_
6. The USER partition should appear in the list of the recognized drives.  
   ![Console](/assets/img/blog/2025-05-01-emmc-dumping-guide/Linux_console.png)
   _SFigure 10. oftware partitions within the USER hardware area_

We’re actually looking at 19 distinct software partitions that reside inside the USER hardware partition. But what about BOOT and RPMB? Reading the USER partition on an eMMC device uses the same commands as reading an SD card, whereas accessing the BOOT and RPMB partitions requires a different command set. Unfortunately, inexpensive SD controllers - including those in most laptops and desktops - don’t support those commands, so they can’t reach those partitions.

To read them, you need a host that understands the full eMMC command set. A handy choice is the BeagleBone Black (BBB): its SD interface is fully eMMC-compatible, allowing it to access all three hardware partitions - BOOT, USER, and RPMB. Just attach the eMMC module, reboot the board, and the partitions will show up under `/dev/`.

> The BeagleBone Black's SD controller can read BOOT, USER, and RPBM hardware eMMC partitions. 
{: .prompt-tip }

1. Unsolder the eMMC chip.
2. Clean the eMMC pads.
3. Reball the eMMC (this time this step is mandatory).
4. Solder the eMMC onto the breakout board.
5. Plug the socket into the SD compartment of the BBB.  
   ![SDReading_BBB](/assets/img/blog/2025-05-01-emmc-dumping-guide/BBB1.jpg)
   _Figure 11. Connecting the breakout board to the BeagleBone Black_
6. Read the partitions using `dd` or `mmc` commands.  
   ![Console](/assets/img/blog/2025-05-01-emmc-dumping-guide/BBB_console.png)
   _Figure 12. Reading eMMC partitions directly with BeagleBone Black_

The command `ls mmc*` shows the eMMC as the base device `mmcblk0`. It contains 19 software partitions - `mmcblk0p1` through `mmcblk0p19` - the same set that appeared on the Dell laptop as `/dev/sda1` to `/dev/sda19`.

On the BeagleBone Black, three additional block devices also appear: `mmcblk0boot0`, `mmcblk0boot1`, and `mmcblk0rpmb`, representing the two BOOT hardware partitions and the RPMB partition that the Dell laptop could not detect.

You can dump the BOOT partitions with `dd`, and you can dump the RPMB partition with the `mmc` utility.

---

## Dumping eMMC Like a Hardware Hacker

This section explains how to access an eMMC by issuing raw commands. To send these commands from a standard PC, you need a compatible interface - our recommended tool is the HydraBus. Alternatively, you could develop an FPGA-based interposer that sits between the PC and the eMMC, but that approach is far more time-consuming.

### Recommended Tool: HydraBus

HydraBus ([hydrabus.com](https://hydrabus.com){:target="_blank" rel="noopener"}) is a hardware hacker’s Swiss Army knife (yes, it has Swiss roots). It supports UART, SPI, I2C, SD/eMMC, acts as a logic bridge, ADC, and can even supply power (3V3 and 5V).  
![HydraBus](/assets/img/blog/2025-05-01-emmc-dumping-guide/HydraBus.jpg)
_Figure 13. HydraBus — the Swiss Army knife for hardware hackers_

To work with an eMMC, it lets you send raw commands, which is perfect if you're experimenting with command sequences, vendor extensions, or reverse engineering RPMB behavior. The current HydraBus firmware can access only the USER and the RPMB partitions, accessing the BOOT partition is a way trickier due to the SD controller state machine and currently the BOOT partition can not be read with HydraBus.

### Steps for the PRO Hardware Hacker
1. Unsolder the eMMC.  
2. Clean the pads.  
3. Reball it (with your eyes closed - you’re a hardware pro now).  
4. Solder it to an adapter board.  
5. Connect the eMMC pins to HydraBus (follow the description here: [HydraBus Binary eMMC mode guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-MMC-mode-guide){:target="_blank" rel="noopener"}).
   ![HydraBus_eMMC_connection](/assets/img/blog/2025-05-01-emmc-dumping-guide/HydraBus_eMMC_connection.jpg)
   _Figure 14. Wiring the eMMC to the HydraBus interface_
6. Run a Python script to issue low-level commands.
   ![HydraBus_eMMC_connection](/assets/img/blog/2025-05-01-emmc-dumping-guide/HydraBus_reading.png)
   _Figure 15. Using HydraBus and Python to read the RPMB partition_

The Python function shown in the last image begins by resetting the eMMC with command CMD0. Reading an RPMB sector then requires sending CMD1, CMD2, CMD3, CMD6, CMD7, CMD8, CMD9, CMD13, CMD18, and CMD23 in the correct sequence while following the RPMB protocol. At one point the eMMC clock must be raised; otherwise, the device will not transition to the next state. When the read completes, the python code parses the raw RPMB packet to isolate the data payload, and displays it with hexdump. A detailed walkthrough of the RPMB protocol is beyond the scope of this post.

### Why Not Use BBB for this sort of research?
- HydraBus gives full control so that you are sure about the current eMMC state.
- HydraBus can reset the eMMC without rebooting the whole SBC.
- Better for glitching, fault injection, and fine-grained research since HydraBus can control the eMMC frequency, timing, accessing the current eMMC state, etc.
