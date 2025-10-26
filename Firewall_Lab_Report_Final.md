# Windows Firewall Configuration Lab Report
## Cybersecurity Internship Task

**Date:** October 26, 2025  
**System:** Windows 11 Home Single Language (Build 26200)  
**Task:** Configure and test Windows Firewall rules  
**Prepared By:** Gautham

---

## Lab Objective

The goal of this lab was to learn how to configure Windows Firewall by creating rules to block specific network traffic, testing those rules, and then removing them to restore the original system state. I also learned how firewalls filter network traffic and why they are important for security.

---

## Lab Environment

**Operating System:** Windows 11 Home  
**Firewall:** Windows Defender Firewall with Advanced Security  
**Network:** Local machine (localhost)  
**Tools Used:** Windows Firewall GUI (wf.msc), PowerShell

---

## Step 1: Open Windows Firewall

I opened Windows Firewall with Advanced Security by:
1. Pressing `Windows + R`
2. Typing `wf.msc`
3. Pressing Enter

This opened the firewall management console where I could see all the existing rules.

---

## Step 2: Check Current Firewall Status

Before making any changes, I checked the firewall status using PowerShell:

```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled
```

**Results:**
```
Name    Enabled
----    -------
Domain     True
Private    True
Public     True
```

**Finding:** Windows Firewall is enabled on all three network profiles (Domain, Private, and Public). This is good security practice because it protects the computer regardless of which network it's connected to.

I also checked existing firewall rules:

```powershell
Get-NetFirewallRule | Where-Object {$_.Direction -eq 'Inbound'} | Select-Object -First 10 DisplayName, Enabled, Action
```

**Finding:** The system already has hundreds of pre-configured firewall rules protecting Windows services and applications.

---

## Step 3: Create Block Rule for Telnet (Port 23)

I used the Windows Firewall GUI to create a new inbound rule to block Telnet traffic on port 23.

### Rule Configuration Steps:

#### Step 3.1: Select Rule Type

![Rule Type Selection](firewall_screenshots/step1_rule_type.png)

I selected **"Port"** because I wanted to block traffic based on a specific port number (port 23 for Telnet).

**Options available:**
- Program: Block/allow specific applications
- **Port: Block/allow specific network ports** ← I selected this
- Predefined: Use Windows built-in rules
- Custom: Create advanced custom rules

---

#### Step 3.2: Specify Protocol and Port

![Protocol and Port Configuration](firewall_screenshots/step2_protocol_port.png)

Configuration:
- **Protocol:** TCP (Telnet uses TCP protocol)
- **Port:** 23 (standard Telnet port)

I entered "23" in the "Specific local ports" field because this is the port that Telnet services listen on.

---

#### Step 3.3: Select Network Profiles

![Profile Selection](firewall_screenshots/step3_profiles.png)

I checked all three network profiles:
- ✅ Domain
- ✅ Private  
- ✅ Public

This ensures the rule applies regardless of which type of network I'm connected to, providing maximum protection.

---

#### Step 3.4: Name the Rule

![Rule Name and Description](firewall_screenshots/step4_name.png)

I gave the rule a clear name and description:
- **Name:** BLOCK TELNET PORT 23
- **Description:** LAB TEST - BLOCKS INBOUND TELNET

Using descriptive names helps identify test rules later when cleaning up.

---

## Step 4: Verify the Rule Was Created

After clicking "Finish," I verified the rule appeared in the firewall rules list.

The rule shows:
- **Name:** BLOCK TELNET PORT 23
- **Group:** (None)
- **Profile:** All
- **Enabled:** Yes
- **Action:** Block
- **Direction:** Inbound

**Screenshot:** The rule is now visible in the Inbound Rules list in Windows Firewall

---

## Step 5: Test the Block Rule

To verify the firewall rule is working, I tested connecting to port 23 using PowerShell:

```powershell
Test-NetConnection -ComputerName localhost -Port 23
```

### Test Results:

```
ComputerName     : localhost
RemoteAddress    : ::1
RemotePort       : 23
InterfaceAlias   : Loopback Pseudo-Interface 1
SourceAddress    : ::1
PingSucceeded    : True
TcpTestSucceeded : False
```

**Analysis:**
- `PingSucceeded: True` - The computer is reachable
- `TcpTestSucceeded: False` - **Connection to port 23 FAILED**

This proves the firewall rule is working correctly and blocking TCP connections to port 23.

### Why did the test fail?

