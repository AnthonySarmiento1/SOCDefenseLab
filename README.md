<h1>Creating A SOC & Defending Attacks</h1>

<h2>Description</h2>

This Project consists of steps on how to:
 <ol type = "1">
  
<li>Build a SOC using LimaCharlie's EDR solution</li>
<li>Setup a Command-and-control (C2) framework</li>
<li>Using Sliver C2 framework to emulate threat actor behaviors</li>
<li>Writing Detection & Response rules to alert for and block attacks thrown by threat actors</li>
</ol>

<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>VMware Workstaton</b>
- <b><a href="https://limacharlie.io/">LimaCharlie’s SaaS</a></b>
- <b><a href="https://bishopfox.com/tools/sliver">Sliver C2 Framework</a></b>
- <b>Python3</b>
<h2>Environments Used </h2>

- <b>Windows 11</b> (Victim)
- <b>Ubuntu Linux</b> (Attacker)
- <b>Windows 10</b> (Security Operations Center)
<h2>Program walk-through:</h2>

<!--  ############################################################################################################## 
<a href=""> </a>
<img src=""/>
       ############################################################################################################### 
       -->

       
<h3>Part 1: Setting Up LimaCharlie</h3>


<p align="center">
<u>First Download <a href="https://www.vmware.com/content/vmware/vmware-published-sites/us/products/workstation-pro/workstation-pro-evaluation.html.html">VMWare</a> and install A <a href="https://ubuntu.com/download/server">Linux</a> and <a href="https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/">Windows 11</a> ISO:</u> 
 <br/>
<img src="https://i.imgur.com/jReyXB2.png" height="80%" width="80%" />
<br />
<br />
Install Sysmon and SwiftOnSecurity’s Sysmon config on your Windows Vm to generate telemetry for LimaCharlie using these administrative powershell commands:  <br/>
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
   
   Now Create an <a href="https://app.limacharlie.io/signup"> LimaCharlie account</a> on your Host machine, or another VM.
  
   <img src="https://imgur.com/RP9cHJC.png"/>
  
    <p>After Creating An Account Create an organization using the "Extended Detection & Response Standard" Template.</p>
    
   <img src="https://imgur.com/jNfEkET.png"/>
    
    Once the organization is created add a sensor to your Windows 11 VM using the "x86-64.exe" architecture.
  </p>
  
 <h3>Part 2: Sliver C2 Framework</h3>
 <h3>Part 3: Launching Attacks</h3>
 <h3>Part 4: Detecting and Blocking Attacks</h3>
