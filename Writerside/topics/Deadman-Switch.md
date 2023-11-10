# Deadman Switch

## Deposit of Last Resort

This script is to handle the unlikely event that I am incapacitated 
and unable execute my duties to the family. 
It is a last resort, and should be used in the case of my immediate
in capacity. 

### What to do
At the back of the router, you will find a USB stick. On this stick
is a daily backup of deathbox which is a combination of key files
that are record of my key accounts and financial information. This
also includes a copy of my will and testament.

However, accesses this will then be able to contact the financial
institutions and execute the will.

### How to do it

Before removing the USB stick, please ensure that the router is off.
Not doing this will risk data corruption.

Once the USB stick is removed, please insert it into a computer and
copy the contents to the local drive. This at leasts protects against
loss.

## Schedule
<note>Machine needs to be on to run.</note>
This jobs runs using Windows Scheduler. It is set to run every day at
09:00. The job runs a command called `deadman` which is compiled python
<tldr>
Install Locale :<ui-path>C:\Users\danfo\AppData\Local\Programs\Python\Python311\Scripts\deadman.exe</ui-path>
</tldr>
---

## Source Code

The source code is located at:
[https://github.com/dfoulkes/deadmanswitch](https://github.com/dfoulkes/deadmanswitch)




## Configuration
The file to backup are stored in a config file called
`$HOME/.config/deadman/config.yaml`.

and follows the following format:

```yaml
---
directories:
            - section: Finances
              data:
                  -  from: "local machine path"
                     to: "remote network path"

            - section: House Files
              data:
                  -  from: "local machine path"
                     to: "remote network path"
```

Sections and files can be added as required.

## Build Instructions

To build, use setuptools:

```bash
python setup.py install
```

This will install the exe userdata directory. This is usually located `C:\Users\danfo\AppData\Local\Programs\Python\Python311\Scripts\deadman.exe`
