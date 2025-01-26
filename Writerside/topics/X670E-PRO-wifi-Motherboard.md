# X670E-PRO wifi Motherboard


## BIOs Settings

<procedure title="Memory Context Restore">
<step>
This stops the memory training everytime the machine posts. Only needed if used in conjunction with
Expo profiles.

<path>
 AI Tweaker > Memory Context Restore > Enabled
</path>
</step>
</procedure>



### 7950X3D Core Parking Settings
<procedure title="Core Parking (CPPC)">
<step>
It's been documents that the default setting of 'auto' for CPPC can cause performance issues as the
Motherboard will favour frequency over core parking. Since the 7950X3D has 3D V-Cache on only one of 
the CCDs, it's important to set this setting to 'Driver' to ensure that the chipset driver is used
soley for scheduling the parking of cores.

<path>
 Advanced > AMD CBS > NBIO Common Options > NBIO Configuration > SMU Common Options > CPPC
</path>
</step>
</procedure>