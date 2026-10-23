# Install n8n on Windows Without Docker

This guide explains how to install **n8n directly on Windows without Docker Desktop** and configure it as a **Windows Service** so it starts automatically when Windows boots.

---

## Requirements

You need:

- Windows 10 or Windows 11
- Node.js
- npm
- n8n
- WinSW for running n8n as a Windows Service

---

# 1. Install Node.js

Download and install Node.js from:

```text
https://nodejs.org/
```

Use a supported Node.js version for your installed n8n version.

During installation, make sure Node.js is added to your Windows `PATH`.

After installation, open PowerShell and check:

```powershell
node -v
```

Example:

```text
v24.22.1
```

Check npm:

```powershell
npm -v
```

---

# 2. Fix PowerShell npm Script Error

If PowerShell shows an error like:

```text
npm.ps1 cannot be loaded because running scripts is disabled on this system
```

run:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

PowerShell may ask:

```text
Do you want to change the execution policy?
```

Enter:

```text
Y
```

Close PowerShell and open it again.

Test:

```powershell
npm -v
```

npm should now work normally.

---

# 3. Install n8n

Install n8n globally:

```powershell
npm install -g n8n
```

Check the installed version:

```powershell
n8n --version
```

---

# 4. Start n8n Manually

Run:

```powershell
n8n
```

n8n should start on:

```text
http://localhost:5678
```

Open this address in your browser.

Your basic setup is:

```text
Windows
   │
   ├── Node.js
   │
   ├── npm
   │
   └── n8n
        │
        └── localhost:5678
```

---

# 5. Find the n8n Executable

Before creating the Windows Service, find where n8n is installed:

```powershell
where.exe n8n
```

Example output:

```text
C:\Users\USERNAME\AppData\Roaming\npm\n8n
C:\Users\USERNAME\AppData\Roaming\npm\n8n.cmd
```

For the Windows Service, use the `.cmd` file:

```text
C:\Users\USERNAME\AppData\Roaming\npm\n8n.cmd
```

Replace `USERNAME` with your Windows username.

---

# 6. Install WinSW

WinSW allows a normal executable or command to run as a Windows Service.

Download WinSW from:

```text
https://github.com/winsw/winsw/releases
```

Download the Windows x64 executable.

Example filename:

```text
WinSW-x64.exe
```

Create a folder:

```powershell
mkdir C:\n8n-service
```

Move the WinSW executable into:

```text
C:\n8n-service
```

Go to the directory:

```powershell
cd C:\n8n-service
```

---

# 7. Rename WinSW

Rename:

```text
WinSW-x64.exe
```

to:

```text
n8n-service.exe
```

Using PowerShell:

```powershell
Rename-Item "WinSW-x64.exe" "n8n-service.exe"
```

Check the files:

```powershell
dir
```

You should see:

```text
n8n-service.exe
```

---

# 8. Create the WinSW Configuration

Create:

```text
n8n-service.xml
```

inside:

```text
C:\n8n-service
```

The final directory should look like:

```text
C:\n8n-service
│
├── n8n-service.exe
└── n8n-service.xml
```

---

# 9. Fix `.xml.txt` File Extension

If Windows creates:

```text
n8n-service.xml.txt
```

instead of:

```text
n8n-service.xml
```

rename it using PowerShell:

```powershell
Rename-Item "n8n-service.xml.txt" "n8n-service.xml"
```

Check:

```powershell
dir
```

You should now see:

```text
n8n-service.exe
n8n-service.xml
```

---

# 10. Configure `n8n-service.xml`

Add the following configuration:

```xml
<service>

  <id>n8n</id>

  <name>n8n Automation</name>

  <description>
    n8n Workflow Automation Service
  </description>

  <executable>C:\Windows\System32\cmd.exe</executable>

  <arguments>
    /c "C:\Users\USERNAME\AppData\Roaming\npm\n8n.cmd"
  </arguments>

  <workingdirectory>C:\n8n-service</workingdirectory>

  <startmode>Automatic</startmode>

  <onfailure action="restart" delay="10 sec"/>

  <logpath>C:\n8n-service\logs</logpath>

  <log mode="roll"></log>

</service>
```

