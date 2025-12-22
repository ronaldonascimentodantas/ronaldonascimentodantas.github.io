<img src="dantas.io-logo.png" width="217" height="76"/>
<link rel="icon" href="dantas.io-favicon.png" type="image/x-icon">

# Complete Guide to Installing and Configuring Azure CLI on Windows 11, WSL, and Linux

## Introduction

The Azure Command-Line Interface (Azure CLI) is a cross-platform command-line tool that enables administrators, developers, and DevOps professionals to manage Azure resources directly from the terminal[1]. This comprehensive guide walks through the installation and configuration processes for Windows 11, Windows Subsystem for Linux (WSL), and native Linux environments. The current version of Azure CLI is 2.72.0[1].

## Prerequisites

Before installing Azure CLI, ensure your system meets the following requirements:

### General Requirements
- Administrator or sudo access on your machine
- Internet connectivity to download installation files
- A valid Azure subscription (for testing commands)

### System-Specific Requirements

**Windows 11:**
- Windows 11 or Windows 10 version 2004 (Build 19041) and higher
- PowerShell 5.1 or higher (included by default)
- WinGet (Windows Package Manager) available by default in Windows 11[1]

**WSL:**
- Windows 10 version 2004+ or Windows 11
- WSL 2 (recommended over WSL 1)
- Ubuntu, Debian, or another supported Linux distribution

**Linux:**
- Python 3.8.x or higher (for certain installation methods)
- Package manager (apt, dnf, zypper, or tdnf depending on distribution)
- Root or sudo access for package installation

---

## Installing Azure CLI on Windows 11

### Method 1: Using WinGet (Recommended)

WinGet is Microsoft's package manager for Windows and is available by default in Windows 11. This is the fastest and most straightforward installation method[1].

**Steps:**

1. Open PowerShell or Command Prompt
2. Execute the following command:

```powershell
winget install --exact --id Microsoft.AzureCLI
```

To install a specific version, add the `--version` parameter:

```powershell
winget install --exact --id Microsoft.AzureCLI --version 2.67.0
```

3. Wait for the installation to complete
4. Verify installation by opening a new terminal and running:

```powershell
az version
```

The output should display the Azure CLI version and its components in JSON format[1].

### Method 2: Using MSI Installer

The MSI (Microsoft Installer) package provides a graphical installation experience and is suitable for users who prefer GUI-based installation[1].

**Steps:**

1. Download the MSI installer:
   - **64-bit** (recommended): https://aka.ms/installazurecliwindowsx64
   - **32-bit**: https://aka.ms/installazurecliwindows

2. Double-click the downloaded `.msi` file
3. Follow the on-screen prompts
4. When prompted, select "Yes" to allow changes to your computer
5. Complete the installation by clicking "Install" and then "Finish"
6. Close and reopen your terminal for changes to take effect

**Important:** After installation, you must close and reopen any active terminal window to use Azure CLI[1].

### Method 3: Using PowerShell

For automated or scripted installations, use PowerShell to download and install the MSI[1]:

```powershell
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
```

For 64-bit installation, replace the URL with `https://aka.ms/installazurecliwindowsx64`[1].

### Method 4: ZIP Package (Preview)

The ZIP package allows installation without administrator privileges[1]:

1. Download: https://azcliprod.blob.core.windows.net/zip/azure-cli-x64.zip
2. Extract to a folder (e.g., `C:\azure-cli`)
3. Add the extracted folder to your PATH environment variable:
   - Open Settings → Environment Variables
   - Select "Path" and click "Edit"
   - Click "New" and add `<extracted-folder>\bin`
   - Restart your terminal

---

## Installing Azure CLI on WSL

Windows Subsystem for Linux allows running a native Linux environment on Windows. Azure CLI installation on WSL follows the Linux installation process[2].

### Step 1: Enable and Install WSL

1. Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This command installs WSL and Ubuntu by default[2].

2. When prompted, create a username and password for your Linux account
3. Restart your computer

To view available distributions, run:

```powershell
wsl --list --online
```

To install a specific distribution, run:

```powershell
wsl --install -d <DistributionName>
```

For example: `wsl --install -d Ubuntu-24.04`[2].

### Step 2: Install Azure CLI in WSL

Open your WSL terminal (Ubuntu or your chosen distribution) and execute:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

This script automatically handles all dependencies and installation steps[2].

### Step 3: Verify Installation

```bash
az --version
```

You should see output similar to:

```json
{
  "azure-cli": "2.72.0",
  "azure-cli-core": "2.72.0",
  "azure-cli-telemetry": "1.1.0",
  "extensions": {}
}
```

### Additional Configuration for WSL

For IntelliSense and enhanced development experience, install the "Azure CLI Tools" extension in Visual Studio Code. Ensure it's enabled for WSL[2].

---

## Installing Azure CLI on Linux

### Ubuntu/Debian Installation

#### Option 1: One-Command Installation (Recommended)

Execute the installation script provided by the Azure CLI team[1]:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### Option 2: Step-by-Step Installation

1. Update package lists and install prerequisites:

```bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```

2. Download and install the Microsoft signing key:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -sLS https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/microsoft.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/microsoft.gpg
```

3. Add the Azure CLI repository:

```bash
AZ_DIST=$(lsb_release -cs)
echo "Types: deb
URIs: https://packages.microsoft.com/repos/azure-cli/
Suites: ${AZ_DIST}
Components: main
Architectures: $(dpkg --print-architecture)
Signed-by: /etc/apt/keyrings/microsoft.gpg" | sudo tee /etc/apt/sources.list.d/azure-cli.sources
```

4. Install Azure CLI:

```bash
sudo apt-get update
sudo apt-get install azure-cli
```

### RHEL/CentOS Installation

1. Import the Microsoft repository key:

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```

2. Add the repository (for RHEL 9 or CentOS Stream 9):

```bash
sudo dnf install -y https://packages.microsoft.com/config/rhel/9.0/packages-microsoft-prod.rpm
```

For RHEL 8:

```bash
sudo dnf install -y https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm
```

3. Install Azure CLI:

```bash
sudo dnf install azure-cli
```

### SUSE/OpenSUSE Installation

1. Import the Microsoft signing key:

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```

2. Add the repository:

```bash
sudo zypper addrepo --name 'Azure CLI' --check https://packages.microsoft.com/yumrepos/azure-cli azure-cli
```

3. Install:

```bash
sudo zypper install --from azure-cli azure-cli
```

Input `2` when prompted to continue by ignoring some dependencies[1].

### Azure Linux (CBL-Mariner) Installation

1. Install trusted certificates:

```bash
sudo tdnf install ca-certificates
```

2. Install Azure CLI:

```bash
sudo tdnf install azure-cli
```

### Installing Specific Versions

To install a particular version on Debian/Ubuntu:

```bash
AZ_DIST=$(lsb_release -cs)
AZ_VER=2.51.0
sudo apt-get install azure-cli=${AZ_VER}-1~${AZ_DIST}
```

For RHEL/CentOS:

```bash
sudo dnf install azure-cli-<version>-1.el7
```

### Verifying Installation on Linux

```bash
az --version
```

---

## Azure CLI Configuration

### Configuration File Locations

- **Linux/macOS:** `$HOME/.azure/config`
- **Windows:** `%USERPROFILE%\.azure`

### Method 1: Interactive Configuration with az init

The `az init` command provides an interactive setup wizard[3]:

```bash
az init
```

This guides you through common configurations such as:
- Output format preferences
- Log file settings
- Telemetry data collection
- Default resource group and location

### Method 2: Setting Configuration with az config

Set individual configuration values:

```bash
az config set defaults.location=westus2 defaults.group=MyResourceGroup
```

Common configuration keys:

| Configuration Key | Purpose |
|-------------------|---------|
| `defaults.group` | Default resource group for all commands |
| `defaults.location` | Default Azure region |
| `defaults.web` | Default app name for webapp commands |
| `defaults.vm` | Default VM name |
| `core.output` | Output format (table, json, tsv, etc.) |
| `core.only_show_errors` | Suppress non-error messages |
| `logging.enable_log_file` | Enable file logging |

Example to set output format to JSON:

```bash
az config set core.output=json
```

### Method 3: Manual Configuration File Editing

Edit the configuration file directly with your text editor:

```bash
nano ~/.azure/config
```

Example configuration file:

```ini
[cloud]
name = AzureCloud