The connection failed because:
1. My firewall rule blocks all inbound TCP traffic to port 23
2. When Test-NetConnection tried to connect, the firewall dropped the packets
3. No connection could be established

This is exactly what we want - the firewall is protecting the system from Telnet connections.

---

## Step 6: Check Listening Ports

I also checked which ports are currently listening on my system:

```powershell
Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, State | Sort-Object LocalPort
```

**Results (sample):**
```
LocalAddress  LocalPort  State
------------  ---------  -----
0.0.0.0             135  Listen
0.0.0.0             445  Listen
127.0.0.1          7768  Listen
```

**Finding:** Port 23 (Telnet) is NOT in the list, which means no Telnet service is running on my computer. Even though nothing is listening on port 23, the firewall rule still blocks any connection attempts, adding an extra layer of security.

---

## Step 7: Remove the Test Rule

After completing the lab, I removed the test rule to restore my system to its original state.

### Removal Steps (GUI Method):

1. Opened Windows Firewall (wf.msc)
2. Clicked "Inbound Rules" in the left panel
3. Found "BLOCK TELNET PORT 23" in the rules list
4. Right-clicked on the rule
5. Selected "Delete"
6. Clicked "Yes" to confirm deletion

### Verification:

I verified the rule was removed by searching for it:

```powershell
Get-NetFirewallRule -DisplayName "BLOCK TELNET PORT 23"
```

**Result:** No rule found - successfully removed!

---

## Summary of Commands Used

### Checking Firewall Status:
```powershell
# Check if firewall is enabled on all profiles
Get-NetFirewallProfile | Select-Object Name, Enabled

# List current inbound rules
Get-NetFirewallRule | Where-Object {$_.Direction -eq 'Inbound'}

# Count total firewall rules
Get-NetFirewallRule | Measure-Object
```

### Testing Network Connections:
```powershell
# Test connection to specific port
Test-NetConnection -ComputerName localhost -Port 23

# Check which ports are listening
Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, State
```

### Verifying and Removing Rules:
```powershell
# Check if specific rule exists
Get-NetFirewallRule -DisplayName "BLOCK TELNET PORT 23"

# Remove rule (requires admin PowerShell)
Remove-NetFirewallRule -DisplayName "BLOCK TELNET PORT 23"
```

---

## How Windows Firewall Filters Traffic

### What is a Firewall?

A firewall is like a security guard at the entrance of a building. It checks every person (network packet) trying to enter or leave and decides whether to allow them based on a set of rules.

### Traffic Filtering Process

When a network packet arrives at my computer, here's what happens:

1. **Packet arrives** at the network card
2. **Windows Firewall intercepts** the packet before it reaches any application
3. **Firewall inspects** the packet header information:
   - Where is it coming from? (Source IP address)
   - Where is it going? (Destination IP address)
   - What port is it targeting? (Port number)
   - What protocol? (TCP, UDP, ICMP)
   - Is it inbound or outbound?
4. **Firewall checks rules** from top to bottom, looking for a match
5. **First matching rule is applied**
6. **Action is taken:**
   - **Allow:** Packet is delivered to the application
   - **Block:** Packet is dropped silently

### Example: My Telnet Block Rule

When I created the rule to block port 23, here's what happened:

**Rule Configuration:**
- Direction: Inbound (traffic coming INTO my computer)
- Port: 23 (Telnet)
- Protocol: TCP
- Action: Block
- Profile: All networks

**How it works:**

1. Someone (or Test-NetConnection) tries to connect to my computer on port 23
2. TCP packet arrives with destination port 23
3. Windows Firewall intercepts it
4. Firewall checks: "Is this inbound TCP traffic to port 23?"
5. Firewall finds my rule: "BLOCK TELNET PORT 23"
6. Packet is DROPPED (thrown away)
7. Connection fails
8. My computer is protected from Telnet attacks

### Why Block Telnet?

Telnet is an old remote access protocol from the 1970s that has serious security problems:

**Security Issues:**
- Sends passwords in plain text (unencrypted)
- Anyone monitoring network traffic can see usernames and passwords
- No encryption for any data transmitted
- Easy for attackers to steal credentials

**Modern Alternative:**
SSH (Secure Shell) replaces Telnet because:
- All traffic is encrypted
- Passwords and data are protected
- Uses port 22 instead of port 23
- Much more secure

This is why I practiced blocking Telnet - in real systems, Telnet should always be disabled or blocked.

### Firewall Filtering Criteria

Windows Firewall can filter traffic based on multiple criteria:

