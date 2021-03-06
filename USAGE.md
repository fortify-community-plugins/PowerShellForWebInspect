# Power Shell For WebInspect Module

## Usage

#### Table of Contents
*   [Configuration](#configuration)
*   [Scans](#scans)
    * [Retrieving Scans](#retrieving-scans)
    * [Retrieving Scan Status](#retrieving-scan-status)
    * [Retrieving Scan Log](#retrieving-scan-log)
    * [Starting a Scan](#starting-a-scan)
    * [Exporting a scan as FPR](#exporting-a-scan-as-fpr)
    * [Exporting scan details as XML](#exporting-scan-details-as-xml)
*   [Policies and Checks](#policies-and-checks)
    * [Retrieving Policies](#retrieving-policies)
    * [Retrieving Policy Details](#retrieving-policy-details)
    * [Retrieving Checks](#retrieving-checks)    
*   [Troubleshooting](#troubleshooting)    

----------

## Configuration

To access the [WebInspect](https://www.microfocus.com/en-us/products/webinspect-dynamic-analysis-dast/) API you will need to 
have installed WebInspect as per the [documentation](https://www.microfocus.com/documentation/fortify-webinspect/) and 
started the Windows **WebInspect API Service** either automatically or using the **Micro Focus Fortify Monitor Tool**. 
Using the Monitor tool you can configure the Authentication method to use:

![Fortify Monitor](Media\fortify-monitor.png)

Assuming you have configured **Basic** Authentication, then you can configure this module using the following:

```PowerShell
$Credential = Get-Credential
Set-WIConfig -ApiUri http://localhost:8083/webinspect -AuthMethod Basic -Credential $Credential
```

You will be requested for your authentication details after the first command which will then be stored on the filesystem
for all future requests. The configuration is encrypted and stored on disk for use in subsequent commands.

To retrieve the current configuration execute the following:

```PowerShell
Get-WIConfig
```

There following configuration settings are available/visible:

- `Proxy` - A proxy server configuration to use
- `ApiUri` - The API endpoint of the WebInspect API you are using
- `AuthMethod` - The authentication method being used
- `Credential` - A PowerShell Credential object
- `ForceVerbose` - Force Verbose output for all commands and subcommands 

Each of these options can be set via `Set-WIConfig`, for example `Set-WIConfig -ForceVerbose` to force verbose output 
in commands and sub-commands.

----------

## Scans

### Retrieving Scans

You can retrieve scans using `Get-WIScans`. For example, to get all of the scans with status 'Complete' and name 'test' you could
use the following:

```Powershell
Get-WIScans -Status "complete" -Name 'test'
```

### Retrieving Scan Status

You can retrieve the status of an individual scan using `Get-WIScanStatus`. For example, to get the status of the scan 
with id "1cec4067-cb67-42ed-8f00-e5220b5afc04" you could use the following:

```PowerShell
Get-WIScanStatus -ScanId "1cec4067-cb67-42ed-8f00-e5220b5afc04"
```
         
### Retrieving Scan Log

You can retrieve the log of an individual scan using `Get-WIScanLog`. For example, to get the log of the scan with 
id "1cec4067-cb67-42ed-8f00-e5220b5afc04" and display the result int a grid you could use the following:

```PowerShell
Get-WIScanLog -ScanId "1cec4067-cb67-42ed-8f00-e5220b5afc04" | Out-GridView
```
                                                                                                                                                                                            
### Starting a New Scan

You can start a new scan using `New-WIScan`; however as there are many options for different types of scans you need
to create new "DescriptorObject's" first with details of the scan. For the most basic scan with "Default" settings you 
could create a `StartScanDescriptorObject` and scan as follows:

```PowerShell
$startScanDescriptor = New-WIStartScanDescriptorObject -SettingsName "Default"
New-WIScan -StartScanDescriptor $startScanDescriptor
```

For a more complex scan you need to create a `ScanSettingsOverrideObject`. For example, to create a scan of the URL
"http://localhost:8443/mywebapp" you could use the following:

```PowerShell
$scanSettingsOverrides = New-WIScanSettingsOverrideObject -ScanName "My Scan Name" `
    -StartUrls "http://localhost:8443/mywebapp"
$startScanDescriptor = New-WIStartScanDescriptorObject -SettingsName "Default"
New-WIScan -StartScanDescriptor $startScanDescriptor
```

Finally, for a scan that uses a login macro called "tcLogin" with macro parameters and a specific Scan policy you could
use the following:

```PowerShell
$macroParams = @(
    New-WIMacroParameterObject -Name "startURL" -Value "http://localhost:8443/mywebapp"
    New-WIMacroParameterObject -Name "username" -Value "user"
    New-WIMacroParameterObject -Name "password" -Value "password"
)
$loginMacroParam = New-WIMacroParametersObject -MacroName swaLogin -MacroParameters $macroParamstc
$scanSettingsOverrides = New-WIScanSettingsOverrideObject -ScanName "My Scan Name" `
    -StartUrls "https://localhost:8443/mywebapp" `
    -LoginMacro "tcLogin" -MacroParameters $loginMacroParam `
    -PolicyId 1008 # 1008 is "Critical and High" policy
$startScanDescriptor = New-WIStartScanDescriptorObject -SettingsName "Default"
New-WIScan -StartScanDescriptor $startScanDescriptor
```

### Stopping a Scan

You can stop a running scan using `Stop-WIScan` as the following:

```Powershell
Stop-WIScan -ScanId "1cec4067-cb67-42ed-8f00-e5220b5afc04"
```

### Starting Scan 

You can start a stopped scan using `Start-WIScan` as the following:

```Powershell
Start-WIScan -ScanId "1cec4067-cb67-42ed-8f00-e5220b5afc04"
```

### Exporting a scan as FPR

You can export a "Completed" scan's results as a Fortify Project Results (FPR) file using `Export-WIScan` as in the 
following:

```PowerShell
Export-WIScanReport -ScanId ff860346-2978-4f14-bae3-004ff0a535c2 -ReportFormat fpr -OutFile test.fpr
```

### Exporting scan details as XML

You can export a "Completed" scan's detailed results as an XML file using `Export-WIScanDetails` as in the 
following:

```PowerShell
Export-WIScanDetails -ScanId ff860346-2978-4f14-bae3-004ff0a535c2 -OutFile test.xml -DetailType Vulnerabilities -Format xml
```

----------

## Policies and Checks

### Retrieving Policies

You can retrieve all of the Scan policies using `Get-WIPolicies` as in the following:

```PowerShell
Get-WIPolicies | Out-GridView
```

### Retrieving Policy Details

You can retrieve the detail of a specific Scan policy using `Get-WIPolicyDetails` as in the following which retrieves
all of the "checks" associated with a policy:

```PowerShell
Get-WIPolicyDetails -PolicyId 7235cf62-ee1a-4045-88f8-898c1735856f | `
            Select-Object -ExpandProperty checks | Out-GridView
```

### Retrieving Checks

You can retrieve details of an individual Check using `Get-WIChecks` as in the following:

```PowerShell
Get-WIChecks | Out-GridView
```

----------  

## Troubleshooting

### Untrusted Repository

If this is the first time you have installed a module from [PSGallery](https://www.powershellgallery.com/), you might receive a message similar to the
following:

```
Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its
InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
```

Select `Y` to install the module this time or you can use `Set-PSRepository` cmdlet.

### Copying Login and Web macros to Service User directory

If you are running the WebInspect API as a service then by default it will look for macros in the directory:
`C:\Windows\System32\config\systemprofile\AppData\Local\HP\HP WebInspect\Tools\Settings`. If you create the macros
using your user account make sure you copy your macros here using a command similar to the following:

```Powershell
copy /Y C:\Users\YOUR_USERNAME\Documents\HP\Tools\swaLogin.webmacro "C:\Windows\System32\config\systemprofile\AppData\Local\HP\HP WebInspect\Tools\Settings"
```

### Removing/creating the configuration file

The configuration file is stored in `%HOME_DRIVE%%HOME_PATH%\AppData\Local\Temp`, e.g. `C:\Users\demo\AppData\Local\Temp`
as `%USERNAME%-hostname-PS4WI.xml`. You can delete this file and re-create it using `Set-WIConfig` if necessary.

### Debugging responses

If you are not receiving the output you expect you can turn on **verbose** output using the `ForceVerbose` option
as in the following:

```Powershell
Set-WIConfig -ForceVerbose $true
```

Then when you execute a command you should see details of all the API calls that are being made.
