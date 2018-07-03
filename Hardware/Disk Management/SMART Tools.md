# Self-Monitoring, Analysis and Reporting Technology System (SMART)
SMART is a monitoring system included in computer hard disk drives (HDDs) and solid-state drives (SSDs). Its primary function is to detect and report various indicators of drive reliability with the intent of anticipating imminent hardware failures.

## Table of Contents

- [Installing Smartmontools](#installing-smartmontools)
- [Using the smartctl command](#using-the-smartctl-command)
  - [Set SMART Support](#set-smart-support)
  - [Display All Data](#display-all-data)
  - [Display Basic Info](#display-basic-info)
  - [Display Health Status](#display-health-status)
  - [Display SMART Attributes](#display-smart-attributes)
  - [Display SMART Self-test Results](#display-smart-self-test-results)
  - [Run SMART Self-test](#run-smart-self-test)
- Advanced smartctl commands
  - [Display SMART Errors](#display-smart-errors)
  - [Display SMART Attributes Behind RAID Controller](#display-smart-attributes-behind-raid-controller)
- Simple Usage
  - [Required Data for Drive Replacement](#required-data-for-drive-replacement)

## Installing Smartmontools
Smartmontools should be installed by default on most 1&1 images. If the smartctl command isn't available for any reason it can be installed with the following commands:

Debian: `apt-get install smartmontools`  
CentOS: `yum install smartmontools`

## Using the smartctl command

The Smartmontools package and the smartctl command line tool provides an interface to a disks’ firmware and specifically to read the SMART data generated and stored by the drive. As long as we know the block device location in the file system, we can get the disk information using the following command

`[root@localhost ~]# smartctl [options] device`

### Set SMART Support
To enable SMART support, the `-s {{on|off}}` option/value can be used to enable or disable.
```
rescue:~# smartctl -s on /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF ENABLE/DISABLE COMMANDS SECTION ===
SMART Enabled.
```
Now that SMART status is enabled, we can look at the SMART data that's been collected and the overall health of the drive.

### Display All Data
Although all the below information and some additional information is available when passing just the `-a` option it's good to understand the information we see in each section and know how to run additional tests if needed.

### Display Basic Info
The drive information can be shown by throwing the `-i` option of smartctl and specifying the drive to check. For example, we see the following information for the `/dev/sda` drive. This tells us the drive manufacturer and serial number as well as other information related to the drive hardware.

```
rescue:~# smartctl -i /dev/sda  
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)  
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org  

=== START OF INFORMATION SECTION ===  
Model Family:     Seagate Barracuda 7200.14 (AF)  
Device Model:     ST1000DM003-1SB102  
Serial Number:    W9A38DA9  
LU WWN Device Id: 5 000c50 09bd58959  
Firmware Version: CC43
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Feb 28 11:18:41 2017 EST
SMART support is: Available - device has SMART capability.
SMART support is: Disabled
```

We can see the drive manufacturer and model under Model Family: as well as the Serial Number: which is why this information is needed to replace the drive. Also, in this example (last line), we can see that the SMART support is available but currently disabled. We'll need to enable SMART support before we can check the stats for this drive or run any tests on it.

### Display Health Status
To show just the overall health of the drive, the `-H` option can be passed to `smartctl`.
```
rescue:~# smartctl -H /dev/sdb
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED
```

### Display SMART Attributes
We can view the data attributes and values using the `-A` option for `smartctl`. The primary data field we look at is the Reallocated Sectors indicated by the highlighted section in the example below. The important attributes are detailed after the break. Many of the values we see for certain attributes are not useful to us in any way. Additionally, some manufacturers report these values in non-standard ways so it's generally safe to ignore them. For example, the Seek_Error_Rate  value on this Seagate drive is 3859636 which looks like a high value but in reality, it's not indicating a problem on this drive.
```
rescue:~# smartctl -A /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 10
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   077   069   006    Pre-fail  Always       -       51771948
  3 Spin_Up_Time            0x0003   098   098   000    Pre-fail  Always       -       0
  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always       -       8
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000f   066   060   045    Pre-fail  Always       -       3859636
  9 Power_On_Hours          0x0032   098   098   000    Old_age   Always       -       2107
 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always       -       7
183 Runtime_Bad_Block       0x0032   099   099   000    Old_age   Always       -       1
184 End-to-End_Error        0x0032   100   100   099    Old_age   Always       -       0
187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always       -       0
188 Command_Timeout         0x0032   100   100   000    Old_age   Always       -       0 0 0
189 High_Fly_Writes         0x003a   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0022   066   066   040    Old_age   Always       -       33/34
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       94
194 Temperature_Celsius     0x0022   034   024   000    Old_age   Always       -       34 (0 24 0)
195 Hardware_ECC_Recovered  0x001a   006   005   000    Old_age   Always       -       51771948
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0
240 Head_Flying_Hours       0x0000   100   253   000    Old_age   Offline      -       2095h+06m+33.640s
241 Total_LBAs_Written      0x0000   100   253   000    Old_age   Offline      -       14329264
242 Total_LBAs_Read         0x0000   100   253   000    Old_age   Offline      -       7790202722
```

| SMART Attribute Name	| Description |
| --------------------- |:-----------:|
| #5 Reallocated_Sector_Ct	| Bad sectors of the drive that were successfully read and replaced with new sectors. The OS does not see these sectors anymore and they cannot affect the OS. Think of these as sectors that the drive has trashed and substituted with other sectors that weren't being used. Even a high number of these reallocated sectors does not indicate it's the cause of problems but it does indicate the drive is beginning to fail. Other attributes will cause errors.|
| #187 Reported Uncorrectable Errors | The count of errors that could not be recovered automatically. |
| #188 Command_Timeout	| The number of operations that timed out and were aborted by the disk firmware. This generally might indicate a problem with the SATA/SCSI cabling. It's not a common error but something to be aware of.|
| #197 Current_Pending_Sector | Count of "unstable" sectors waiting to be reallocated, because of Reported Uncorrectable Errors. If these sectors have been reallocated, this count is decreased and Reallocated_Sector_Ct is increased. Simply, is the transition between #187 and #5 when the data survives or #187 and #198 when the data does not survive.|
| #198 Offline_Uncorrectable	| The total count of uncorrectable errors when reading/writing a sector.

### Display SMART Self-test Results
To read the results of a self-test, we can pass the `-l selftest` option. If a test is currently running, we will also see the remaining percentage of the test.  If a test completes with errors, we know that the drive needs to be replaced immediately. If the drive is behind a Software or Hardware RAID then we shouldn't immediately need to worry about any lost data since the RAID rebuild will copy the data over to the replaced drive if the other RAID disks are in good health.

```
rescue:~# smartctl -l selftest /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Extended offline    Self-test routine in progress 90%      2132         -
# 2  Short offline       Completed without error       00%      2132         -
# 3  Short offline       Completed without error       00%      2017         -
# 4  Extended offline    Completed without error       00%      2012         -
# 5  Short offline       Completed without error       00%      2011         -
```

### Run SMART Self-test
If anything appears out of range in the SMART data, we can force the drive to run a short or long self-test by passing the `-t {{short|long}}` option to `smartctl`. This will trigger an immediate scan of the drive. This can be run while the live system is running but will cause additional disk IO and may affect performance. A short test will typically take a few minutes while a long test may take a few hours on a 1TB drive.
```
rescue:~# smartctl -t short /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Short self-test routine immediately in off-line mode".
Drive command "Execute SMART Short self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 1 minutes for test to complete.
Test will complete after Wed Mar  1 11:57:14 2017
```
Both short and long tests will tell us the estimated time for the test to complete. In these scenarios, it's about a minute for a short test and almost two hours for a long test.
```
rescue:~# smartctl -t long /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".
Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 104 minutes for test to complete.
Test will complete after Wed Mar  1 13:43:14 2017
```
## Advanced smartctl commands

### Display SMART Errors
Since we can just replace the dead or dying drive in a RAID array and rebuild it, we rarely need to look at the actual errors on a drive. If we ever need to, the `-l error` option will show us the errors that have been logged and is a good place to start when trying to recover lost or corrupted data.
```
rescue:~# smartctl -l error /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Error Log Version: 1
No Errors Logged
```

### Display SMART Attributes Behind RAID Controller
#### Step 1: Find the logical device path
If a hardware RAID controller is being used, we first need to find the make of the hardware RAID. On most Linux distributions, `lsblk` will list any block devices on the machine. This will tell us the logical device assigned to the controller. In the example below, /dev/`sda` is this logical device.
```
rescue:~# lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda             8:0    0 931.5G  0 disk
├─sda1          8:1    0     4G  0 part /
├─sda2          8:2    0     2G  0 part [SWAP]
└─sda3          8:3    0 925.5G  0 part
  ├─vg00-usr  253:0    0     5G  0 lvm  /usr
  ├─vg00-var  253:1    0     5G  0 lvm  /var
  └─vg00-home 253:2    0     5G  0 lvm  /home
```
#### Step 2: Find the Hardware Controller
Once we know the logical device, we can run `smartctl -i device` and let smartctl try to discover the RAID Controller. If the output shows an error, the driver is typically provided in the output.
```
rescue:~# smartctl -i /dev/sda
smartctl 6.5 2016-01-24 r4214 [x86_64-linux-4.4.0-128-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

Smartctl open device: /dev/sda failed: DELL or MegaRaid controller, please try adding '-d megaraid,N'
```
If the output does not show an error, then we'll see the make of the RAID Controller. In the example below, `Areca`.
```
smartctl -i /dev/sda
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               Areca
Product:              ARC-1110-VOL#00
Revision:             R001
User Capacity:        1,000,204,402,688 bytes [1.00 TB]
Logical block size:   512 bytes
Rotation Rate:        10000 rpm
Logical Unit id:      0x0004d927fffff800
Serial number:        0000001449594848
Device type:          disk
Transport protocol:   Fibre channel (FCP-2)
Local Time is:        Tue Jul  3 14:28:02 2018 EDT
SMART support is:     Available - device has SMART capability.
SMART support is:     Enabled
Temperature Warning:  Disabled or Not Supported
```
#### Step 3: Use the correct driver to query the SMART attributes.
We can select the RAID driver with smartctl by passing the `-d [driver],[#]` option where `[driver]` is the hardware RAID make and `[#]` is the drive number behind the controller. This number can be 0 or 1 indexed so expect to try 0, 1, 2... to find each drive.
```
rescue:~# smartctl -i -d megaraid,1 /dev/sda
smartctl 6.5 2016-01-24 r4214 [x86_64-linux-4.4.0-128-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

/dev/sda [megaraid_disk_01] [SAT]: Device open changed type from 'megaraid,1' to 'sat+megaraid,1'
=== START OF INFORMATION SECTION ===
Device Model:     TOSHIBA MG04ACA200N
Serial Number:    37GCK231FVMC
LU WWN Device Id: 5 000039 7abc8102e
Add. Product Id:  DELL(tm)
Firmware Version: FJ2D
User Capacity:    2,000,398,934,016 bytes [2.00 TB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA8-ACS (minor revision not indicated)
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Jul  3 08:56:33 2018 EDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```
When this command outputs an error, it should indicate the actual device path to use. In the example below, we need to switch the `/dev/sda` path to `/dev/sg1`.
```
rescue:~# smartctl -i -d areca,1 /dev/sda
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Smartctl open device: /dev/sda [areca_disk#01_enc#01] failed: Input/output error
```

## Quick Usage
### Required Data for Drive Replacement
In order to create a drive replacement request, the output from `smartctl -iA device` **must be** provided to Server Support Team and to our DC-OPS Team. The `Reallocated_Sector_Ct` attribute should be > 0 or, while uncommon, other attributes should indicate a failing drive before submitting the ticket.
```
rescue:~# smartctl -iHA /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)  
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org  

=== START OF INFORMATION SECTION ===  
Model Family:     Seagate Barracuda 7200.14 (AF)  
Device Model:     ST1000DM003-1SB102  
Serial Number:    W9A38DA9  
LU WWN Device Id: 5 000c50 09bd58959  
Firmware Version: CC43
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Feb 28 11:18:41 2017 EST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 10
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   077   069   006    Pre-fail  Always       -       51771948
  3 Spin_Up_Time            0x0003   098   098   000    Pre-fail  Always       -       0
  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always       -       8
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000f   066   060   045    Pre-fail  Always       -       3859636
  9 Power_On_Hours          0x0032   098   098   000    Old_age   Always       -       2107
 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always       -       7
183 Runtime_Bad_Block       0x0032   099   099   000    Old_age   Always       -       1
184 End-to-End_Error        0x0032   100   100   099    Old_age   Always       -       0
187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always       -       0
188 Command_Timeout         0x0032   100   100   000    Old_age   Always       -       0 0 0
189 High_Fly_Writes         0x003a   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0022   066   066   040    Old_age   Always       -       33/34
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       94
194 Temperature_Celsius     0x0022   034   024   000    Old_age   Always       -       34 (0 24 0)
195 Hardware_ECC_Recovered  0x001a   006   005   000    Old_age   Always       -       51771948
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0
240 Head_Flying_Hours       0x0000   100   253   000    Old_age   Offline      -       2095h+06m+33.640s
241 Total_LBAs_Written      0x0000   100   253   000    Old_age   Offline      -       14329264
242 Total_LBAs_Read         0x0000   100   253   000    Old_age   Offline      -       7790202722
```

## Examples
### MegaRaid
```
# LIST BLOCK DEVICES
rescue:~# lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda             8:0    0   1.8T  0 disk
├─sda1          8:1    0     4G  0 part /
├─sda2          8:2    0     2G  0 part [SWAP]
└─sda3          8:3    0   1.8T  0 part
  ├─vg00-usr  252:0    0 184.2G  0 lvm  /usr
  ├─vg00-var  252:1    0     5G  0 lvm  /var
  └─vg00-home 252:2    0     5G  0 lvm  /home
sr0            11:0    1  1024M  0 rom
# TRY TO QUERY BLOCK DEVICE
rescue:~# smartctl -iA /dev/sda
smartctl 6.5 2016-01-24 r4214 [x86_64-linux-4.4.0-128-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

Smartctl open device: /dev/sda failed: DELL or MegaRaid controller, please try adding '-d megaraid,N'
# FIRST DISK IN ARRAY
rescue:~# smartctl -iA /dev/sda -d megaraid,0
smartctl 6.5 2016-01-24 r4214 [x86_64-linux-4.4.0-128-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

/dev/sda [megaraid_disk_00] [SAT]: Device open changed type from 'megaraid,0' to 'sat+megaraid,0'
=== START OF INFORMATION SECTION ===
Device Model:     TOSHIBA MG04ACA200N
Serial Number:    37GAK3RSFVMC
LU WWN Device Id: 5 000039 7abb81b4a
Add. Product Id:  DELL(tm)
Firmware Version: FJ2D
User Capacity:    2,000,398,934,016 bytes [2.00 TB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA8-ACS (minor revision not indicated)
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Jul  3 09:03:32 2018 EDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 16
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000b   100   100   050    Pre-fail  Always       -       0
  2 Throughput_Performance  0x0004   100   100   000    Old_age   Offline      -       0
  3 Spin_Up_Time            0x0027   100   100   001    Pre-fail  Always       -       3172
  4 Start_Stop_Count        0x0032   100   100   000    Old_age   Always       -       18
  5 Reallocated_Sector_Ct   0x0033   100   100   050    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000a   100   100   000    Old_age   Always       -       0
  8 Seek_Time_Performance   0x0004   100   100   000    Old_age   Offline      -       0
  9 Power_On_Hours          0x0032   084   084   000    Old_age   Always       -       6541
 10 Spin_Retry_Count        0x0032   100   100   000    Old_age   Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       17
192 Power-Off_Retract_Count 0x0032   100   100   000    Old_age   Always       -       15
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       18
194 Temperature_Celsius     0x0022   100   100   000    Old_age   Always       -       29 (Min/Max 19/39)
196 Reallocated_Event_Count 0x0032   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0030   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x0032   200   253   000    Old_age   Always       -       0
241 Total_LBAs_Written      0x0032   100   100   000    Old_age   Always       -       4591751241
242 Total_LBAs_Read         0x0032   100   100   000    Old_age   Always       -       5534821321

# SECOND DISK IN ARRAY
rescue:~# smartctl -iA /dev/sda -d megaraid,1
smartctl 6.5 2016-01-24 r4214 [x86_64-linux-4.4.0-128-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

/dev/sda [megaraid_disk_01] [SAT]: Device open changed type from 'megaraid,1' to 'sat+megaraid,1'
=== START OF INFORMATION SECTION ===
Device Model:     TOSHIBA MG04ACA200N
Serial Number:    37GCK231FVMC
LU WWN Device Id: 5 000039 7abc8102e
Add. Product Id:  DELL(tm)
Firmware Version: FJ2D
User Capacity:    2,000,398,934,016 bytes [2.00 TB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA8-ACS (minor revision not indicated)
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Jul  3 09:03:34 2018 EDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 16
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000b   100   100   050    Pre-fail  Always       -       0
  2 Throughput_Performance  0x0004   100   100   000    Old_age   Offline      -       0
  3 Spin_Up_Time            0x0027   100   100   001    Pre-fail  Always       -       3230
  4 Start_Stop_Count        0x0032   100   100   000    Old_age   Always       -       17
  5 Reallocated_Sector_Ct   0x0033   100   100   050    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000a   100   100   000    Old_age   Always       -       0
  8 Seek_Time_Performance   0x0004   100   100   000    Old_age   Offline      -       0
  9 Power_On_Hours          0x0032   084   084   000    Old_age   Always       -       6540
 10 Spin_Retry_Count        0x0032   100   100   000    Old_age   Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       16
192 Power-Off_Retract_Count 0x0032   100   100   000    Old_age   Always       -       14
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       17
194 Temperature_Celsius     0x0022   100   100   000    Old_age   Always       -       29 (Min/Max 20/38)
196 Reallocated_Event_Count 0x0032   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0030   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x0032   200   253   000    Old_age   Always       -       0
241 Total_LBAs_Written      0x0032   100   100   000    Old_age   Always       -       4586258573
242 Total_LBAs_Read         0x0032   100   100   000    Old_age   Always       -       4448981797
```

#### Areca
```
# LIST BLOCK DEVICES
rescue:~# lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda             8:0    0 931.5G  0 disk
├─sda1          8:1    0     4G  0 part /
├─sda2          8:2    0     2G  0 part [SWAP]
└─sda3          8:3    0 925.5G  0 part
  ├─vg00-usr  253:0    0     5G  0 lvm  /usr
  ├─vg00-var  253:1    0     5G  0 lvm  /var
  └─vg00-home 253:2    0     5G  0 lvm  /home
# TRY TO QUERY BLOCK DEVICE
rescue:~# smartctl -iA /dev/sda
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               Areca
Product:              ARC-1110-VOL#00
Revision:             R001
User Capacity:        1,000,204,402,688 bytes [1.00 TB]
Logical block size:   512 bytes
Rotation Rate:        10000 rpm
Logical Unit id:      0x0004d927fffff800
Serial number:        0000001449594848
Device type:          disk
Transport protocol:   Fibre channel (FCP-2)
Local Time is:        Tue Jul  3 15:06:37 2018 EDT
SMART support is:     Available - device has SMART capability.
SMART support is:     Enabled
Temperature Warning:  Disabled or Not Supported

=== START OF READ SMART DATA SECTION ===
Current Drive Temperature:     30 C
Drive Trip Temperature:        25 C

Manufactured in week 30 of year 2002
Specified cycle count over device lifetime:  4278190080
Accumulated start-stop cycles:  256
Elements in grown defect list: 0

# FIND CORRECT DEVICE PATH
rescue:~# smartctl -iA -d areca,1 /dev/sda
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Device /dev/sg1 appears to be an Areca controller.
do_scsi_cmnd_io with write buffer failed code = ffffffff
Smartctl open device: /dev/sda [areca_disk#01_enc#01] failed: Input/output error
# FIRST DISK IN ARRAY
rescue:~# smartctl -iA -d areca,1 /dev/sg1
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda 7200.14 (AF)
Device Model:     ST1000DM003-1SB10C
Serial Number:    Z9A2PM9R
LU WWN Device Id: 5 000c50 0912e87fa
Firmware Version: CC43
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Jul  3 15:07:08 2018 EDT

==> WARNING: A firmware update for this drive may be available,
see the following Seagate web pages:
http://knowledge.seagate.com/articles/en_US/FAQ/207931en
http://knowledge.seagate.com/articles/en_US/FAQ/223651en

SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 10
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   073   063   006    Pre-fail  Always       -       23640640
  3 Spin_Up_Time            0x0003   097   097   000    Pre-fail  Always       -       0
  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always       -       26
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000f   073   060   045    Pre-fail  Always       -       23854513
  9 Power_On_Hours          0x0032   089   089   000    Old_age   Always       -       9927
 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always       -       26
183 Runtime_Bad_Block       0x0032   100   100   000    Old_age   Always       -       0
184 End-to-End_Error        0x0032   100   100   099    Old_age   Always       -       0
187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always       -       0
188 Command_Timeout         0x0032   100   100   000    Old_age   Always       -       0 0 0
189 High_Fly_Writes         0x003a   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0022   071   061   040    Old_age   Always       -       29 (Min/Max 22/35)
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       438
194 Temperature_Celsius     0x0022   029   022   000    Old_age   Always       -       29 (0 22 0 0 0)
195 Hardware_ECC_Recovered  0x001a   021   006   000    Old_age   Always       -       23640640
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0
240 Head_Flying_Hours       0x0000   100   253   000    Old_age   Offline      -       9923h+31m+55.487s
241 Total_LBAs_Written      0x0000   100   253   000    Old_age   Offline      -       752223309
242 Total_LBAs_Read         0x0000   100   253   000    Old_age   Offline      -       3841647

# FIRST DISK IN ARRAY
rescue:~# smartctl -iA -d areca,2 /dev/sg1
smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-514.16.1.el7.x86_64] (local build)
Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda 7200.14 (AF)
Device Model:     ST1000DM003-1SB10C
Serial Number:    Z9A2PHAQ
LU WWN Device Id: 5 000c50 0912ede3e
Firmware Version: CC43
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Jul  3 15:07:13 2018 EDT

==> WARNING: A firmware update for this drive may be available,
see the following Seagate web pages:
http://knowledge.seagate.com/articles/en_US/FAQ/207931en
http://knowledge.seagate.com/articles/en_US/FAQ/223651en

SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 10
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   073   063   006    Pre-fail  Always       -       23685984
  3 Spin_Up_Time            0x0003   097   097   000    Pre-fail  Always       -       0
  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always       -       26
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000f   073   060   045    Pre-fail  Always       -       23319524
  9 Power_On_Hours          0x0032   089   089   000    Old_age   Always       -       9928
 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always       -       26
183 Runtime_Bad_Block       0x0032   100   100   000    Old_age   Always       -       0
184 End-to-End_Error        0x0032   100   100   099    Old_age   Always       -       0
187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always       -       0
188 Command_Timeout         0x0032   100   100   000    Old_age   Always       -       0 0 0
189 High_Fly_Writes         0x003a   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0022   069   059   040    Old_age   Always       -       31 (Min/Max 21/37)
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       437
194 Temperature_Celsius     0x0022   031   021   000    Old_age   Always       -       31 (0 21 0 0 0)
195 Hardware_ECC_Recovered  0x001a   019   006   000    Old_age   Always       -       23685984
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0
240 Head_Flying_Hours       0x0000   100   253   000    Old_age   Offline      -       9924h+25m+16.948s
241 Total_LBAs_Written      0x0000   100   253   000    Old_age   Offline      -       752223485
242 Total_LBAs_Read         0x0000   100   253   000    Old_age   Offline      -       3886495
```