[core]
first_run = yes
output = table
collect_telemetry = yes

[logging]
enable_log_file = no
```

### Environment Variables

Configuration can also be set via environment variables using the format `AZURE_{SECTION}_{NAME}`:

```bash
export AZURE_CORE_OUTPUT=json
export AZURE_DEFAULTS_GROUP=MyResourceGroup
export AZURE_DEFAULTS_LOCATION=eastus
```

---

## Authentication and Login

### Interactive Login

The most straightforward authentication method uses your default web browser:

```bash
az login
```

This command:
1. Opens your default browser automatically
2. Displays the Azure sign-in page
3. Prompts you to enter your account credentials
4. Returns an authentication token to the CLI[3]

### Device Code Flow

For systems without a browser or in restricted environments, use device code authentication:

```bash
az login --use-device-code
```

Instructions appear in the terminal:
1. Copy the displayed authentication code
2. Navigate to https://aka.ms/devicelogin in any browser
3. Enter the code when prompted
4. Sign in with your Azure account[3]

### Verifying Active Subscription

After login, check your active subscription:

```bash
az account show
```

Output example:

```json
{
  "environmentName": "AzureCloud",
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "isDefault": true,
  "name": "My Subscription",
  "state": "Enabled",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "user": {
    "name": "user@example.com",
    "type": "user"
  }
}
```

### Switching Subscriptions

If you have multiple subscriptions:

```bash
az account list -o table
az account set --subscription "Subscription Name"
```

---

## Managing Extensions

Extensions provide additional commands and functionality beyond the core Azure CLI[4].

### Listing Extensions

View available extensions:

```bash
az extension list-available --output table
```

View installed extensions:

```bash
az extension list
```

### Installing Extensions

Install a specific extension:

```bash
az extension add --name <extension-name>
```

Example - Install the Azure DevOps extension:

```bash
az extension add --name azure-devops
```

### Updating Extensions

Update an installed extension:

```bash
az extension update --name <extension-name>
```

Update all extensions:

```bash
az extension update --name <extension-name> --upgrade-all
```

### Removing Extensions

Uninstall an extension:

```bash
az extension remove --name <extension-name>
```

### Dynamic Extension Installation

Starting with CLI version 2.10.0, extensions can be automatically installed when their commands are used. This feature is enabled by default[4].

---

## Troubleshooting

### Common Issues and Solutions

#### PATH Variable Not Set (Windows)

**Problem:** "az command not found" after installation

**Solution:** Close and reopen your terminal window to refresh the PATH environment variable[1].

#### Proxy Blocks Connection

**Problem:** Installation fails due to proxy restrictions

**Windows Configuration:**

In PowerShell:

```powershell
(New-Object System.Net.WebClient).Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
```

**Linux Configuration:**

Set proxy environment variables:

```bash
export HTTP_PROXY=http://[proxy]:[port]
export HTTPS_PROXY=https://[proxy]:[port]
```

For basic authentication:

```bash
export HTTP_PROXY=http://[username]:[password]@[proxy]:[port]
export HTTPS_PROXY=https://[username]:[password]@[proxy]:[port]
```

Required HTTPS endpoints:
- `https://aka.ms/`
- `https://azcliprod.blob.core.windows.net/`
- `https://packages.microsoft.com`

#### Updating Azure CLI

Update to the latest version:

```bash
az upgrade
```

This command updates all installed extensions by default[1].

#### Uninstalling Azure CLI

**Windows:**
1. Open Settings → Apps → Installed apps
2. Search for "Azure CLI"
3. Click on "Microsoft CLI 2.0 for Azure"
4. Select "Uninstall"

