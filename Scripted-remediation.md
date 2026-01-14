<h1>🧪 Hands-On: Windows 11 Authenticated Vulnerability Scanning & Scripted Remediation</h1>

<p>
In this lab, I performed an <strong>authenticated vulnerability scan</strong> against a Windows 11 virtual machine using <strong>Tenable Vulnerability Management</strong>, intentionally introduced common Windows vulnerabilities, remediated them using <strong>PowerShell scripts</strong>, and validated risk reduction through rescanning.
</p>

<details>
  <summary><strong>Click to expand</strong></summary>

  <p>
    This content is hidden by default and shown when expanded.
  </p>

</details>


<hr />

<h2>☁️ Step 1: Provision a Windows 11 Virtual Machine</h2>

<p>
I provisioned a Windows 11 virtual machine and ensured secure credential practices.
</p>

<ul>
  <li>Created a Windows 11 VM</li>
  <li>Used a strong, non-default username and password</li>
  <li>Noted the VM name to avoid deleting the wrong resource later</li>
</ul>

<p>
Using weak or default credentials on exposed VMs frequently leads to compromise.
</p>

<img width="543" height="459" alt="image" src="https://github.com/user-attachments/assets/442f1582-9aff-4941-aab0-a944135d985b" />


<hr />

<h2>🔥 Step 2: Disable Windows Firewall</h2>

<p>
To ensure full scan visibility, I disabled the Windows Firewall on the VM.
</p>

<ul>
  <li>Opened Windows Firewall management (<code>wf.msc</code>)</li>
  <li>Disabled all firewall profiles</li>
</ul>


<hr />

<h2>🔐 Step 3: Enable Remote Administrative Access</h2>

<p>
Before scanning, I enabled remote administrative access for local accounts by modifying the Windows registry.
</p>

<p>
I ran the following PowerShell command <strong>as Administrator</strong>:
</p>

<pre>
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
-Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
</pre>

<p>
This allows local administrator accounts to authenticate remotely with full privileges during credentialed scans.
</p>

<hr />

<h2>🧩 Step 4: Create an Authenticated Scan in Tenable</h2>

<p>
I configured an authenticated vulnerability scan using a DISA STIG-aligned template.
</p>

<ul>
  <li>Selected the <strong>Windows 11 DISA STIG</strong> scan template</li>
  <li>Chose <strong>Internal Scanner</strong> (Cloud Scanner if Internal was unavailable)</li>
  <li>Entered the VM IP address</li>
  <li>Configured Windows credentials</li>
</ul>

<img width="1853" height="751" alt="image" src="https://github.com/user-attachments/assets/3bd5f21a-6073-494a-a99f-3e6a757ce030" />


<hr />

<h2>⚠️ Step 5: Introduce Windows Vulnerabilities</h2>

<p>
To simulate real-world risk, I intentionally introduced several common vulnerabilities.
</p>

<ul>
  <li>Installed an outdated version of Firefox</li>
  <li>Enabled SMBv1</li>
  <li>Enabled deprecated cryptographic protocols (SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1)</li>
</ul>

<img width="631" height="61" alt="image" src="https://github.com/user-attachments/assets/5d9d5c2c-9455-4cd7-8d9a-6d48542921bd" />


<hr />

<h2>🔄 Step 6: Restart the Virtual Machine</h2>

<p>
I restarted the VM to ensure all configuration changes were applied.
</p>


<hr />

<h2>📈 Step 7: Run the Authenticated Scan</h2>

<p>
I ran the authenticated scan again to detect the newly introduced vulnerabilities.
</p>

<ul>
  <li>Observed insecure software findings (Firefox)</li>
  <li>Detected SMBv1 exposure</li>
  <li>Identified deprecated SSL/TLS protocols</li>
</ul>

<img width="1859" height="145" alt="image" src="https://github.com/user-attachments/assets/c728aa85-7564-4a1a-adb0-0384507af4e4" />


<hr />

<h2>🛠️ Step 8: Scripted Remediation</h2>

<p>
I remediated the vulnerabilities using PowerShell scripts.
</p>

<ul>
  <li>Uninstalled insecure Firefox version</li>
  <li>Disabled SMBv1</li>
  <li>Disabled deprecated SSL/TLS protocols</li>
</ul>

<p>
Example remediation commands:
</p>

powershell -ExecutionPolicy Bypass -File "C:\users\labuser\desktop\remediation-FireFox-uninstall.ps1"

<details>
  <summary><strong>Firefox Remediation</strong></summary>
  <p>
powershell -ExecutionPolicy Bypass -File "remediation-FireFox-uninstall.ps1"
# Define the path to the uninstall helper
    
$uninstallHelperPath = 'C:\Program Files\Mozilla Firefox\uninstall\helper.exe'

Check if the uninstall helper exists

