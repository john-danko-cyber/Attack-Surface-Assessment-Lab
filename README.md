# Assess Attack Surface

## Summary
A CompTIA Security+ live lab (2.2.12, SY0-701 objective 2.3 Describe threat vectors and sources) in the simulated Structureality environment. Working from the provided virtual machines, I assessed three attack surfaces: a phishing email, an unlocked workstation, and the network border.

The email was titled DC Critical Alert and used urgency wording to push the reader to act. The headers showed a Return-Path of support@structurealty.com and a Received line from support.structurealty.com at 139.155.190.84, a typo of structureality.com. The techniques in use were typosquatting and urgency. The View alert details link opened a third-party notification page on structureality.dischord.com, which loaded scripts from support.structurealty.com on TCP port 3000. That extra script source is a cross-site scripting issue on a third-party site, a supply chain vector. In a lab exercise framed as access to a seized threat-actor server (or a red-team exercise), I observed how the browser had been hooked. The BeEF panel listed the session at 203.0.113.1, fingerprinted as Microsoft Edge 152 on Windows 10, with a BEEFHOOK cookie. A Fake Notification Bar was then pushed to that session, prompting the user to run an "applet: 'Java' from Microsoft Inc."

On the host side, the PC had been left unlocked, which is a lunchtime attack. UPDATOR.exe was on the DVD drive (D:), 69,262 KB in size, with no embedded or catalog digital signatures, and it was already running as a background process using 158.3 MB of memory. A Microsoft Defender quick scan checked 13,527 files in 53 seconds and found 0 threats, even while UPDATOR was running. Tamper protection was off. Process Monitor, filtered to Image path d:\updator.exe, showed repeated UDP Send and UDP Receive events from the PC to the domain controller, which is also the DNS server, in rounds at 11:50 and 11:53. That pattern is consistent with initiating DNS traffic and with periodic beaconing.

On the network border, the OPNsense WAN rules expose HTTP, HTTPS, SMTP, SSH, IPsec VPN, and IMAP. The rule "Publish intranet sites (unsecure)" allows TCP port 80 to 172.16.0.201 on the screened subnet. Before a denial-of-service incident, the firewall baseline was about 42% memory, 2% disk, a load average of 0.42, and a state table that fit on one page. During the incident, memory, CPU, and disk stayed near that baseline, incoming traffic spiked, and the state table grew to 2,525 entries, many of them TIME_WAIT connections from 139.155.190.8 to 172.16.0.201:80. The component most affected was the state table. The source address is in the same 139.155.190.x range that sent the phishing email.

## My Role
I completed this hands-on in CompTIA's live lab on November 24, 2026. I inspected the phishing message and headers in Thunderbird, reviewed the third-party notification page and its script sources in the browser developer tools, and observed how the threat actor hooked the browser from a panel the lab presented as a seized threat-actor server. On the unlocked PC I documented UPDATOR.exe, ran a Defender quick scan, and used Process Monitor to record its network activity. On the OPNsense border firewall I reviewed the WAN rules, recorded a baseline, and compared the dashboard and state table during the incident. I used an administrator account to elevate Process Monitor only because this was a lab. In a real investigation, unknown software like this should be analyzed in a sandbox, not on a live workstation. Structureality and this environment are part of the course lab, not a real workplace or my own network. The write-up is my own.

## Key Findings
- The phishing email combined typosquatting (structurealty.com) with urgency wording. The third-party notification page then loaded scripts from the threat domain, which hooked the browser.
- Signature-based antivirus missed UPDATOR.exe, an unsigned program running from removable media on an unlocked workstation and beaconing to the DNS server over UDP.
- A flood of connections from 139.155.190.8 filled the firewall state table (2,525 entries) while CPU, memory, and disk stayed near baseline. Plain HTTP to the screened subnet, and the lack of load balancing, left the public site exposed.

## Evidence

### 1. Phishing email: DC Critical Alert
In Thunderbird on ADMIN, Jaime received a message from Structureality Support with the subject DC Critical Alert. The body calls itself an urgent notification for all administrators and includes a View alert details button. The visible From address is already support@structurealty.com.

![Phishing email DC Critical Alert](screenshots/01-thunderbird-dc-critical-alert.png)

### 2. Message headers showing the typosquat
The source headers show Return-Path: support@structurealty.com and a Received line from support.structurealty.com at 139.155.190.84, before the message reached the internal mail gateway. structurealty.com is a typo of structureality.com. Combined with the urgency wording, the techniques are typosquatting and urgency.

![Email headers with typosquat domain](screenshots/02-email-headers-typosquat.png)

### 3. Third-party notification page
View alert details opened a third-party notification page at structureality.dischord.com. The page showed an alert about failed logon attempts to the DC over RDP.

![Third-party notification page](screenshots/03-dischord-notification-page.png)

### 4. Developer tools: scripts from the threat domain
In the Sources tab, the page had made a secondary connection to support.structurealty.com on TCP port 3000. That host is the threat domain, and the extra scripts are a third-party supply chain vulnerability.

![DevTools Sources showing the threat domain](screenshots/04-devtools-sources-threat-domain.png)