1. **Direction:**
   - **Inbound:** Traffic coming INTO your computer from the network
   - **Outbound:** Traffic going OUT from your computer to the network

2. **Protocol:**
   - **TCP:** Reliable, connection-based (used for web, email, file transfer)
   - **UDP:** Fast, connectionless (used for video streaming, DNS, gaming)
   - **ICMP:** Diagnostic protocol (used for ping)

3. **Port Numbers:**
   - Well-known ports: 0-1023 (standardized services)
   - Registered ports: 1024-49151 (vendor applications)
   - Dynamic ports: 49152-65535 (temporary connections)

4. **IP Addresses:**
   - Source IP: Where the traffic is coming from
   - Destination IP: Where the traffic is going
   - Can allow/block specific IPs or ranges

5. **Application/Program:**
   - Can create rules for specific .exe files
   - Example: Allow Chrome.exe but block other browsers

6. **Network Profile:**
   - Domain: Corporate networks
   - Private: Home/work trusted networks
   - Public: Coffee shops, airports (most restrictive)

### Rule Processing Order

Windows Firewall processes rules in a specific order:

1. **Block rules are checked first**
2. **Allow rules are checked second**
3. **Default action** if no rule matches (usually block inbound, allow outbound)

**Example:**
If you have two rules:
- Rule 1: BLOCK port 80
- Rule 2: ALLOW port 80

The BLOCK rule wins because block rules are evaluated before allow rules.

### Stateful Firewall

Windows Firewall is "stateful," which means it remembers connection states:

**Example:**
- You open a web browser and visit google.com (outbound connection to port 443)
- Web server sends data back (inbound traffic from port 443)
- Firewall allows the response because YOU started the connection
- But if a stranger tries to connect to YOUR port 443, it's blocked (they didn't start from inside)

This is smart because:
- You don't need to create allow rules for responses to your own requests
- Only traffic YOU initiated can come back in
- Unsolicited inbound connections are blocked

---

## Common Ports Reference

Here are common ports and whether they should be blocked:

| Port | Service | Protocol | Should Block? | Why? |
|------|---------|----------|---------------|------|
| 21 | FTP | TCP | Yes | Insecure file transfer, use SFTP instead |
| 22 | SSH | TCP | Allow if needed | Secure remote access |
| 23 | Telnet | TCP | Yes | Sends passwords in plain text |
| 25 | SMTP | TCP | Allow only if mail server | Email sending |
| 53 | DNS | UDP | Allow | Name resolution (needed) |
| 80 | HTTP | TCP | Allow if web server | Unencrypted web traffic |
| 443 | HTTPS | TCP | Allow if web server | Encrypted web traffic |
| 445 | SMB | TCP | Block from internet | Windows file sharing |
| 3389 | RDP | TCP | Block or restrict IPs | Remote Desktop - often attacked |
| 3306 | MySQL | TCP | Block from internet | Database should be internal only |
| 8080 | HTTP Alt | TCP | Block unless needed | Alternative web port |

---

## Security Best Practices

From this lab, I learned these firewall best practices:

### 1. Default Deny Approach
- Block all inbound traffic by default
- Only allow specific services that are needed
- This is called a "whitelist" approach
- Much safer than allowing everything and blocking bad stuff

### 2. Least Privilege Principle
- Only open ports that are absolutely necessary
- Close ports immediately when no longer needed
- Don't leave unnecessary services running

### 3. Use Specific Rules
- Don't create broad "allow all" rules
- Specify exact ports, protocols, and IP addresses when possible
- Example: Instead of "allow all HTTP," specify "allow HTTP from 192.168.1.0/24 only"

### 4. Regular Rule Review
- Periodically check firewall rules
- Remove old or unused rules
- Update rules when services change

### 5. Different Rules for Different Networks
- Use strictest rules on Public networks (coffee shops, airports)
- Can be slightly more relaxed on Private networks (home)
- Domain networks follow corporate policies

### 6. Enable Logging
- Turn on firewall logging for blocked connections
- Review logs regularly for suspicious activity
- Repeated blocks from same IP might indicate an attack

### 7. Defense in Depth
- Firewall is one layer of security
- Also use: antivirus, keep software updated, strong passwords, MFA
- Don't rely on firewall alone

---

## Lessons Learned

From completing this firewall configuration lab, I learned:

1. **Windows Firewall is powerful and complex**
   - Hundreds of built-in rules protecting Windows services
   - Can control traffic in very specific ways

