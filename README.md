# sshproxy
This tool was designed with covert pentesting in mind. It it designed to mask traffic with SSH using binded ports and proxychains.

# Use Case
This focuses on pivoting undected (or at least, if you are detected, not much info can be derived from the traffic).

There are 3 boxes in the network, all \*nix machines. One is your attack box, the second is a box you have pwned and the third is a box you want to pwn, but you are at a delima. How do you route your traffic from the attack box, through the pwned box, to the target box?

You run this script and then prepend all following network based recon commands with "proxychains ". Take note that the script makes a proxychain.conf file, so if you move out of the working directory, bring it with you. This leverages the fact that proxychains checks the working directory for a configuration file first.

## What this is actually doing...
1. Creates a local proxychains file based on the existing one (/etc/proxychains.conf), so any custom settings are copied over and the main one is not accidentally messed up. In this file, it specifies the IP/domain and port pair that is specified during execution as a proxy
2. It connects to the IP provided with the user and password provided via SSH
3. Immediately after getting a terminal, it executes a command to create a binded port run by SSH

## In-Depth Explanation of Traffic
For this example, we are assuming we only scanned one port on the target and routed it through another box (Pwned Box).
- Port - the port you specify in the sshproxy.sh tool
- e - ephemeral port
- 22 - port 22

1. Attack:e --> Pwned:Port TCP, SYN 
2. Pwned:Port --> Attack:e TCP, SYN/ACK
3. Attack:e --> Pwned:Port TCP, ACK Three way handshake using the ssh binded port
4. Attack:e --> Pwned:Port TCP, PSH/ACK Forwards data for port scan
5. Pwned:Port --> Attack:e TCP, ACK
6. Pwned:22 --> Attack:e SSH, Response from target
7. Attack:e --> Pwned:22 SSH, ACK
8. Pwned:Port --> Attack:e FIN/ACK
9. Attack:e --> Pwned:Port FIN/ACK
10. Pwned:Port --> Attack:e ACK

### Blue Team Notes
- All SSH responses are on the same data stream but the TCP streams are different per port scanned
- In testing, all SSH responses had data 88 bytes long, total packet length was 156 bytes (may not be indicative of all topologies)
- A TCP RST exposes the SSH client being used, RSTs are marked in red in Wireshark by default

### Red Team Notes
- If the account connection is severed, you have to re-initiate the connection to turn the proxy back on
- Be aware that if there are network connection issues, it will likely expose some data in clear text
- This does not prevent blue team from identifying the attack box, only the target

#### Tools Verified to Work
- nmap
- nikto
- dirb
- curl
- nc (both bind and reverse shells)
