# Windows configurations

## ADCS

## RDS

## GPO

### Prohibit access to Control Panel and PC settings

> [!NOTE]
> User Configuration -> Policies -> Administrative Templates -> Control Panel

### Create a logon banner

> [!NOTE]
> Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies – > Security Options

### Mapping network drivers

> [!NOTE]
> User Configuration -> Preferences -> Windows Settings

### Installing printers

> [!NOTE]
> User Configuration -> Preferences -> Windows Settings

### Installing Software

> [!NOTE]
> Computer Configuration -> Policies -> Software Settings

### Add desktop shortcuts

> [!NOTE]
> User Configuration -> Preferences -> Windows Settings

### Modify registry settings

> [!NOTE]
> Computer Configuration -> Preferences -> Windows Settings

### Disable first login animation

> [!NOTE]
> Policies -> Administrative Templates -> System -> Logon

### Disable local Administrator

> [!NOTE]
> Policies -> Windows Settings -> Security settings -> Local Policies -> Security Options

### Enable users log in to DC

> [!NOTE]
> Policies -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment

### Force Wallpaper

> [!NOTE]
> Policies -> Administrative Templates -> Desktop -> Desktop

### Targetings

- Target computers with a specific operating system
- Target a security group
- Target computers by IP address
- RAM, CPU Speed.

```ps
gpupdate /force

gpresult /r
```

## PS

## Hyper-V
