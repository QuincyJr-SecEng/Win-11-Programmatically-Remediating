<h1>🧪 Hands-On: Windows 11 Authenticated Vulnerability Scanning & Scripted Remediation</h1>

<p>
In this lab, I performed an <strong>authenticated vulnerability scan</strong> against a Windows 11 virtual machine using <strong>Tenable Vulnerability Management</strong>, intentionally introduced common Windows vulnerabilities, remediated them using <strong>PowerShell scripts</strong>, and validated risk reduction through rescanning.
</p>

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

<p><strong>📸 Screenshot:</strong> Windows 11 VM Overview</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

<hr />

<h2>🔥 Step 2: Disable Windows Firewall</h2>

<p>
To ensure full scan visibility, I disabled the Windows Firewall on the VM.
</p>

<ul>
  <li>Opened Windows Firewall management (<code>wf.msc</code>)</li>
  <li>Disabled all firewall profiles</li>
</ul>

<p><strong>📸 Screenshot:</strong> Windows Firewall Disabled</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<p><strong>📸 Screenshot:</strong> PowerShell Registry Modification</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<p><strong>📸 Screenshot:</strong> Authenticated Scan Configuration</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<p><strong>📸 Screenshot:</strong> Outdated Firefox Installed</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

<p><strong>📸 Screenshot:</strong> SMBv1 Enabled</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

<hr />

<h2>🔄 Step 6: Restart the Virtual Machine</h2>

<p>
I restarted the VM to ensure all configuration changes were applied.
</p>

<p><strong>📸 Screenshot:</strong> VM Restart</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<p><strong>📸 Screenshot:</strong> Vulnerabilities Detected</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<pre>
powershell -ExecutionPolicy Bypass -File "remediation-FireFox-uninstall.ps1"
powershell -ExecutionPolicy Bypass -File "remediation-SMBv1.ps1"
powershell -ExecutionPolicy Bypass -File "toggle-win11-protocols.ps1"
</pre>

<p>
Optionally, I executed all remediation steps using a batch file.
</p>

<p><strong>📸 Screenshot:</strong> PowerShell Remediation Execution</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

<hr />

<h2>🔁 Step 9: Restart and Rescan</h2>

<p>
After remediation, I restarted the VM and ran a final authenticated scan.
</p>

<ul>
  <li>Verified vulnerabilities were remediated</li>
  <li>Observed reduced findings and improved security posture</li>
</ul>

<p><strong>📸 Screenshot:</strong> Post-Remediation Scan Results</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

<hr />

<h2>📊 Step 10: Export & Compare Results</h2>

<p>
I exported scan results and compared pre- and post-remediation findings.
</p>

<ul>
  <li>Validated remediation effectiveness</li>
  <li>Confirmed risk reduction</li>
</ul>

<p><strong>📸 Screenshot:</strong> Scan Comparison</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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

<p><strong>📸 Screenshot:</strong> Resource Cleanup</p>
<p><em>[INSERT SCREENSHOT HERE]</em></p>

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
