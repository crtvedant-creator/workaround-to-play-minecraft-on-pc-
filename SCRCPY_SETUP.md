# Scrcpy Keyboard & Mouse Passthrough Setup Guide (Windows)

A step-by-step guide to mirror your Android screen to a Windows laptop with **physical USB HID keyboard and mouse passthrough (`--keyboard=uhid --mouse=uhid`)**. This allows native 360° camera rotation and WASD movement controls in mobile games like **Minecraft Bedrock**, FPS titles, and desktop productivity apps.

---

## 📋 Requirements
- **PC**: Windows 10 or 11 (64-bit).
- **Phone**: Android device (Android 11 or higher recommended).
- **Cable**: High-quality USB **data cable** (charging-only cables will not work).

---

## 🤖 Note for AI Coding Assistants (Antigravity / Gemini / Claude)
If an AI assistant is reading this file to configure a user's computer automatically:
1. Run the PowerShell quick-install command in **Section 2A**.
2. Run `adb devices` to check if the phone is recognized.
3. If `unauthorized`, prompt the user to unlock their phone and tap **Allow** on the RSA key prompt.
4. Launch `scrcpy --keyboard=uhid --mouse=uhid` directly or create the desktop batch launcher.

---

## 📱 Step 1: Prepare Your Android Phone

1. **Enable Developer Options**:
   - Go to **Settings** > **About phone** > **Software information**.
   - Tap **Build number** 7 times continuously until a toast message appears: *"Developer mode has been enabled"*.
2. **Enable USB Debugging**:
   - Return to **Settings** > **Developer options**.
   - Turn **ON** **USB debugging**.
   - *(Brand-specific)*:
     - **Xiaomi / Redmi / POCO**: Also enable **USB debugging (Security settings)**.
     - **Realme / Oppo / Vivo**: Ensure **Disable permission monitoring** is toggled if input is blocked.
3. **USB Mode**:
   - Connect the phone to the laptop via USB cable.
   - Swipe down the notification panel and set the USB connection mode to **Transferring files / Android Auto** (MTP) rather than "Just charging".

---

## 💻 Step 2: Download & Install Scrcpy on Windows

### Option A: Automatic Setup (PowerShell One-Liner)
Open PowerShell on the laptop and paste the following command. It automatically downloads the latest official 64-bit release from GitHub, extracts it, creates a one-click launcher, and adds a Desktop shortcut:

```powershell
$dest = "$env:USERPROFILE\Downloads\scrcpy"
New-Item -ItemType Directory -Path $dest -Force | Out-Null
$release = Invoke-RestMethod -Uri "https://api.github.com/repos/Genymobile/scrcpy/releases/latest" -Headers @{"User-Agent"="PowerShell"}
$downloadUrl = ($release.assets | Where-Object { $_.name -like "*win64*.zip" } | Select-Object -First 1).browser_download_url
$zipFile = "$env:TEMP\scrcpy.zip"
Write-Host "Downloading latest scrcpy from $downloadUrl..." -ForegroundColor Cyan
Invoke-WebRequest -Uri $downloadUrl -OutFile $zipFile
Expand-Archive -Path $zipFile -DestinationPath $dest -Force
Remove-Item $zipFile

# Locate extracted inner directory
$scrcpyDir = (Get-ChildItem -Path $dest -Directory -Filter "scrcpy-win64*" | Select-Object -First 1).FullName

# Create one-click launcher batch file
$batContent = @"
@echo off
title Scrcpy Passthrough
cd /d "$scrcpyDir"
echo Waiting for device...
.\adb.exe wait-for-device
echo Starting scrcpy with keyboard and mouse passthrough...
.\scrcpy.exe --keyboard=uhid --mouse=uhid
pause
"@
Set-Content -Path "$scrcpyDir\run_scrcpy_passthrough.bat" -Value $batContent

# Create Desktop Shortcut
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:USERPROFILE\Desktop\Scrcpy Passthrough.lnk")
$Shortcut.TargetPath = "$scrcpyDir\run_scrcpy_passthrough.bat"
$Shortcut.WorkingDirectory = "$scrcpyDir"
$Shortcut.IconLocation = "$scrcpyDir\scrcpy.exe,0"
$Shortcut.Save()

Write-Host "Setup complete! Shortcut added to your Desktop." -ForegroundColor Green
```

