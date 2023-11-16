# Pi Hole Configuration

Changes to the values.yaml file are network specific. The following changes are required:

## White List Domains
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

## A Records For Internal Services

```yaml
  customDnsEntries:
    - address=/plex.domain.cloud/10.1.1.12
    - address=/proxmox.domain.cloud/10.1.1.13
    - address=/nas.domain.cloud/10.1.1.14
    - address=/work.domain.cloud/10.1.1.15

```