if (Test-Path $uninstallHelperPath) {
    #If the file exists, execute it silently
    Invoke-Expression "& `"$uninstallHelperPath`" /S"
    Write-Host "Firefox uninstall command executed."
} else {
    Write-Host "Firefox uninstall helper does not exist at the specified path."
}
</p></details>


powershell -ExecutionPolicy Bypass -File "remediation-SMBv1.ps1"

<details>
  <summary><strong>SMB 1.0 Remediation</strong></summary>
  <p>
# Disable SMBv1 - CIFS File Sharing Support
Write-Output "Disabling SMBv1 Protocol..."
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Disable the SMBv1 Client
$clientKeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters"
$clientDriverPath = "HKLM:\SYSTEM\CurrentControlSet\Services\mrxsmb10"
Write-Output "Disabling SMBv1 Client..."
if (Test-Path $clientKeyPath) {
    Set-ItemProperty -Path $clientKeyPath -Name "AllowInsecureGuestAuth" -Value 0
}
if (Test-Path $clientDriverPath) {
    Set-ItemProperty -Path $clientDriverPath -Name "Start" -Value 4
} else {
    Write-Output "SMBv1 Client driver registry path does not exist. It may not be necessary or supported on this system."
}

# Disable the SMBv1 Server
$serverKeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters"
Write-Output "Disabling SMBv1 Server..."
if (Test-Path $serverKeyPath) {
    Set-ItemProperty -Path $serverKeyPath -Name "SMB1" -Value 0
} else {
    Write-Output "SMBv1 Server registry path does not exist. Check if SMBv1 is supported on this system."
}

Write-Output "SMBv1 has been disabled on your system. Please review the output for any potential issues."
</p></details>


powershell -ExecutionPolicy Bypass -File "toggle-win11-protocols.ps1"


<details>
  <summary><strong>Toggle Protocols</strong></summary>
  <p>
# Variable to determine if we want to make the computer secure or insecure
$makeSecure = $true

# Check if the script is run as Administrator
function Check-Admin {
    $identity = [System.Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = New-Object System.Security.Principal.WindowsPrincipal($identity)
    $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
}

# Main script
if (-not (Check-Admin)) {
    Write-Error "Access Denied. Please run with Administrator privileges."
    exit 1
}

# SSL 2.0 settings
$serverPathSSL2 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server"
$clientPathSSL2 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "SSL 2.0 has been disabled."
} else {
    New-Item -Path $serverPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "SSL 2.0 has been enabled."
}

# SSL 3.0 settings
$serverPathSSL3 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server"
$clientPathSSL3 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "SSL 3.0 has been disabled."
} else {
    New-Item -Path $serverPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "SSL 3.0 has been enabled."
}

# TLS 1.0 settings
$serverPathTLS10 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server"
$clientPathTLS10 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.0 has been disabled."
} else {
    New-Item -Path $serverPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.0 has been enabled."
}

# TLS 1.1 settings
$serverPathTLS11 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server"
$clientPathTLS11 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Client"

if ($makeSecure) {
    New-Item -Path $serverPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.1 has been disabled."
} else {
    New-Item -Path $serverPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.1 has been enabled."
}

# TLS 1.2 settings
$serverPathTLS12 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server"
$clientPathTLS12 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client"

if ($makeSecure) {
    New-Item -Path $serverPathTLS12 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS12 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS12 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS12 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS12 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS12 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.2 has been enabled."
} else {
    New-Item -Path $serverPathTLS12 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS12 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS12 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    New-Item -Path $clientPathTLS12 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS12 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS12 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null

    Write-Host "TLS 1.2 has been disabled."
}

Write-Host "Please reboot for settings to take effect."
</p></details>


<p>
Optionally, I executed all remediation steps using a batch file.
</p>

<img width="919" height="213" alt="image" src="https://github.com/user-attachments/assets/cc42fd8f-1d2e-490b-b095-0c992b673745" />


<hr />

<h2>🔁 Step 9: Restart and Rescan</h2>

<p>
After remediation, I restarted the VM and ran a few final authenticated scans.
</p>

<ul>
  <li>Verified vulnerabilities were remediated</li>
  <li>Observed reduced findings and improved security posture</li>
</ul>

<img width="1851" height="483" alt="image" src="https://github.com/user-attachments/assets/d16883ab-2a20-4332-81ae-b3fff5b141db" />


<hr />

<h2>📊 Step 10: Export & Compare Results</h2>

<p>
I exported scan results and compared pre- and post-remediation findings.
</p>

<ul>
  <li>Validated remediation effectiveness</li>
  <li>Confirmed risk reduction</li>
</ul>

<img width="1851" height="481" alt="image" src="https://github.com/user-attachments/assets/f2f078be-f045-4701-88e2-33e2f28e23fa" />

<img width="1851" height="503" alt="image" src="https://github.com/user-attachments/assets/ca43a2fb-909b-4ef3-a100-3739195cdab3" />

<img width="1851" height="501" alt="image" src="https://github.com/user-attachments/assets/370b3688-9699-4229-9eac-8eb747eedbec" />


<hr />

<h2>🧹 Step 11: Cleanup</h2>

<p>
After completing validation, I removed all lab resources.
</p>

<ul>
  <li>Deleted the Tenable scan</li>
  <li>Deleted the Windows 11 virtual machine</li>
</ul>

<p>
This ensures cost control and prevents unnecessary exposure.
</p>

<hr />

<h2>🎯 Skills Demonstrated</h2>

<ul>
  <li>Authenticated vulnerability scanning</li>
  <li>Windows 11 system hardening</li>
  <li>DISA STIG-aligned assessments</li>
  <li>PowerShell-based remediation</li>
  <li>Vulnerability lifecycle validation</li>
  <li>SOC & Vulnerability Management workflows</li>
</ul>
