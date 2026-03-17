I Built a SOC Detection Lab and Wrote Custom Rules That Caught Real Attacks


A hands-on walkthrough of LimaCharlie EDR, Atomic Red Team, and MITRE ATT&CK detection from start to finish

I built a hands-on SOC detection lab using LimaCharlie EDR. I deployed sensors, simulated MITRE ATT&CK reconnaissance techniques, and created custom detection rules that successfully caught suspicious commands in real-time. This gave me practical experience with the detect-analyze-alert workflow that’s essential for SOC operations.

Action: Created a LimaCharlie account and accessed the security operations dashboard.

What is LimaCharlie? LimaCharlie is a SecOps Cloud Platform designed to enhance security operations for modern enterprises. It provides endpoint detection and response (EDR), threat detection, and incident response capabilities in a unified cloud-based platform.


<img width="975" height="534" alt="image" src="https://github.com/user-attachments/assets/40aa49a4-9701-4b8a-bf8d-d499de4581c6" />



Step 1. Create an account with LimaCharlie. LimaCharlie is a SecOps Cloud Platform designed to enhance security operations for modern enterprises.

After you make an account, you will see your dashboard.

<img width="931" height="356" alt="image" src="https://github.com/user-attachments/assets/cf9c28d1-6382-4ec9-9de1-4cb33cf22673" />

Step 2: Configure Installation Keys

Navigate to the Installation Keys section in the LimaCharlie dashboard
Delete the default installation keys
Create your own custom installation key (I named mine “test”)

<img width="720" height="808" alt="image" src="https://github.com/user-attachments/assets/7e748664-c1a8-4f0f-ac9c-60a2d604af98" />


Step 3: Download the Windows Sensor

Go to the Installations section in LimaCharlie
Download the Windows sensor agent to your system

<img width="720" height="773" alt="image" src="https://github.com/user-attachments/assets/07fd9fb2-fdca-4baa-918d-25114a3d3bf8" />


Step 4: Install the Sensor via PowerShell

Open PowerShell as Administrator (right-click PowerShell → Run as administrator)
Navigate to your Downloads folder:
Mine: PS C:\WINDOWS\system32> cd C:\Users\Owner\Downloads
PS C:\Users\Owner\Downloads> dir

<img width="720" height="88" alt="image" src="https://github.com/user-attachments/assets/65c0ecbf-77ec-4452-8419-a3c6f439d613" />


Run the installation command: PS C:\Users\Owner\Downloads> .\hcp_win_x64_release_4.33.23.exe -i [YOUR_SENSOR_KEY]

<img width="720" height="778" alt="image" src="https://github.com/user-attachments/assets/5c8861b3-8d45-42ba-8dfd-b815e6cbf911" />


Step 5: Verify Successful Installation

After successful installation, you should see your endpoint registered in the LimaCharlie dashboard.


<img width="720" height="230" alt="image" src="https://github.com/user-attachments/assets/5c85b54b-f92e-4b2d-a818-1bf9e991c72e" />

Step 6. My endpoint details:

Hostname: DESKTOP-3RG77DU.hsd1.mn.comcast.net
Status: Active and reporting telemetry

<img width="720" height="764" alt="image" src="https://github.com/user-attachments/assets/57533ab2-c933-4813-999a-53ff4213fc92" />



Phase 2: Installing Atomic Red Team

What is Atomic Red Team? Attack simulation framework that generates malicious activity for detection testing.

<img width="720" height="336" alt="image" src="https://github.com/user-attachments/assets/26f58497-0d2c-4eb2-b6c9-b5ef7436df01" />


1. Issue I ran into: Windows Defender Blocked Installation

Solution: Add exclusion in Windows Defender

Steps:

1. Open Windows Security
2. Click “Virus & threat protection."
3. Click “Manage settings."
4. Scroll to "Exclusions."
5. Click “Add or remove exclusions” → “Add an exclusion” → "Folder."
6. Select your Atomic Red Team installation folder
Why? Windows Defender flags attack simulation tools as threats. Exclusions allow testing in controlled lab environments.