2. **Rules must be specific to be effective**
   - Need to specify direction, port, protocol, profile
   - Each rule targets specific traffic patterns

3. **Testing is critical**
   - Always verify rules work as expected
   - Test-NetConnection is useful for verification

4. **Telnet is insecure and should be blocked**
   - Plain text passwords are dangerous
   - Modern systems use SSH instead

5. **Block rules take priority over allow rules**
   - Firewall checks block rules first
   - This makes sense for security - safer to block by default

6. **Documentation is important**
   - Clear rule names help identify purpose later
   - Descriptions prevent accidental deletion of important rules

7. **Cleanup is part of the process**
   - Always remove test rules
   - Keep firewall rules organized and minimal

---

## Lab Results

| Task | Status | Method Used |
|------|--------|-------------|
| Open Windows Firewall | ✅ Completed | GUI (wf.msc) |
| Check firewall status | ✅ Completed | PowerShell |
| List current rules | ✅ Completed | PowerShell & GUI |
| Create block rule for port 23 | ✅ Completed | GUI wizard |
| Verify rule creation | ✅ Completed | GUI & PowerShell |
| Test block rule | ✅ Completed | Test-NetConnection |
| Document all steps | ✅ Completed | Screenshots & commands |
| Remove test rule | ✅ Completed | GUI |
| Verify removal | ✅ Completed | PowerShell |

---

## Troubleshooting Notes

### Issue: Need Administrator Access for PowerShell Commands

**Problem:** Some PowerShell commands require administrator privileges to create or delete firewall rules.

**Solution:** 
- Use Windows Firewall GUI (wf.msc) instead - no admin PowerShell needed
- Or right-click PowerShell → "Run as administrator"

### Issue: Understanding Test Results

**Problem:** Initially wasn't sure if `TcpTestSucceeded: False` meant success or failure.

**Solution:** 
- False means connection was blocked - this is GOOD for a block rule
- It proves the firewall is working correctly

---

## Conclusion

I successfully completed the Windows Firewall configuration lab. I created an inbound rule to block Telnet traffic on port 23, tested it to verify it was working, and then removed it to restore the system to its original state.

The most important thing I learned is that firewalls are a critical first line of defense for computer security. They filter network traffic based on rules, blocking unwanted connections while allowing legitimate traffic. Understanding how to configure firewall rules is an essential skill for cybersecurity.

I now understand:
- How to create firewall rules using the Windows GUI
- How to test if firewall rules are working
- Why certain ports like Telnet should be blocked
- How firewalls filter traffic based on multiple criteria
- The importance of cleaning up test rules

This hands-on experience helped me understand firewall concepts much better than just reading about them.

---

## Appendix: Screenshots

### Screenshot 1: Rule Type Selection
![Step 1 - Selecting Port Rule Type](firewall_screenshots/step1_rule_type.png)

Shows the "New Inbound Rule Wizard" where I selected "Port" as the rule type to create a rule that controls TCP/UDP port connections.

---

### Screenshot 2: Protocol and Port Configuration  
![Step 2 - Configuring TCP Port 23](firewall_screenshots/step2_protocol_port.png)

Shows the protocol selection (TCP) and port specification (23). This tells the firewall which traffic to block.

---

### Screenshot 3: Network Profile Selection
![Step 3 - Selecting All Network Profiles](firewall_screenshots/step3_profiles.png)

Shows all three network profiles checked (Domain, Private, Public) to ensure the rule applies to all network types.

---

### Screenshot 4: Rule Name and Description
![Step 4 - Naming the Rule](firewall_screenshots/step4_name.png)

Shows the final step where I named the rule "BLOCK TELNET PORT 23" and added a description for easy identification.

---

## Additional Resources

**Microsoft Documentation:**
- Windows Defender Firewall: https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-firewall/
- PowerShell Firewall Cmdlets: https://docs.microsoft.com/en-us/powershell/module/netsecurity/

**Learning Resources:**
- SANS Firewall Guide: https://www.sans.org/reading-room/whitepapers/firewalls/
- Common Port Numbers: https://www.iana.org/assignments/service-names-port-numbers/

**Security Standards:**
- NIST Firewall Guide (SP 800-41): https://csrc.nist.gov/publications/detail/sp/800-41/rev-1/final
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks/

---

**Report Prepared By:** Gautham  
**Date:** October 26, 2025  
**Internship:** Cybersecurity Training Program  
**Task:** Firewall Configuration and Traffic Filtering Lab

---

End of Report
