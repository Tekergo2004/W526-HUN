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
