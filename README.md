<h1 align="center">💻 macOS ESP - Automated Setup</h1>

<div align="center">
  <p>
    <a href="https://twitter.com/UgurKocDe">
      <img src="https://img.shields.io/badge/Follow-@UgurKocDe-1DA1F2?style=flat&logo=x&logoColor=white" alt="Twitter Follow"/>
    </a>
    <a href="https://www.linkedin.com/in/ugurkocde/">
      <img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin" alt="LinkedIn"/>
    </a>
    <img src="https://img.shields.io/github/license/ugurkocde/IntuneAssignmentChecker?style=flat" alt="License"/>
  </p>
  <p>
    <img src="https://img.shields.io/github/downloads/ugurkocde/macOSESP/total.svg" alt="Github All Releases"/>
  </p>
</div>

![macOSESP](/media/macOSESP.png)

A customizable SwiftDialog-based Enrollment Status Page solution for macOS devices, providing a smooth onboarding experience for your users.

> ⚠️ **Preview Release Notice**
>
> The customizations of the UI and the ESP steps will be released soon.
>
> This project is currently in preview/beta phase. While it's functional, you may encounter bugs or incomplete features. We encourage you to:
>
> - Test and provide feedback
> - Report any issues you find
> - Suggest improvements
> - Share your experience
>
> Your feedback will help improve this tool for everyone!

