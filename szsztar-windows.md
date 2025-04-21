# Windows configurations

## ADCS

## [RDS](https://msfreaks.wordpress.com/2018/10/06/step-by-step-windows-2019-remote-desktop-services-using-the-gui/)

> [!NOTE]
> If you have to set up RDS on just one device use session based, quick config and it will be done much faster.

### 2 Servers

> [!NOTE]
> In this scenaro I will have two Windows servers where the following services will be configured. ADDS has been preconfigured.

#### SRV1

- DC
- ADCS
- RD Licensing

#### SRV2

- RD Web Access
- RD Gateway
- RD Connection Broker
- RD SessionHost

#### Settings

- Standard Deployment
- Session-based desktop deployment
- RD Connection Broker > SRV2
- RD Web Access >  check in the checkbox
- RD Session Host > SRV2
- Check in _Restart .... if required_ and press **Install**

#### After deployment

> [!NOTE]
> Go to RDS overview and do these steps:

- Click RD Licensing > Choose SRV1
- Click RD Gateway > Choose SRV2
- | --> type SRV2's FQDN
- Press Tasks and choose Edit Deployment Process from the dropdown menu
- | --> RD Gateway > Use These RD Gateway server settings, type in FQDN, choose Password authentication and check in both checkboxes
- | --> RD Licensing > Choose Per User
- | --> RD Web Access > Next
- | --> Certificates > Create certificate, export it into .pfx and import them here.

#### DNS

> [!NOTE]
> If you haven't done already create a DNS zone and record for the server.

#### Collections

- Go to collections click **Tasks** and choose **Create Session Collection** from the Dropdown menu
- Name it as you want and give it a description if you need.
- Choose RD Session Host from the menu (it will show SRV2 if you done it in the way i had).
- Specify a **user group** that can access this collection (if you're using child domains you have to do [this](https://wiki.penguin.hu/virtualization/remoteapp) workaround)
- Check in **Enable user profile disks**
- You're done with the configuration.

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

### Enable run ps scripts

```ps
Get-ExecutionPolicy
Set-ExecutionPolicy
```

### For

```ps
for ($I = 1; $i -le 5; $i++) {
    Write-Output $i
}

$ servers = @("Server1", "Server2")
for($i=0; $i -le $servers.Length; $i++) {
    Write-Output $servers[$i]
}
```

### Foreach

```ps
foreach ($item in $collection) {
    # Perform actions with $item
}
```

### While

```ps
$i = 0

while ($i -lt 5) {

    Write-Host "Iteration: $i"

    $i++

}
```

### If else

```ps
$num = 10

if ($num -gt 5) {

    Write-Host "$num is greater than 5"

} else {

    Write-Host "$num is less than or equal to 5"

}
```

### Elseif

```ps
$num = 10

if ($num -gt 10) {

    Write-Host "$num is greater than 10"

} elseif ($num -eq 10) {

    Write-Host "$num is equal to 10"

} else {

    Write-Host "$num is less than 10"

}
```

### Get Windows current version

```ps
Set-Location HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion
Get-ItemProperty .\Run
```

### Managing files, folders, ACLs

```ps
New-Item -Path "C:\Reports" -ItemType Directory
Copy-Item -Path "C:\OldReports\*.pdf" -Destination "C:\Reports"

# Add a new access rule 
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("DOMAIN\UserGroup","ReadAndExecute","Allow") 
$acl.SetAccessRule($rule)

# Apply the new ACL 
Set-Acl -Path "C:\ConfidentialReports" -AclObject $acl
```

### n User creation script

```ps
Param(
    [int] $count
)

$password = ConvertTo-SecureString -AsPlainText -Force "Passw0rd"
$longpassword = ConvertTo-SecureString -AsPlainText -Force "Passw0rd@Passw0rd"

$groups = @(
    @{username='mkt';ou='ou=MKT,dc=your,dc=domain';password=$password;group="MKT"},
    @{username='hr';ou='ou=HR,dc=your,dc=domain';password=$longpassword;group="HR"}
)

foreach ($group in $groups) {
    for ($i = 1; $i -le $count; $i++) {
        $username = "$($group.username)$($i)"

        if (Get-ADUser -Filter {SamAccountName -eq $username}) {
            Write-Host "Skipping user $($username)"
        } else {
            Write-Host "Creating user $($username)"

            $user = New-ADUser -Name $username `
                -SamAccountName $username `
                -PassThru `
                -Path "ou=$($group.ou)$($basepath)" `
                -PasswordNeverExpires $true `
                -ChangePasswordAtLogon $false `
                -AccountPassword $group.password `
                -Enabled $true

            Add-ADGroupMember -Identity $group.group -Members $user
        }
    }
}
```

## Hyper-V

### Installation

```ps
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

### Virtual switch types

- Internal (VM <--> host)
- External (Bridge)
- Private (VM)

```ps
New-VMSwitch -name "Interace-name" -switchtype internal/external/private
Get-VMSwitch
```

### Nested virtualization

```ps
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```

### MAC address spoofing

Should be enabled on the physical Hyper-V host

```ps
Set-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
