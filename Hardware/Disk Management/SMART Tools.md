# Self-Monitoring, Analysis and Reporting Technology System (SMART) Tools

## Installing Smartmontools
Smartmontools should be installed by default on most 1&1 images. If the smartctl command isn't available for any reason it can be installed with the following commands:

Debian: `apt-get install smartmontools`  
CentOS: `yum install smartmontools`

## Smartctl CLI and S.M.A.R.T. Attributes

The Smartmontools package and the smartctl command line tool provides an interface to a disks’ firmware and specifically to read the SMART data generated and stored by the drive. As long as we know the block device location in the file system, we can get the disk information using the following command

`[root@localhost ~]# smartctl [options] device`

### Getting all information
Although all the below information and some additional information is available when passing just the `-a` option it's good to understand the information we see in each section and know how to run additional tests if needed.

### Showing drive info
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

### Enable/Disable SMART support
To enable SMART support, the `-s {{on|off}}` option/value can be used to enable or disable.
```
rescue:~# smartctl -s on /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF ENABLE/DISABLE COMMANDS SECTION ===
SMART Enabled.
```

Now that SMART status is enabled, we can look at the SMART data that's been collected and the overall health of the drive.

### Showing overall health of drive
To show just the overall health of the drive, the `-H` option can be passed to `smartctl`.
```
rescue:~# smartctl -H /dev/sdb
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED
```

### Running SMART self-test
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

### Reading SMART self-test results
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

### Reading SMART errors
Since we can just replace the dead or dying drive in a RAID array and rebuild it, we rarely need to look at the actual errors on a drive. If we ever need to, the `-l error` option will show us the errors that have been logged and is a good place to start when trying to recover lost or corupted data.
```
rescue:~# smartctl -l error /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Error Log Version: 1
No Errors Logged
```

### Getting SMART data
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
| Reallocated_Sector_Ct	| Bad sectors of the drive that were successfully read and replaced with new sectors. The OS does not see these sectors anymore and they cannot affect the OS. Think of these as sectors that the drive has trashed and substituted with other sectors that weren't being used. Even a high number of these reallocated sectors does not indicate it's the cause of problems but it does indicate the drive is beginning to fail. Other attributes will cause errors.
| Command_Timeout	| The number of operations that timed out and were aborted by the disk firmware. This generally might indicate a problem with the SATA/SCSI cabling. It's not a common error but something to be aware of.
| Current_Pending_Sector | Similar to Reallocated sectors but the firmware was unable to read the data in the sector and was unable to seamlessly copy the data to a good sector and trash the old one. As these sectors are still "visible" to the OS, they can cause problems and you may see corrupted data on the disk. These errors would need to be manually corrected through OS tools.
| Offline_Uncorrectable	| General error message indicating uncorrectable errors when reading/writing a sector.

### Getting SMART stats for a drive behind Hardware RAID
If a hardware RAID controller is being used, we need to provide `smartctl` the RAID driver and the RAID endpoint so that we access the drive directly and can request the SMART attribute values. We could also use the RAID driver through CLI and then request the SMART attributes for each drive but I find it easiest to use `smartctl` any time we are looking for SMART attributes. The RAID driver is provided with the `-d [3ware,#|areca,#]` option. Since the raw device we provide to `smartctl` this time is the hardware controller, we need to provide the disk number as `,#` after the driver.
```
rescue:~# smartctl -i -d areca,1 /dev/sg1
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.10.104] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda 7200.14 (AF)
Device Model:     ST1000DM003-1SB10C
Serial Number:    Z9A240BX
LU WWN Device Id: 5 000c50 0910073e7
Firmware Version: CC43
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Form Factor:      3.5 inches
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS T13/1699-D revision 4
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Mon Feb 27 17:16:59 2017 EST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```
