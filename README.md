# MDM Intune Stuff - The serverless school

## Backgrounds

Are some Login Backgrounds made with Blender.

## Working Stuff

### Installing latest winget

I'm using the great Install Script from [https://github.com/asheroto/winget-install](https://github.com/asheroto/winget-install/releases/latest/download/winget-install.ps1).

In every Win32 Script I'm using the following Stuff (so im be sure to have allways _winget_ installed).
I'm also setting ENV Variables to call _winget_ from CLI.

```Powershell
function Install-Winget {
  <#
    .SYNOPSIS
    Installs winget if its not installed, uses Script from https://github.com/asheroto/winget-install/releases/latest/download/winget-install.ps1.
    Stores the Script in C:\ProgramData\WinGetPackages
    .EXAMPLE
    Install-Winget
  #>
  [CmdletBinding()]
  param ()
  Write-Host "Creating Winget Packages Folder" -ForegroundColor Yellow
  New-Item -Path C:\ProgramData\WinGetPackages -Force -ItemType Directory | Out-Null
  Set-Location C:\ProgramData\WinGetPackages

  #Downloading Packagefiles
  $outfile = "C:\ProgramData\WinGetPackages\winget-install.ps1"
  Write-Host "Downloading Microsoft.UI.Xaml latest Version ..."
  Invoke-WebRequest -Uri "https://github.com/asheroto/winget-install/releases/latest/download/winget-install.ps1" -OutFile $outfile -UseBasicParsing
  .\winget-install.ps1
}

function Set-Owner-Everyone {
  <#
    .SYNOPSIS
    Change $Env:ProgramFiles\WindowsApps Folder Owner and Right to everyone
    .EXAMPLE
    Set-Owner-Everyone -Path "C:\Jolly\"
  #>
  [CmdletBinding()]
  param (
    [string]$Path
  )
  $ACL = Get-ACL -Path $Path
  $permission = "Jeder", "FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow'
  $rule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission

  $ACL.SetAccessRule($rule)
  Set-Acl -Path $Path -AclObject $ACL
}

function Add-PATH-Variable {
  <#
    .SYNOPSIS
    Add Path to System ENV:PATH Variable
    .EXAMPLE
    Add-PATH-Variable -Path "C:\Jolly\"
  #>
  [CmdletBinding()]
  param (
    [string]$Path
  )
  # check if Value allready exists
  $envPATH = [Environment]::GetEnvironmentVariable('Path', 'Machine')
  $parts = $envPATH.split(";")
  $found = $false
  foreach ($p in $parts) {
    if ($p -eq $Path) {
      $found = $true
      Break
    }
  }

  if (!$found) {
    Write-Host "Setting PATH > $Path"
    [Environment]::SetEnvironmentVariable("Path", "$Path;" + $envPATH, [EnvironmentVariableTarget]::Machine)
  }
}

# Check for winget or install it =================================
Install-Winget
# Set Owner and Env Path
$winappsPath = "$Env:ProgramFiles\WindowsApps"
Set-Owner-Everyone -Path $winappsPath
Add-PATH-Variable -Path $winappsPath
# winget done ====================================================
```

# Installing Printers

I'm using the greate script from [itelio.com - Raphael Baud](https://clouduncovered.itelio.com/themengebiete/intune/ip-drucker-installieren/).  
By using this script, it's really simple to install Printer Drivers via Intune.