Replace:

```text
USERNAME
```

with your actual Windows username.

For example:

```xml
<arguments>
    /c "C:\Users\john\AppData\Roaming\npm\n8n.cmd"
</arguments>
```

---

# 11. Install n8n as a Windows Service

Open **PowerShell as Administrator**.

Go to:

```powershell
cd C:\n8n-service
```

Install the service:

```powershell
.\n8n-service.exe install
```

---

# 12. Start the Service

Start n8n:

```powershell
.\n8n-service.exe start
```

Check the service status:

```powershell
.\n8n-service.exe status
```

A successful result should look similar to:

```text
Started
```

---

# 13. Check Windows Services

Press:

```text
Win + R
```

Enter:

```text
services.msc
```

Find:

```text
n8n Automation
```

It should show:

```text
Status: Running

Startup Type: Automatic
```

Because the startup mode is `Automatic`, n8n will start automatically when Windows starts.

---

# 14. Open n8n

Open:

```text
http://localhost:5678
```

You should now have access to the n8n interface.

---

# Final Architecture

```text
Windows Boot
    │
    ▼
Windows Service Manager
    │
    ▼
WinSW
    │
    ▼
cmd.exe
    │
    ▼
n8n.cmd
    │
    ▼
Node.js
    │
    ▼
n8n
    │
    ▼
localhost:5678
```

---

# Useful Commands

## Check Node.js

```powershell
node -v
```

## Check npm

```powershell
npm -v
```

## Check n8n

```powershell
n8n --version
```

## Find n8n Path

```powershell
where.exe n8n
```

## Go to Service Folder

```powershell
cd C:\n8n-service
```

## List Files

```powershell
dir
```

## Install Service

```powershell
.\n8n-service.exe install
```

## Start Service

```powershell
.\n8n-service.exe start
```

## Stop Service

```powershell
.\n8n-service.exe stop
```

## Restart Service

```powershell
.\n8n-service.exe restart
```

## Check Service Status

```powershell
.\n8n-service.exe status
```

## Uninstall Service

First stop it:

```powershell
.\n8n-service.exe stop
```

Then uninstall:

```powershell
.\n8n-service.exe uninstall
```

---

# Recommended Folder Structure

```text
C:\
│
├── n8n-service\
│   ├── n8n-service.exe
│   ├── n8n-service.xml
│   └── logs\
│
└── Users\
    └── USERNAME\
        └── AppData\
            └── Roaming\
                └── npm\
                    └── n8n.cmd
```

---

# Important Notes

### n8n must already work manually

Before configuring WinSW, confirm that this works:

```powershell
n8n
```

and that you can open:

```text
http://localhost:5678
```

Do not create the Windows Service until n8n works correctly from PowerShell.

---

### Check the n8n path

Always confirm your path with:

```powershell
where.exe n8n
```

Do not blindly copy another user's path.

---

### Run service commands as Administrator

Commands such as:

```powershell
.\n8n-service.exe install
```

should be executed from an Administrator PowerShell.

---

### Windows Service starts automatically

With:

```xml
<startmode>Automatic</startmode>
```

n8n starts when Windows starts.

You do not need to:

- open PowerShell
- manually execute `n8n`
- log in and start n8n yourself
- run Docker Desktop

---

# Troubleshooting

## npm.ps1 is blocked

Run:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Then restart PowerShell.

---

## n8n command is not found

Check:

```powershell
where.exe n8n
```

If nothing appears, reinstall:

```powershell
npm install -g n8n
```

---

## Service starts but n8n is not available

Check:

```text
C:\n8n-service\logs
```

Also test n8n manually:

```powershell
n8n
```

If n8n does not work manually, fix that problem before troubleshooting WinSW.

---

## Check if port 5678 is listening

Run:

```powershell
netstat -ano | findstr :5678
```

If n8n is running, you should see a listening connection for port `5678`.

---

# Result

After completing the setup:

- n8n runs directly on Windows
- Docker Desktop is not required
- n8n runs as a Windows Service
- n8n starts automatically after Windows boot
- WinSW can restart n8n after a failure
- logs are stored locally
- n8n is available at `http://localhost:5678`