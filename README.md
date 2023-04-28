# Install Docker Desktop For Windows

## PowerShell Installation
### Step 1. Enable the Windows Subsystem for Linux
Open PowerShell as Administrator and run:

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### Step 2. check Windows 10 version
Windows 10, version 1903 or higher is required to run WSL2
To check your version and build number, select Windows logo key + R, type winver, select OK.

### Step 3. Enable Virtual Machine feature

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> If you face any errors, it could be because Virtualization is not enabled on your laptop. Please restart, press Esc or F10 to boot into BIOS. Go to advanced system settings, and enable Intel VT, Intel VT-d or AMD-V( If you have an AMD processor). Save these settings and exit. And try step 3 again.

> BIOS has several other options. Please make sure to not change anything else, it could lead to your system not booting up and a visit to TechSpot.

> **Restart your machine to complete the WSL install and update to WSL 2.**

### Step 4. Download the Linux kernel update package
Update WSL to WSL2 by downloading it from the following link and installing it.

[WSL2 Linux kernel update package for x64 machines](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

### Step 5. Set WSL 2 as your default version

```
wsl --set-default-version 2
```

### Step 6. Install your Linux distribution of choice
> We don’t have access to Microsoft Store, so you will have to use Powershell to download it.We recommend Ubuntu 18.04. You could choose other distros too, from the following link [Distros that can be downloaded using Powershell](https://superuser.com/questions/1271682/is-there-a-way-of-installing-ubuntu-windows-subsystem-for-linux-on-win10-v170) 
- Go into some folder into which you want the file to be downloaded
```
cd <somefolder>
Invoke-WebRequest -Uri https://aka.ms/wsl-ubuntu-1804 -OutFile Ubuntu.appx -UseBasicParsing
```

- Once the download is complete, extract the package
```
Rename-Item .\Ubuntu.appx .\Ubuntu.zip
Expand-Archive .\Ubuntu.zip .\Ubuntu
```

- Go to the folder in your windows explorer, and right click on ubuntu1804.exe and run it as admin
Since this is your first run, You will then need to create a user account and password for your new Linux distribution. It does not need to be the same as your RACF ID and password but that is recommended.

> If you want to be able to launch the linux distro from anywhere, the following optional step can be done. Add your distro path to the Windows environment PATH (C:\Users\Administrator\Ubuntu for example), using PowerShell:
```
$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
[System.Environment]::SetEnvironmentVariable("PATH", $userenv + ";C:\Users\Administrator\Ubuntu", "User")
```

**CONGRATULATIONS! You've successfully installed and set up a Linux distribution that is completely integrated with your Windows operating system!**

### Step 7. Install Docker for Windows

- [Download Docker](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)

- Run the installation with all the defaults.  If you are running a supported system, Docker Desktop prompts you to enable WSL 2 during installation. Read the information displayed on the screen and enable WSL 2 to continue.


### Step 8. Open Docker Desktop
Docker should be running in the system tray. Click it to start using Docker.

## How to uninstall Docker Machine under Windows 10[ref](https://stackoverflow.com/questions/42161471/how-to-uninstall-docker-machine-under-windows-10)
1. go to `C:\Program Files\Docker`, by opening `cmd` as administrator.
2. run 
```
takeown /R /F *
```
3. run
```
ICACLS * /T /Q /C /RESET
```
**Don't run in `Program files` folder, otherwise you will go to bootloop after restart, go to `Docker` folder first.**

4. Create the small file with following content and saved with extension `.ps1` in Program files folder, and right click on it and Run with `Powershell`.
```
kill -force -processname 'Docker for Windows', com.docker.db, vpnkit, com.docker.proxy, com.docker.9pdb, moby-diag-dl, dockerd

try {
    ./MobyLinux.ps1 -Destroy
} Catch {}

$service = Get-WmiObject -Class Win32_Service -Filter "Name='com.docker.service'"
if ($service) { $service.StopService() }
if ($service) { $service.Delete() }
Start-Sleep -s 5
Remove-Item -Recurse -Force "~/AppData/Local/Docker"
Remove-Item -Recurse -Force "~/AppData/Roaming/Docker"
if (Test-Path "C:\ProgramData\Docker") { takeown.exe /F "C:\ProgramData\Docker" /R /A /D Y }
if (Test-Path "C:\ProgramData\Docker") { icacls "C:\ProgramData\Docker\" /T /C /grant Administrators:F }
Remove-Item -Recurse -Force "C:\ProgramData\Docker"
Remove-Item -Recurse -Force "C:\Program Files\Docker"
Remove-Item -Recurse -Force "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Docker"
Remove-Item -Force "C:\Users\Public\Desktop\Docker for Windows.lnk"
Get-ChildItem HKLM:\software\microsoft\windows\currentversion\uninstall | % {Get-ItemProperty $_.PSPath}  | ? { $_.DisplayName -eq "Docker" } | Remove-Item -Recurse -Force
Get-ChildItem HKLM:\software\classes\installer\products | % {Get-ItemProperty $_.pspath} | ? { $_.ProductName -eq "Docker" } | Remove-Item -Recurse -Force
Get-Item 'HKLM:\software\Docker Inc.' | Remove-Item -Recurse -Force
Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\Run -name "Docker for Windows" | Remove-Item -Recurse -Force
#Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\UFH\SHC | ForEach-Object {Get-ItemProperty $_.PSPath} | Where-Object { $_.ToString().Contains("Docker for Windows.exe") } | Remove-Item -Recurse -Force $_.PSPath
#Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\UFH\SHC | Where-Object { $(Get-ItemPropertyValue $_) -Contains "Docker" }
```
> [reference](https://grainger.atlassian.net/wiki/spaces/MLOPS/pages/44617205679/Install+Docker+Desktop+For+Windows)
> [Docker Desktop](https://grainger.atlassian.net/wiki/spaces/PD/pages/162935210008/Docker+Desktop)
