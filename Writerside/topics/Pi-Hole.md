# Pi Hole 

## System Overview

## Packages

located on WSL under `~/source/pi-cluster/pihole`

## Build

| Package Name | Artifact Name | Deployment Type |
|--------------|---------------|-----------------|
| pihole       | pihole        | Helm            |

<note> Pipeline location </note>


### Source Code

* [Github dfoulkes/pihole](https://github.com/dfoulkes/pihole) 
### Configuration

Changes to the values.yaml file are network specific. The following changes are required:

```yaml

  whitelist:
    - sdk.split.io  # Village Pizza 
    - p.typekit.net # Village Pizza
    - os.ss2.us     # Village Pizza
    - insideruser.microsoft.com # Microsoft insiders programme
    - login.microsoftonline-p.com # MS Power Automate
    - secure.aadcdn.microsoftonline-p.com # MS Power Automate
    - auth.gfx.ms # MS Power Automate
    - windows.net # MS Power Automate
    - passport.net # MS Power Automate
```

