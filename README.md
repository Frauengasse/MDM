# MDM Intune Stuff of our school

# Backgrounds

Are some Login Backgrounds made with Blender.

# Working Stuff

## PS winget

If using `Invoke-WebRequest` be aware if using PS Version < 6 you have to use the Parameter `-UseBasicParsing`.  
Otherwise it will fail with an error message like _invoke-webrequest explore module missing_.

## Installing lates winget

It looks like this (during my testings, it works like a charm)

```Powershell
$wingetVersion = "1.21.0.0"
# winget allready installed? check againt Version ....
$hasPackageManager = Get-AppPackage -name 'Microsoft.DesktopAppInstaller'
if (!$hasPackageManager -or [version]$hasPackageManager.Version -lt [version]$wingetVersion) {
  Write-Host "Winget is not installed, trying to install latest version from Github" -ForegroundColor Yellow
  Try {
    Write-Host "Creating Winget Packages Folder" -ForegroundColor Yellow
    New-Item -Path C:\ProgramData\WinGetPackages -Force -ItemType Directory | Out-Null
    Set-Location C:\ProgramData\WinGetPackages

    #Downloading Packagefiles
    $outfile = "C:\ProgramData\WinGetPackages\microsoft.ui.xaml.2.7.0.zip"
    Write-Host "Downloading Microsoft.UI.Xaml.2.7.0 ..."
    Invoke-WebRequest -Uri "https://www.nuget.org/api/v2/package/Microsoft.UI.Xaml/2.7.0" -OutFile $outfile -UseBasicParsing
    Expand-Archive C:\ProgramData\WinGetPackages\microsoft.ui.xaml.2.7.0.zip -Force

    $outfile = "C:\ProgramData\WinGetPackages\Microsoft.VCLibs.x64.14.00.Desktop.appx"
    Write-Host "Downloading Microsoft.VCLibs.140.00.UWPDesktop ..."
    Invoke-WebRequest -Uri "https://aka.ms/Microsoft.VCLibs.x64.14.00.Desktop.appx" -OutFile $outfile  -UseBasicParsing

    #Winget get latest Version ===============================
    Write-Host "Downloading latest Winget Release Infos ..."
    $data = (Invoke-WebRequest -Uri "https://api.github.com/repos/microsoft/winget-cli/releases/latest"  -UseBasicParsing).Content
    $URL = $data | ConvertFrom-Json | Select-Object -ExpandProperty "assets" | Where-Object "browser_download_url" -Match '.msixbundle' | Select-Object -ExpandProperty "browser_download_url"
    $licenceURL = $data | ConvertFrom-Json | Select-Object -ExpandProperty "assets" | Where-Object "browser_download_url" -Match '.xml' | Select-Object -ExpandProperty "browser_download_url"

    # Get Licence File
    $outfile = 'C:\ProgramData\WinGetPackages\license.xml'
    Write-Host "Downloading Licence File  $licenceURL"
    Invoke-WebRequest -Uri $licenceURL -OutFile $outfile  -UseBasicParsing

    # Get MSI File
    $outfile = 'C:\ProgramData\WinGetPackages\Winget.msixbundle'
    Write-Host "Downloading Winget msixbundle ..."
    Invoke-WebRequest -Uri $URL -OutFile $outfile  -UseBasicParsing

    #Installing dependencies + Winget
    Add-ProvisionedAppxPackage -online -PackagePath:.\Winget.msixbundle -DependencyPackagePath .\Microsoft.VCLibs.x64.14.00.Desktop.appx, .\microsoft.ui.xaml.2.7.0\tools\AppX\x64\Release\Microsoft.UI.Xaml.2.7.Appx -LicensePath 'C:\ProgramData\WinGetPackages\license.xml'
    Write-Host "Starting sleep for Winget to initiate" -Foregroundcolor Yellow
    Start-Sleep 2
  }
  Catch {
    Throw "Failed to install Winget"
    Break
  }
}
```