**Linux (Ubuntu/Debian):**

```bash
sudo apt-get remove -y azure-cli
sudo rm /etc/apt/sources.list.d/azure-cli.sources
```

**Linux (RHEL/CentOS):**

```bash
sudo dnf remove azure-cli
sudo rm /etc/yum.repos.d/azure-cli.repo
```

#### WSL Installation Issues

If Azure CLI fails on WSL, the issue may be with WSL itself, not the CLI installation:

1. Run an identical installation on a native Linux VM to verify
2. Update Windows to the latest version
3. Check WSL GitHub issues for known problems
4. File a new issue if the problem persists

---

## Best Practices

1. **Use Package Managers:** Always install Azure CLI through your distribution's package manager when available for automatic updates[1].

2. **Secure Credential Storage:** Never hardcode credentials in scripts. Use service principals, managed identities, or device code authentication[3].

3. **Enable Tab Completion:** Configure shell-specific tab completion for faster command entry:

   **For Bash/Zsh:**
   ```bash
   eval "$(register-python-argcomplete az)"
   ```

   **For PowerShell:** Use the built-in `Register-ArgumentCompleter` cmdlet included in Azure CLI 2.49.0+[1].

4. **Regular Updates:** Keep Azure CLI updated to receive the latest features and security patches:

   ```bash
   az upgrade
   ```

5. **Version Control:** Pin extension versions in automation scripts to ensure consistency:

   ```bash
   az extension add --name <extension-name> --version <version>
   ```

6. **Monitor Configuration:** Review your configuration periodically to ensure it aligns with your organization's standards[3].

---

## Conclusion

Installing and configuring Azure CLI on Windows 11, WSL, and Linux is straightforward with multiple installation methods available for each platform. The key steps involve:

1. Selecting the appropriate installation method for your platform
2. Installing Azure CLI using the recommended approach (WinGet for Windows, native package managers for Linux)
3. Authenticating with your Azure account using `az login`
4. Configuring default values and preferences with `az config set` or `az init`
5. Installing additional extensions as needed for specialized tasks

With Azure CLI properly configured, you can efficiently manage Azure resources, automate deployments, and streamline your cloud operations from the command line.

<img src="Azure_CLI-Install-infograph.png"/>

---

## References

[1] Microsoft Learn. (2025, December 1). Install the Azure CLI. Retrieved from https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

[2] Microsoft Learn. (2025, November 4). Installing the Azure CLI on the Linux Subsystem in Windows 11. Retrieved from https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest

[3] Microsoft Learn. (2025, September 1). Azure CLI configuration options. Retrieved from https://learn.microsoft.com/en-us/cli/azure/azure-cli-configuration?view=azure-cli-latest

[4] Microsoft Learn. (2025, November 3). Sign in with Azure CLI at a command line. Retrieved from https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli-interactively?view=azure-cli-latest

[5] Microsoft Learn. (2025, November 3). Manage Azure CLI Extensions - Install, Update, and Remove. Retrieved from https://learn.microsoft.com/en-us/cli/azure/azure-cli-extensions-overview?view=azure-cli-latest

[6] Virtual Packets. (2024, February 29). Windows Subsystem for Linux (WSL2) & Azure CLI. Retrieved from https://www.virtualpackets.com/windows-subsystem-for-linux-wsl2-azure-cli/

[7] DevOpsCube. (2025, June 10). Setting Up Azure CLI on Ubuntu Linux. Retrieved from https://devopscube.com/setting-azure-cli-ubuntu-linux/

[8] Liquid Web. (2024, November 24). How to Install Azure CLI on Linux (AlmaLinux). Retrieved from https://www.liquidweb.com/help-docs/install-azure-cli-linux-almalinux/

---

## *Disclaimer:*

- This guide is provided as-is for educational purposes. Always refer to the official Microsoft Learn documentation for the most current information and support.
- This comprehensive guide was compiled from official Microsoft Learn documentation and industry best practices for Azure CLI installation and configuration across multiple platforms.
- Azure CLI Version Covered:** 2.72.0+