### 5. BeEF panel with a hooked browser
On a host the lab framed as a seized threat-actor server (or a red-team exercise), I opened the BeEF panel at http://support.structurealty.com:3000/ui/panel. Under Online Browsers, the hooked session was the notification page, listed as 203.0.113.1.

![BeEF panel showing the hooked browser](screenshots/05-beef-panel-hooked-browser.png)

### 6. Hooked browser details
The panel fingerprinted the session as Microsoft Edge 152 on Windows 10 (Win32, Blink). A BEEFHOOK cookie was present, and the page host was structureality.dischord.com.

![BeEF hooked browser details](screenshots/06-beef-browser-details.png)

### 7. Fake Notification Bar module
Under Social Engineering, the Fake Notification Bar module was selected. The notification text was: This website wants to run the following applet: 'Java' from Microsoft Inc.

![BeEF Fake Notification Bar module](screenshots/07-beef-fake-notification-bar.png)

### 8. Fake notification bar on ADMIN
Back on the ADMIN browser, the bar appeared at the top of the third-party notification page, prompting the user to accept the security popup.

![Fake notification popup on ADMIN](screenshots/08-admin-fake-notification-popup.png)

### 9. Email-vector demo complete
The email-vector steps are complete. The fake notification bar is still visible on the hooked session, and the Sources tree still shows support.structurealty.com:3000.

![Email vector demo complete](screenshots/09-email-demo-complete.png)

### 10. UPDATOR.exe on the DVD drive
On the unlocked PC, signed in as structureality\Angel, DVD Drive (D:) contained UPDATOR, an application last modified on 4/7/2026 and 69,262 KB in size.

![UPDATOR.exe on the DVD drive](screenshots/10-dvd-drive-updator.png)

### 11. No digital signatures
The Digital Signatures tab on UPDATOR Properties shows no embedded signatures and no catalog signatures.

![UPDATOR properties with no digital signatures](screenshots/11-updator-no-digital-signature.png)

### 12. UPDATOR running in Task Manager
UPDATOR was running as a background process, using 158.3 MB of memory, with no visible window.

![Task Manager showing UPDATOR running](screenshots/12-task-manager-updator-running.png)

### 13. Defender quick scan found no threats
A Microsoft Defender quick scan at 11:47 AM checked 13,527 files in 53 seconds and found 0 threats, even while UPDATOR was running. Tamper protection was off.

![Windows Security quick scan with no threats](screenshots/13-windows-security-quick-scan.png)

### 14. Process Monitor: UDP traffic to the DC
Filtered to Image path d:\updator.exe, Process Monitor recorded repeated UDP Send and UDP Receive events from the PC to DC.ad.structureality.com. The events came in rounds at 11:50 and 11:53, a beaconing pattern consistent with DNS traffic to the domain's DNS server.

![Process Monitor showing UPDATOR UDP DNS traffic](screenshots/14-procmon-updator-udp-dns.png)

### 15. OPNsense dashboard baseline
Before the incident, memory was about 42%, disk was at 2%, and the load average was 0.42, 0.36, 0.29. The firewall was otherwise idle.

![OPNsense dashboard baseline](screenshots/15-opnsense-dashboard-baseline.png)

### 16. WAN firewall rules
The top of the WAN rules list shows HTTP, HTTPS, SMTP, SSH, and IPsec VPN exposed, with IMAP rules further down. The rule that allows plain HTTP (port 80) to 172.16.0.201 on the screened subnet is titled Publish intranet sites (unsecure).

![OPNsense WAN firewall rules](screenshots/16-opnsense-wan-rules.png)

### 17. State table baseline
Firewall Diagnostics: States fit on a single page, with a normal mix of ESTABLISHED and TIME_WAIT connections from a few internal hosts.

![OPNsense state table baseline](screenshots/17-opnsense-states-baseline.png)

### 18. Dashboard during the incident
During the incident, memory was about 39%, disk was still 2%, and the load average was 0.35, 0.37, 0.32, so CPU, RAM, and disk were not the main impact. The Traffic In graph spiked to about 2.5 Mb.

![OPNsense dashboard during the incident](screenshots/18-opnsense-dashboard-during-incident.png)

### 19. State table during the incident
The state table grew to 2,525 entries across many pages. New entries include TIME_WAIT connections from 139.155.190.8 to 172.16.0.201:80, hitting the Publish intranet sites rule. The component most affected is the state table.

![OPNsense state table during the incident](screenshots/19-opnsense-states-during-incident.png)

## Defensive Takeaways
- Check email headers for lookalike domains, and treat unexpected urgency as a reason to slow down, not click.
- Third-party pages that pull scripts from other domains expand the attack surface.
- Lock the screen when stepping away, and set an automatic lock. Do not run unsigned software from removable media.
- Signature-based antivirus only catches known threats. Behavior-based monitoring can still show command-and-control patterns such as DNS beaconing.
- Analyze unknown executables in a sandbox, not on a live workstation outside a controlled exercise.
- Close plain HTTP (port 80) when the site can be served on HTTPS, and plan for load balancing, failover, and rate limiting so a state-table flood cannot take the service down.
- A baseline of firewall resource and state counts is what makes an incident visible.

## Disclaimer
This is a CompTIA Security+ course live lab in a simulated environment. Structureality is a fictional company used for training. The screenshots and write-up document my own work in that lab; they do not represent a real production network or employer.
