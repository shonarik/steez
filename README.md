# My Steez

This repository is all about how I do things. Anything that I want to remember and share belongs here.

## Windows Fresh Install

```
# This is not really a script but a collection of snippets. Many of these actions require reboot or are otherwise interactive.
# Follow the instructions on screen as it is the way of Windows.

# Install WSL2
# Manully install the latest release: https://github.com/microsoft/WSL/releases
# Reboot after manual installation and then jump into this script.
wsl --install ubuntu

# Install PowerShell
winget install --id Microsoft.PowerShell --source winget

# Enable Windows Sandbox
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
Enable-WindowsOptionalFeature -FeatureName "Containers-DisposableClientVM" -All -Online

# This WSB Config works with NVIDIA GPU (disables it in the sandbox), otherwise it crashes on start
@"<Configuration>
  <VGpu>Disable</VGpu>
  <Networking>Enable</Networking>
</Configuration>"@ | Set-Content -Path <Path to personal utilities>

# Install Docker CE (Windows)
$DockerUrl = "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1"
$DockerTempFile = New-TemporaryFile
Invoke-WebRequest $DockerUrl -OutFile "$($DockerTempFile).ps1" -UseBasicParsing
Invoke-Expression -Command "$($DockerTempFile).ps1"
Remove-Item "$($DockerTempFile).ps1"
Remove-Item $DockerTempFile

# Install Git
winget install --id Git.Git -e --source winget
winget install --id GitHub.GitHubDesktop -e --source winget

# Install pgAdmin
winget install --id PostgreSQL.pgAdmin --source winget

# Install VSCode
winget install --id Microsoft.VisualStudioCode --source winget

# Install AWS CLI
winget install --id Amazon.AWSCLI --source winget

# Prep for NVIDIA GPU in Docker
# Guid here: https://docs.nvidia.com/cuda/wsl-user-guide/index.html
# 1. Install latest GPU driver from here: https://www.nvidia.com/en-us/drivers/
# 2. 

```

* **Install Powershell** via `winget` because that is the recommended way: `winget install --id Microsoft.PowerShell --source winget`
* **Install Docker CE** via script from Microsoft at https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1. It appears that Docker does not publish this on their website, but Microsoft still does.

## Ubuntu Fresh Install

```
# Use Git for Windows Credentials Helper
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"

# Install Windows Hello Sudo
mkdir ~/wsl-hello-sudo
cd wsl-hello-sudo
wget http://github.com/nullpo-head/WSL-Hello-sudo/releases/latest/download/release.tar.gz
tar xvf release.tar.gz
cd release
./install.sh

# Install Docker
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Install .NET 9
sudo add-apt-repository ppa:dotnet/backports
sudo apt-get update
sudo apt-get install -y dotnet-sdk-9.0
```

### References

* [WSL2 - Releases](https://github.com/microsoft/WSL/releases)
* [Install PowerShell using WinGet (recommended)](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5#install-powershell-using-winget-recommended)
* [GitHub - Windows Containers - Install Docker CE](https://github.com/microsoft/Windows-Containers/blob/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1)
* [Install Windows Sandbox](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-install)
* [Git - Downloads](https://git-scm.com/downloads/win)
* [Get started using Git on WSL](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git)
* [Install .NET SDK or .NET Runtime on Ubuntu](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet9&pivots=os-linux-ubuntu-2404)

## Arch Linux

### Installation

From `pwsh` do the following:

```
wsl --update
wsl --install archlinux
```

### OOTBE

Arch starts as `root` without password. The nature of `wsl` is that you do not need the password for the default user. However, if you do want a `sudo` based experience then you will need a password. Additionally, I recomment that you create another user other than `root`. In other words, you should set a `root` password (you will likely never need this, but worhtwhile to avoid having to change your default `wsl` user to `root` in the future - i.e. you can `su` to `root` as needed), install `sudo`, create another user, add this user to `sudoers` and then set your new user as default.

```
# Set `root` password:
passwd

# Create a new user named `<user>` - of course you should replace `<user>` with the actual user name
useradd -m <user>
passwd <user>

# Update `pacman`
pacman -Syu

# Install `vi`
pacman -S vi

# Install `sudo`
pacman -S sudo

# Add `<user>` to `sudoers`
visudo # Within `visudo` add the following: `<user>   ALL=(ALL:ALL) ALL`
```

From `pwsh` set your new user as default:

```
wsl --manage archlinux --set-default-user <user>
```

From here, exit and restart `archlinux`. You should start in a prompt for `<user>`.

Next, we'll enable Windows Hello for `sudo` - in other words, you will use Windows Hello in place of your password for `<user>`.

```
# Install `wget`
sudo pacman -S wsl-hello-sudo-bin

# Installation
mkdir ~/wsl-hello-sudo
cd wsl-hello-sudo
wget https://github.com/lzlrd/wsl-hello-sudo/releases/download/v3.0.0/release.tar.gz
tar xvf release.tar.gz
cd release
./install.sh # Follow instructions
# Configure PAM when done
# Add `auth sufficient pam_wsl_hello.so` to `/etc/pam.d/sudo`
# Example
##%PAM-1.0
#auth            sufficient      pam_wsl_hello.so
#auth            include         system-auth
#account         include         system-auth
#session         include         system-auth
```

Graphics Acceleration

```
sudo pacman -S mesa
sudo pacman -S mesa-utils
sudo pacman -S vulkan-dzn
sudo pacman -S vulkan-icd-loader
```

**.bashrc**
```
export GALLIUM_DRIVER=d3d12
export LIBVA_DRIVER_NAME=d3d12
```

Other apps:

```
# Install nvim
sudo pacman -S nvim

# Install git
sudo pacman -S git

# Use Git for Windows Credentials Helper
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"

# Install Docker
sudo pacman -S docker

# Install Web Browser Integration
sudo pacman -S xdg-utils

# Install Azure CLI
sudo pacman -S azure-cli

# Install nodejs
sudo pacman -S nodejs

# Install opencode-ai
curl -fsSL https://opencode.ai/install | bash

```

### References

* [Install Arch Linux on WSL](https://wiki.archlinux.org/title/Install_Arch_Linux_on_WSL)
* [Users and groups](https://wiki.archlinux.org/title/Users_and_groups#Example_adding_a_user)
* [Sudo](https://wiki.archlinux.org/title/Sudo)
* [Docker](https://wiki.archlinux.org/title/Docker)

## Docker Without Docker Desktop

I prefer the command line so Docker Desktop does not offer much advantage. Additionally, it's not clear the switching between Linux and Windows is all that useful - I mean if I want to use a Linux container I just open WSL2. Windows?- well, that's the default.