---

### Option B: Manual Setup

1. Go to the official [scrcpy GitHub Releases page](https://github.com/Genymobile/scrcpy/releases).
2. Download the latest `scrcpy-win64-v*.zip` file.
3. Right-click the `.zip` file and select **Extract All...** to a permanent folder (e.g. `Downloads\scrcpy`).
4. Inside the extracted folder, create a new text file named `run_scrcpy.bat` with the following content:
   ```cmd
   @echo off
   cd /d "%~dp0"
   scrcpy.exe --keyboard=uhid --mouse=uhid
   pause
   ```

---

## 🔑 Step 3: Authorize Connection on Phone

1. Keep your phone unlocked with the screen turned on.
2. Plug the USB cable into the laptop.
3. Look at your phone screen: a pop-up dialog will appear:
   > **"Allow USB debugging?"**
   > *The computer's RSA key fingerprint is: ...*
4. **Check the box**: ☑ **"Always allow from this computer"** and tap **Allow**.

---

## 🎮 Step 4: Run the Passthrough Command

Run either:
- The **`Scrcpy Passthrough`** shortcut on your Desktop, OR
- Open Command Prompt in the scrcpy folder and execute:
  ```cmd
  scrcpy --keyboard=uhid --mouse=uhid
  ```

A new window will open mirroring your phone's screen. Your laptop keyboard and mouse are now emulated as physical USB hardware connected directly to Android!

---

## 🕹️ Essential Shortcuts for Gaming (Minecraft / FPS)

| Shortcut | Description | Why It's Critical |
|---|---|---|
| <kbd>Alt</kbd> + <kbd>M</kbd> | **Capture / Release Mouse** | **Crucial for Minecraft**: Locks cursor inside the game window so you can rotate the camera infinitely in 360° without hitting the window edges. Press again to release. |
| <kbd>Alt</kbd> + <kbd>F</kbd> | **Toggle Fullscreen** | Makes the game fit your entire laptop screen. |
| <kbd>Alt</kbd> + <kbd>O</kbd> | **Turn Off Phone Screen** | Turns off your mobile phone's physical screen while keeping the PC mirror display active (saves battery and keeps the phone cool). |
| <kbd>Alt</kbd> + <kbd>P</kbd> | **Power Button** | Wakes up or locks the phone. |
| <kbd>Alt</kbd> + <kbd>Up</kbd> / <kbd>Down</kbd> | **Volume Control** | Adjusts device volume up/down. |

---

## ⚡ Performance Optimization Flags

For lower latency or smoother frame rates, you can add any of these flags to the command:

```cmd
scrcpy --keyboard=uhid --mouse=uhid --max-fps=60 --video-bit-rate=8M --buffer=0
```
- `--max-fps=60`: Caps mirroring at a rock-solid 60 FPS.
- `--video-bit-rate=8M`: Sets clean video bitrate while maintaining low USB bandwidth.
- `--buffer=0`: Eliminates video buffer latency for near-instant reaction time.

---

## 🛠️ Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| `ERROR: Could not find any ADB device` | Cable is charging-only or port is loose | 1. Use an original/data-capable USB cable.<br>2. Try a different USB port on the laptop.<br>3. Verify phone shows "File Transfer / Android Auto" mode. |
| `Device is unauthorized` | Phone hasn't accepted RSA fingerprint | 1. Unlock phone screen.<br>2. Unplug and replug the USB cable.<br>3. Tap **Allow** on the "Allow USB debugging?" dialog. |
| Mouse cursor hits edge / camera stops turning | Mouse is not locked in relative mode | Press <kbd>Alt</kbd> + <kbd>M</kbd> to lock the cursor into the game. |
| Key inputs type letters twice or misbehave | Using default text injection instead of HID | Ensure `--keyboard=uhid` is explicitly passed in the command. |