> 🚀 **Quick Start Guide for the preview release**
>
> Follow these simple steps to get started:
>
> 1. 📥 Download the latest PKG file from [Releases](https://github.com/ugurkocde/macOSESP/releases/latest)
> 2. 📱 Deploy the PKG to a test device through Microsoft Intune
> 3. ⚡ The setup process will start automatically after installation

## 🌟 Table of Contents

- [🌟 Table of Contents](#-table-of-contents)
- [🌟 Features](#-features)
- [📋 Prerequisites](#-prerequisites)
- [🛠 Configuration](#-configuration)
  - [UI Customization](#ui-customization)
    - [UI customization](#ui-customization-1)
    - [ESP steps](#esp-steps)
  - [Script Configuration](#script-configuration)
- [📦 Building the Package](#-building-the-package)
- [🚀 Deployment](#-deployment)
  - [Pre-install script](#pre-install-script)
  - [Post-install script](#post-install-script)
- [📝 Logging](#-logging)
- [🤝 Contributing](#-contributing)
- [⚖️ License](#️-license)

## 🌟 Features

- Custom UI with progress tracking
- Configurable setup phases
- User-friendly interface with device information display
- Support for company branding
- Automated script execution
- Detailed logging

## 📋 Prerequisites

- [Packages](http://s.sudre.free.fr/Software/Packages/about.html) installed on your admin machine
- Latest version of [SwiftDialog](https://github.com/swiftDialog/swiftDialog/releases)
- macOS device with admin privileges

## 🛠 Configuration

### UI Customization

Open `SwiftDialog_IntunePKG\Payload\Application Support\SwiftDialogResources\swiftdialog.json` in the editor of your choice.

#### UI customization

```json
  "message": "Hello {userfullname}, welcome to COMPANY! We're currently configuring your Mac for optimal performance.",
  "icon": "/Library/Application Support/SwiftDialogResources/icons/CompanyPortal.png",
  "title": "macOS Setup - COMPANY Onboarding",
  "infobox": "We're setting up your Mac with essential applications and settings.\n\n**Note:** This process will take a few minutes and your Mac will restart automatically when finished. Please do not interrupt the setup.\n\n**Device Info:**\n- **Name:** {computername}\n- **Serial:** {serialnumber}\n- **OS Version:** {osversion}",
  "button1text": "Setup in Progress...",
  "button1disabled": true,
  "blurscreen": true,
  "ontop": true,
  "width": "80%",
  "height": "80%",
  "progress": "1",
  "progresstext": "Initializing setup...",
  "helpmessage": "Scan the QR code for help",
  "helpimage": "qr=https://www.COMPANY.com/it-support"
```

#### ESP steps

```json
"listitem": [
    {
      "title": "Company Portal",
      "icon": "/Library/Application Support/SwiftDialogResources/icons/CompanyPortal.png",
      "status": "pending",
      "statustext": "Pending"
    },
    .
    .
    .
    {
      "title": "Reboot",
      "icon": "SF=arrow.counterclockwise",
      "status": "pending",
      "statustext": "Pending"
    }
  ]
```

### Script Configuration

Scripts are stored in `SwiftDialog_IntunePKG\Payload\Application Support\SwiftDialogResources\scripts`. The number at the beginning of a script defines the execution order.

## 📦 Building the Package

1. Open `SwiftDialog_IntunePKG\Swift Dialog Onboarding\Swift Dialog Onboarding.pkgproj` with Packages
2. Update Dialog.app:
   - Remove existing Dialog.app from `Library/Application Support/Dialog`
   - Download latest [Dialog.app](https://github.com/swiftDialog/swiftDialog/releases)
   - Add new Dialog.app to the package
3. Verify all paths are correct (folders should appear blue, not red)
4. Click `Build` to create the package

## 🚀 Deployment

### Pre-install script

Replace `PKG_URL` with current version from Github repo.
[Download Pre Script](<https://github.com/microsoft/shell-intune-samples/blob/master/macOS/Config/Swift%20Dialog%20(PKG)/IntuneScripts/intunePre.sh>)

```bash
#!/bin/bash

############################################################################################
##
## Pre-install Script for Swift Dialog
##
## VER 1.0.0
##
############################################################################################

# Define any variables we need here:
logDir="/Library/Application Support/Microsoft/IntuneScripts/Swift Dialog"
DIALOG_BIN="/path/to/SwiftDialog"  # Set this to the path where SwiftDialog is expected to be installed
PKG_PATH="/var/tmp/dialog.pkg"
PKG_URL="https://github.com/swiftDialog/swiftDialog/releases/download/v2.5.4/dialog-2.5.4-4793.pkg"

# Start Logging
mkdir -p "$logDir"
exec > >(tee -a "$logDir/preinstall.log") 2>&1

if [ -e "/Library/Application Support/Dialog" ]; then
    echo "$(date) | PRE | Removing previous installation"
    rm -rf "/Library/Application Support/Dialog"
    rm -rf "/Library/Application Support/SwiftDialogResources"
    rm -rf "/usr/local/bin/dialog"
fi

# Download the SwiftDialog .pkg
curl -L -o "$PKG_PATH" "$PKG_URL"

# Install SwiftDialog from the downloaded .pkg file
sudo installer -pkg "$PKG_PATH" -target /

if [[ $? -eq 0 ]]; then
    echo "$(date) | POST | Swift Dialog has been installed successfully."
else
    echo "$(date) | ERROR | Swift Dialog installation failed."
    exit 1
fi


echo "$(date) | PRE | Completed Pre-install script"
exit 0
```

### Post-install script

Replace `dialog-2.5.4-4793.pkg` in `PKG_PATH` with current version.
[Download Post Script](<https://github.com/microsoft/shell-intune-samples/blob/master/macOS/Config/Swift%20Dialog%20(PKG)/IntuneScripts/intunePost.sh>)

```bash
#!/bin/bash

############################################################################################
##
## Post-install Script for Swift Dialog
##
## VER 1.0.0
##
############################################################################################

# Define any variables we need here:
logDir="/Library/Application Support/Microsoft/IntuneScripts/Swift Dialog"
DIALOG_BIN="/usr/local/bin/dialog"
PKG_PATH="/var/tmp/dialog-2.5.4-4793.pkg"

# Start Logging
mkdir -p "$logDir"
exec > >(tee -a "$logDir/postinstall.log") 2>&1

# Check if SwiftDialog is installed
if [[ ! -f "$DIALOG_BIN" ]]; then
  echo "$(date) | POST | Swift Dialog is not installed [$DIALOG_BIN]. Installing now..."

  # Install SwiftDialog from the .pkg file
  if [[ -f "$PKG_PATH" ]]; then
    sudo installer -pkg "$PKG_PATH" -target /

    if [[ $? -eq 0 ]]; then
      echo "$(date) | POST | Swift Dialog has been installed successfully."
    else
      echo "$(date) | ERROR | Swift Dialog installation failed."
      exit 1
    fi
  else
    echo "$(date) | ERROR | Package file not found at $PKG_PATH. Exiting."
    exit 1
  fi
else
  echo "$(date) | POST | Swift Dialog is already installed."
fi

# Wait for Desktop
until ps aux | grep /System/Library/CoreServices/Dock.app/Contents/MacOS/Dock | grep -v grep &>/dev/null; do
    echo "$(date) |  + Dock not running, waiting [1] seconds"
    sleep 1
done
echo "$(date) | Dock is here, lets carry on"

# Run Swift Dialog
/usr/local/bin/dialog --jsonfile "/Library/Application Support/SwiftDialogResources/swiftdialog.json" --width 1280 --height 670 --blurscreen --ontop &

# Wait for Swift Dialog to start
START=$(date +%s) # Set the start time so we can calculate how long we've been waiting
echo "$(date) | POST | Waiting for Swift Dialog to Start..."
# Loop for 1 minutes (60 seconds)
until ps aux | grep /usr/local/bin/dialog | grep -v grep &>/dev/null; do
    # Check if the 60 seconds have passed
    if [[ $(($(date +%s) - $START)) -ge 60 ]]; then
        echo "$(date) | POST | Failed: Swift Dialog did not start within 60 seconds"
        exit 1
    fi
    echo -n "."
    sleep 1
done
echo "OK"

echo "$(date) | POST | Processing scripts..."
for script in /Library/Application\ Support/SwiftdialogResources/scripts/*.*; do
  echo "$(date) | POST | Executing [$script]"
  xattr -d com.apple.quarantine "$script" >/dev/null 2>&1
  chmod +x "$script" >/dev/null 2>&1
  nice -n 20 "$script"
done

# Once we're done, we should write a flag file out so that we don't run again
sudo touch "$logDir/onboardingComplete"
exit 0
```

## 📝 Logging

Logs are stored in:

- Tool logs: `/var/tmp/swiftdialog.log`
- Script logs: `/var/tmp/script_[number].log`
- Installation logs: `/Library/Application Support/Microsoft/IntuneScripts/Swift Dialog/`

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a PR.

## ⚖️ License

MIT License - see the [LICENSE](LICENSE) file for details.
