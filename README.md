# sshproxy
This tool was designed with covert pentesting in mind. It it designed to mask traffic with SSH using binded ports and proxychains.

## Use Case
This focuses on pivoting undected (or at least, if you are detected, not much info can be derived from the traffic). Because of the complexity of this topic, it will be explained with an example.

## \*Nix to \*Nix Scenario
There are 3 boxes in the network, all \*nix machines. One is your attack box, the second is a box you have pwned and the third is a box you want to pwn, but you are at a delima. How do you route your traffic from the attack box, through the pwned box, to the target box?

### What this is actually doing...
1. Creates a local proxychains file based on the existing one, so any custom settings are copied over and the main one is not accidentally messed up. In this file, it specifies the IP/domain and port pair that is specified during execution as a proxy
2. It connects to the IP provided with the user and password provided via SSH
3. Immediately after getting a terminal, it executes a command to create a binded port run by SSH

### Diagrams Are Nice
- Attack Box --> [proxychains] --> [ssh bind on pwnedbox] --> targetbox
- Attack Box <-- [proxychains] <-- [ssh bind on pwnedbox] <-- targetbox

