# Automated-Packaging –PREVIEW–
This is a repository of JSON configuration files to showcase automated packaging with Cloudpaging technology.

As this is a preview Numecent does not offer professional support services for automated packaging at this time. All support efforts are community driven. Please visit our [community discussion forum here](https://numecent.freshdesk.com/support/discussions/forums/1000229144).

## Overview
Cloudpaging is a foundational technology framework and represents Numecent's vision to transform native software delivery, deployment and provisioning from the Cloud, both public and private, and on-premises. This patented technology makes it possible to lift and shift existing client applications to a new operating environment without all the hassle and expense of upgrading to new versions of your existing software.

The Cloudpaging Studio is where the science begins in the form of application packaging. The Studio prepares the application for automated deployment, updates, and access settings based upon the predetermined permission levels within your organization. You can package your apps for Windows 7 and easily lift and shift them over to Windows 10. 

For more information, please visit [www.numecent.com](https://www.numecent.com/).

## Environment Setup
In order to ensure the best compatibility and performance with automated packaging, it is necessary to take the following steps to setup your packaging environment. 

These steps assume that a virtual machine is already established with Cloudpaging Studio Installed.

1. Open PowerShell as an administrator in your virtual environment and run the following command: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`

2. Run the included PowerShell script: `CloudpagingStudio-prep.ps1`. 
This script automates a lot of the environment set up requirements, such as disabling Windows Updates and disabling UAC.

3. JSON configuration files assume a basic project structure. This project structure can be overridden using the command line parameters discussed below but if you wish to follow the default structure, do the following: 
	* Create a folder in C drive called “NIP_software”. 
	* Copy the folder from GitHub (containing the JSON configuration file) into this directory.
	* Within the JSON folder, Create two folders. 
		+ One folder named `Installer_Cfg`, which is where the application's installer will be placed. 
		* The other named `Output` which is where the output will be generated by default.

4. Now the power shell script can be executed using the following command:
```
powershell.exe -noexit "& ‘path\to\studio-nip.ps1' -config_file_path 'c:\NIP_software\YourFolderName\Name_Of_Your_Config_File.json'"
```

5. If you wish to use the additional parameters discussed below, the command structure would look like this:
```
powershell.exe -noexit "& ‘path\to\studio-nip.ps1' -config_file_path 'C:\NIP_software\YourFolderName\Name_Of_Your_Config_File.json'" -installer_path 'path\to\installer'
```

## How to use Automated-Packaging Instructions
Cloudpaging Studio comes with a set of interfaces and configurations to allow for automated packaging of Windows applications that can be installed silently without user interaction. 

Now, by using JSON config instruction files and Powershell, you can easily create and use automated packaging instructions that can be added to your desktop provisions workflows and CI/CD pipelines.

### JSON Config Instruction Setup

The file `Example_JSON_1.0.json` details the basic format of the JSON config instruction file.

Refer to the uploaded JSON config instruction files for more examples.

The following is an example with comments:

**Project Settings**
```
    "ProjectSettings": {
        "ProjectName": "Evernote_6-21-2-8717_32bit_x64_NLR_English_Rel1",
        "ProjectDescription": "Evernote is an app designed for note taking and organizing task management.",
        "IconFile": "C:\\Program Files (x86)\\Evernote\\Evernote\\Evernote.exe",
        "ProjectFileName": "",
        "WorkingFolder": "C:\\Program Files (x86)\\Evernote\\Evernote\\",
        "ProjectFolder": "C:\\NIP_software\\Evernote\\Output",
        "CommandLine": "C:\\Program Files (x86)\\Evernote\\Evernote\\Evernote.exe",
        "CommandLineParams": "",
        "TargetOS": [
            "Win7-x64",
            "Win8-x64"
        ]
    },
```
The Project Settings area is used to contain information pertaining to the application being captured. This information is saved as part of your Cloudpaging Studio project.

`CommandLine` is the path to the executable that will be cloudified.

**PreCaptureCommands**
```
"PreCaptureCommands": ["ECHO \"Commands here will be executed before capturing begins!\""],
```
Commands added to the array in this section will be copied to a bat file and executed before the capture process begins, each entry in the array is added as a new line to the bat file.

**CaptureSettings**
```
    "CaptureSettings": {
        "CaptureTimeoutSec": 300,
        "CaptureAllProcesses": false,
        "IgnoreChangesUnderInstallerPath": true,
        "ReplaceRegistryShortPaths": true,
        "RegistryExclusions": [
            "HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Windows\\CurrentVersion\\Uninstall",
            "HKEY_USERS\\\\.DEFAULT\\\\Software\\\\Microsoft\\\\Windows\\\\Windows Error Reporting",
            "HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Browser Helper Objects",
            "HKEY_CURRENT_USER\\SOFTWARE\\Policies",
            "HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\IdentityCRL",
            "HKEY_CURRENT_USER\\SOFTWARE\\Microsoft"
        ],
        "FileExclusions": [
            "%winDir%\\Installer\\*.msp",
            "%winDir%\\Installer\\*.msi"
        ],
        "ProcessExclusions": [],
        "ProcessInclusions": {
            "IncludeChildProccesses": true,
            "Include": []
        }
    },
```
The Capture Settings area contains the settings about how Cloudpaging Studio should capture the process.

`CaptureAllProcesses` specifies if studio should capture all the processes it detects when true, or to exclusively capture the installer and its child processes when false.

For most installations temporary files will be generated at the same path as the installer. If `IgnoreChangesUnderInstallerPath` is false, studio will capture these changes.

When `ReplaceRegistryShortPaths` is false, studio will leave registry path shortcuts in the project rather than containing the full registry key path.

Keys, files, and processes placed in their respective exclusion arrays will be ignored during the capture process, while processes added in the `Include` field will be added to the capture.

**CaptureCommands**
```
    "CaptureCommands": {
        "Enabled": true,
        "Prerequisites": {
            "Enabled": false,
            "Commands": []
        },
        "InstallerPrefix": "msiexec /i ",
        "InstallerPath": "c:\\NIP_software\\LibreOffice\\Installer_cfg\\LibreOffice_Win_x64.msi",
        "InstallerCommands": " /qn /norestart ",
        "PostInstallActions": {
            "Enabled": false,
            "Commands": []
        },
        "DebugMode": false
    },
```
In order to speed up installation and seamlessly package applications, bat file are generated to execute installers quietly. Additionally, any commands executed by these bat files will be included in the capture. If this feature is not desired it can be disabled by the `Enabled` field. In this case, the user would still need to click through the installer as a normal installation.

Different installers require different parameters in order to install silently. 

The format for how they are executed is:

`InstallerPrefix + InstallerPath + Installer Commands`

So in this example it would yield the command:

`msiexec /I “c:\NIP_software\LibreOffice\Installer_cfg\LibreOffice_Win_x64.msi”/qn /norestart`

Any commands placed in `PostInstallActions` would be executed after this command. Any Commands Placed in `Prerequisites` would be executed before this command.

By default, the following lines are added to the beginning of the bat files:
```
@ECHO OFF
SET SOURCE=%~dp0
SET SOURCE=%SOURCE:~0,-1%
```
Setting `DebugMode` to true will prevent this from happening.

**Modify Assets**
```
    "ModifyAssets": {
        "AddFiles": {},
        "ModifyKeys": {
            "Key1": {
                "Location": "HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Silverlight",
                "Keys": [
                    "UpdateMode=dword:00000002",
                    "UpdateConsentMode=dword:00000000"
                ]
            },
            "Key2": {
                "Location": "HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Silverlight",
                "Keys": [
                    "UpdateMode=dword:00000002",
                    "UpdateConsentMode=dword:00000000"
                ]
            }
        }
    },
```

```
    "ModifyAssets": {
        "AddFiles": {
            "File1": {
                "Name": "master_preferences",
                "Destination": "C:\\Program Files (x86)\\Google\\Chrome\\Application\\",
                "Content": [
                    "{",
                    "\"homepage\": \"http://www.google.com\",",
                    "\"homepage_is_newtabpage\": false,\"",
                    "\"browser\": {",
                    "\"show_home_button\": true,",
                    "\"check_default_browser\" : false\"",
                    "},",
                    "\"bookmark_bar\": {",
                    "\"show_on_all_tabs\": true",
                    "},",
```
Some applications require additional files to be generated in order to be installed successfully. The `AddFiles` section will create these files and then place them in the desired destination. This action will be included in the capture process.
Key modifications included in `ModifyKeys` will also be included in the capture.

**VirtualizationSettings**
```
    "VirtualizationSettings": {
        "DefaultDispositionLayer": 3,
        "DefaultServiceVirtualizationAction": "Register",
        "SandboxFileExclusions": [],
        "SandboxRegistryExclusions": []
    },
```
After the capture is complete, Cloudpaging Studio will use these settings in the virtualization process.

**SecurityOverrideSettings**
```
    "SecurityOverrideSettings": {
        "AllowAccessLayer4": {
            "AllowReadAndCopy": true,
            "Proccesses": []
        },
        "DenyAccessLayer3": []
    },
```
Processes included in `AllowAccessLayer4` Will be able to read and copy layer 4 resources if `AllowReadAndCopy` is true. Processes Included in `DenyAccessLayer3` will not be able to access resources at layer 3 or layer 4. Most applications should not require this feature.

**OutputSettings**
```
    "OutputSettings": {
        "EncryptionMethod": "None",
        "CompressionMethod": "None",
        "OutputFileNameNoExt": "",
        "FinalizeIntoSTP": true,
        "OutputFolder": "C:\\NIP_software\\GoogleChrome\\output\\"
    }
```
This area contains the output settings for Cloudpaging Studio.

The possible values for `CompressionMethod` are: “LZMA”, “None”

The possible values for `EncryptionMethod` are: “AES-256” ,”AES-256-Enhanced”, “None”

The optional parameter `OutputFileNameNoExt` allows for specifying a name for the output file. If this is left blank then studio will generate a name.

### Automated-Packaging Execution

To use these instructions, install Cloudpaging Studio, download the Powershell script called `studio-nip.ps1` and supply the JSON config along with these parameters:

**Mandatory:**
`-config_file_path ‘path\to\config\file’`
Path to the JSON configuration file.

**Optional:**
`-installer_path ‘path\to\installer’`
Using this parameter will override the value stored in CaptureCommands.InstallerPath

`-output_folder ‘path\to\output’`
Using this parameter will override the value stored in OutputSettings.OutputFolder

`-working_folder ‘path\to\working\folder\’`
Using this parameter will override value stored in ProjectSettings.WorkingFolder

`-debug_mode ‘true’`
Using this parameter includes more detailed output from the console, prevents generated files from being deleted upon packaging completion, and prevents dat files from being reverted to their pre-capture states.

## Uploading New Automated-Packaging Instructions
New Automated-Packaging instruction files can be uploaded by selecting Upload files, and then drag-and-dropping the folder containing the application JSON config file into the file upload area.

Please upload only the JSON configuration files to the appropriate application folder.

JSON files should be uploaded in the form:

`/Automated-Packaging/MyApplication/MyApplication_Packaging_Config_File.json`