<img width="720" height="547" alt="image" src="https://github.com/user-attachments/assets/5da4b690-9835-452a-98d0-248815493e14" />


<img width="617" height="478" alt="image" src="https://github.com/user-attachments/assets/fe015843-1d04-437f-896a-6a200d006bcd" />


2. Now let’s RUN SOME ATTACKS and watch LimaCharlie detect them!

Import-Module “C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1” -Force

What Each Part Does:

Import-Module

PowerShell command that loads a module (collection of functions/tools) into your current session
Makes the Atomic Red Team commands available to use
“C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1”

File path to the Atomic Red Team module manifest
.psd1 = PowerShell Data file that describes the module
Contains all the attack simulation functions
-Force

Forces the module to load even if it’s already loaded
Overwrites any existing version
Ensures you’re using the latest version
What This Accomplishes:

After running this command, you can now execute attack simulations like

Invoke-AtomicTest — Run specific MITRE ATT&CK techniques
Generate malicious activity that LimaCharlie will detect
Invoke-AtomicTest T1003.001 -TestNumbers 1

What it does:

T1003.001 = MITRE ATT&CK technique for credential dumping (stealing passwords from memory)
-TestNumbers 1 = Runs test #1 for this technique
Simulates how attackers steal credentials from Windows systems
3. Alternative Attack Simulation

Issue: The Atomic Red Team test didn’t work for me.
Solution: Used built-in Windows commands to generate suspicious activity.
Problem: Invoke-AtomicTest T1003.001 -TestNumbers 1 didn’t work for me.
Solution: Used built-in Windows commands instead to create suspicious activity.
Commands I ran powershell, net user HackerTestUser Pass123! /add, whoami /priv and net localgroup administrators
What these do:
Create suspicious user account (persistence), Check my privileges (recon) and List admins (privilege escalation attempt).

<img width="720" height="225" alt="image" src="https://github.com/user-attachments/assets/bfaee24f-9b05-4cb1-b3b8-6a71456c7498" />

4. After running the whoami command, I navigated to the LimaCharlie Dashboard → Detections section to verify whether the rule triggered.
The detection appeared immediately, confirming that the rule successfully identified the suspicious activity and generated an alert as expected.

<img width="720" height="400" alt="image" src="https://github.com/user-attachments/assets/aa430b06-083a-4ecd-847b-5044015d1efc" />

<img width="720" height="180" alt="image" src="https://github.com/user-attachments/assets/b9ea0766-6177-4b68-b68b-e67887a63480" />

Phase 3: Automating Detection with Custom Rules


<img width="720" height="278" alt="image" src="https://github.com/user-attachments/assets/ca398030-80b3-4e2c-8307-72debf6a9239" />

What Each Part of the Rule Does

Event: NEW_PROCESS
• Monitors every new program that starts on the system

Operator: and
• All conditions below must be true for the rule to trigger

File Path Check (whoami.exe)
• Case insensitive (matches whoami.exe, WHOAMI.EXE, etc.)
• Requires an exact match (not just a partial name)
• Watches specifically for:
C:\Windows\System32\whoami.exe

Parent Process Check (PowerShell)
• Examines which process launched whoami.exe
• Triggers if the parent process path contains powershell.exe
• Identifies PowerShell as the execution source

Action: report
• Generates a security alert

Alert Name:
“Suspicious reconnaissance detected — whoami from PowerShell”

In Simple Terms

IF whoami.exe runs
AND it was launched by PowerShell
THEN generate a security alert

Why This Is Suspicious

Attackers commonly use PowerShell to perform reconnaissance after gaining access.
Running whoami helps them identify the current user, privileges, and group memberships.

This behavior is not inherently malicious, but it is high-risk in attack chains and worth alerting on.

Testing the Rule

Command executed in PowerShell:

Result:
✅ Alert immediately appeared in the LimaCharlie Detections Dashboard


<img width="720" height="261" alt="image" src="https://github.com/user-attachments/assets/d3cd6c3b-9249-409a-bd01-9e4dce42c1d7" />

























