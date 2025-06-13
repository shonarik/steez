# My Steez

This repository is all about how I do things. Anything that I want to remember and share belongs here.

## Windows Fresh Install

```
# Install WSL2
# Manully install the latest release: https://github.com/microsoft/WSL/releases
# Reboot after manual installation and then jump into this script.
wsl --install ubuntu

# Install PowerShell
winget install --id Microsoft.PowerShell --source winget

# Enable Windows Sandbox
Enable-WindowsOptionalFeature -FeatureName "Containers-DisposableClientVM" -All -Online

# Install Docker CE (Windows)
$DockerUrl = "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1"
$DockerTempFile = New-TemporaryFile
Invoke-WebRequest $DockerUrl -OutFile "$($DockerTempFile).ps1" -UseBasicParsing
Invoke-Expression -Command "$($DockerTempFile).ps1"
Remove-Item "$($DockerTempFile).ps1"
Remove-Item $DockerTempFile

```

* **Install Powershell** via `winget` because that is the recommended way: `winget install --id Microsoft.PowerShell --source winget`
* **Install Docker CE** via script from Microsoft at https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1. It appears that Docker does not publish this on their website, but Microsoft still does.

### References

* [WSL2 - Releases](https://github.com/microsoft/WSL/releases)
* [Install PowerShell using WinGet (recommended)](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5#install-powershell-using-winget-recommended)
* [GitHub - Windows Containers - Install Docker CE](https://github.com/microsoft/Windows-Containers/blob/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1)
* [Install Windows Sandbox]([https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-install))

## Docker Without Docker Desktop

I prefer the command line so Docker Desktop does not offer much advantage. Additionally, it's not clear the switching between Linux and Windows is all that useful - I mean if I want to use a Linux container I just open WSL2. Windows?- well, that's the default.
