# Issues

## RTAI related

For the latency test, you should run:
```
cd /usr/realtime/testsuite/kern/latency/
./run
# ctrl+c to stop
```

Once you've finished checking you don't have any overruns and that your latencies are *acceptable*, check that the timers are not going crazy (the computer might hang later if so):
```
dmesg | grep clock
```
Should say that the current clock is **TSC**, and **not** switched back to **refined-jiffies**:
```
[    0.248375] Switched to clocksource refined-jiffies
[    0.257802] Switched to clocksource acpi_pm
[    0.622418] rtc_cmos 00:05: setting system clock to 2014-10-16 09:15:25 UTC (1413450925)
[    0.649505] PTP clock support registered
[    0.933432] e1000e 0000:00:19.0 eth0: registered PHC clock
[    1.543413] tsc: Refined TSC clocksource calibration: 2793.657 MHz
[    2.544694] Switched to clocksource tsc
```

You can also do the same with the m3 system:
```
m3rt_insmods
m3rt_rmmods
dmesg | grep clock
```
And check if the timers stall.

Troubleshooting:

* After the latency test, the TSC timer stalls or get unstable (see some explanation [here](https://software.intel.com/en-us/blogs/2013/06/20/eliminate-the-dreaded-clocksource-is-unstable-message-switch-to-tsc-for-a-stable)), it switches to refined-jiffies. It means that the computer is not properly configured for RTAI. You should disable every unnecessary modules (look for blacklist.conf) that might cause timers to stall. 

### 'No Intel chipset found' :

Locate the id of your chipset:
```bash
lspci -vvv -nn | grep ISA
```

My output is :
```bash
00:1f.0 ISA bridge [0601]: Intel Corporation B75 Express Chipset LPC Controller [8086:1e49] (rev 04)
```

The name of my device B75 and the id is ***1e49*** (end of the line).
So now open the following file :

```bash
gedit ~/rtai/base/arch/x86/calibration/smi-module.c
```

And add the name and id of your device as the other ones in the file. For example :

```c
#ifndef PCI_DEVICE_ID_INTEL_B75
#define PCI_DEVICE_ID_INTEL_B75  0x1e49
#endif
```

AND

```c
{ PCI_DEVICE(PCI_VENDOR_ID_INTEL, PCI_DEVICE_ID_INTEL_B75) },
```bash


Save, re-compile and re-install:
```bash
sudo make install
```


You should now see "Enabling SMI workaround" when you insert the rtai_smi module.

