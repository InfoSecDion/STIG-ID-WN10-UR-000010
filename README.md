# STIG-ID-WN10-UR-000010

## Synopsis
This PowerShell script ensures that the maximum size of the Windows Application event log is at least 32768 KB (32 MB).

## Notes
- **Author**: Dion Alexander
- **LinkedIn**: 
- **GitHub**: 
- **Date Created**: 2025-11-20
- **Last Modified**: 2025-11-20
- **Version**: 1.0
- **CVEs**: N/A
- **Plugin IDs**: N/A
- **STIG-ID**: WN10-AC-000005
  
## Tested On
- **Date(s) Tested**: 
- **Tested By**: 
- **Systems Tested**: 
- **PowerShell Ver.**: 

## Usage
Put any usage instructions here.

Example syntax:

```powershell
# STIG: Configure "Access this computer from the network" (SeNetworkLogonRight)

# Define the policy name and groups to include
$PolicyName = "SeNetworkLogonRight"
$Accounts    = @("Administrators", "Remote Desktop Users")
$AccountsVal = $Accounts -join ", "

# Prepare temp config path
$configPath = "C:\Temp\secpol.cfg"

# Ensure temp folder exists
if (-not (Test-Path -Path "C:\Temp")) {
    New-Item -ItemType Directory -Path "C:\Temp" -Force | Out-Null
}

# Export current policy
Write-Host "Exporting current security policy..."
secedit /export /cfg $configPath | Out-Null

# Validate export
if (-not (Test-Path -Path $configPath)) {
    Write-Error "Failed to export security policy. Check permissions."
    exit 1
}

# Load the config file
$config = Get-Content $configPath

# Update or add the user right
$policyFound = $false
for ($i = 0; $i -lt $config.Count; $i++) {
    if ($config[$i] -match "^\s*$PolicyName\s*=") {
        $config[$i] = "$PolicyName = $AccountsVal"
        $policyFound = $true
        break
    }
}

if (-not $policyFound) {
    $config += "$PolicyName = $AccountsVal"
}

# Save updated config
Set-Content -Path $configPath -Value $config

# Apply updated policy
Write-Host "Applying updated security policy..."
secedit /configure /db "C:\Windows\security\Database\secedit.sdb" /cfg $configPath /areas USER_RIGHTS | Out-Null

# Cleanup
if (Test-Path -Path $configPath) {
    Remove-Item -Path $configPath -Force
} else {
    Write-Warning "Temporary file '$configPath' was not found for cleanup."
}

Write-Host "Policy 'Access this computer from the network' has been configured successfully."
Write-Host "Running gpupdate /force..."
gpupdate /force | Out-Null
```
