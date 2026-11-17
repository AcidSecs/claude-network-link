---
name: network-link
description: Guide the user through linking two Windows PCs on the same home network with SSH so Claude can work on the second PC. Use when the user says "connect my laptop", "link my other PC", "set up SSH", or "run something on my second computer". Windows, Desktop Commander.
---

# Network Link (SSH between two PCs on your own network)

**What this is, in plain words:** SSH is a secure remote door into another PC. A key replaces the password: like a house key that only your PC has, instead of a code someone could guess. This setup works on your home network only.

## Step 0: settings
Read `Documents\ClaudeSetup\settings.json`. If `modules.network_link` is not true, say "Network Link is off" and stop.

## Who does what (say this first)
- **The user does every admin and security step** (installing the SSH server, the firewall, turning password logins off). Claude never types those, and never edits the SSH server's settings or the firewall.
- **Claude** makes the tick-box guide, explains each step, and runs only the harmless CHECK commands after the user says a step is done.
- Nothing is opened to the internet. If a step would do that, stop and tell the user.

## Words
- **Host** = the PC you want to reach (the second PC).
- **Client** = the PC you connect from (the one Claude is linked to).
- Claude must be linked to the Client through Desktop Commander. A cloud session cannot reach your home network, so it cannot do this.

## Ask first (one at a time)
1. Host's Windows user name and the Host's name or address on the network.
2. Is that user an administrator on the Host? (decides which key file is used)
3. Key with a passphrase (safer; Claude needs the ssh-agent loaded once per login) or without (simpler; anyone who gets the key file gets in)? Recommend the passphrase.

## Build the guide
Make the tick-box guide from `guide/GUIDE_TEMPLATE.html.txt` (see "Guides" below), with these stages. Put each command in a copy chip. After EVERY step add a CHECK line with the exact thing the user should see.

**Stage 1: on the Host (run PowerShell as administrator; user does this)**
1. Install: `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`. CHECK: `Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'` shows Installed.
2. Start now and at every boot: `Set-Service -Name sshd -StartupType Automatic` then `Start-Service sshd`. CHECK: `Get-Service sshd` shows Running.
3. Firewall: Windows normally creates the rule `OpenSSH-Server-In-TCP`. Limit it to the home network: `Set-NetFirewallRule -Name OpenSSH-Server-In-TCP -RemoteAddress LocalSubnet`. CHECK: `Get-NetFirewallRule -Name OpenSSH-Server-In-TCP | Get-NetFirewallAddressFilter` shows LocalSubnet. (UNCONFIRMED on every Windows version: if the command errors, stop and report the message.)
4. Find the Host's address: `ipconfig`, note the IPv4 address. Suggest the user reserve that address for the Host in their router so it does not change (the router screen differs per brand; give general steps only).

**Stage 2: on the Client (user's own PC)**
5. Make a key: `ssh-keygen -t ed25519` (accept the default path; choose a passphrase). CHECK: the file `id_ed25519.pub` exists in the user's `.ssh` folder.
6. Copy the PUBLIC key (the `.pub` file, never the private one) to the Host. Standard user: append it to `.ssh\authorized_keys` on the Host. Administrator: append it to `C:\ProgramData\ssh\administrators_authorized_keys` and lock the file with `icacls.exe` so only Administrators and SYSTEM can read it (Microsoft's OpenSSH key management page gives the exact commands; show them with the real user and host filled in).
7. Load the key once per login: `Get-Service ssh-agent | Set-Service -StartupType Automatic`, `Start-Service ssh-agent`, `ssh-add` the key (administrator PowerShell for the service steps).

**Stage 3: test (Claude can run this CHECK)**
8. `ssh -o BatchMode=yes USER@HOST "echo link-ok"` must print `link-ok` with no password prompt. If it asks for a password or says permission denied, go to the troubleshooting list.

**Stage 4: turn passwords off (user does this, only after step 8 works)**
9. On the Host, in an administrator editor, open `C:\ProgramData\ssh\sshd_config` and set `PasswordAuthentication no`, then `Restart-Service sshd`. CHECK from the Client: `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no USER@HOST` is refused, and step 8 still works.

## Using the link afterwards
Run commands on the Host with Desktop Commander: `ssh -o BatchMode=yes USER@HOST "<command>"`. Lessons learned:
- If the Host's shell is cmd, chain commands with `&&`, not `;`.
- Programs that wait for typed input (for example an interactive chat program) hang over SSH; call their web or API interface instead.
- Copy files with `scp`.

## Troubleshooting (give the user the matching line)
- Connection refused or timed out: the service is not running, or the firewall rule is off or limited to the wrong network.
- Permission denied (publickey): the public key is in the wrong file (administrator vs standard user), or the key file's permissions are too open.
- Asks for a password: the key was not loaded; run `ssh-add` again.
- The address changed: reserve it in the router.

## Health check
On request: run the step 8 check; confirm `ssh` exists on the Client. If something fails, search Anthropic's and Microsoft's current docs, PROPOSE a fix, and ASK before changing anything.

## Safety
Private keys and passphrases never go in chat, guides or files shared with Claude. Never expose the Host to the internet. Never run admin or firewall commands for the user.

## Guides for steps the user must do
Copy `guide/GUIDE_TEMPLATE.html.txt` to a NEW file named like `My_Guide.html` in the user's Documents folder (the template is stored as .txt on purpose), change only the GUIDE object near the bottom, and follow the instructions at the top of the template. Make it extremely detailed: about five small steps for each real step, plain English, the reason in brackets, a copy chip for every command, and a CHECK for every step. Keep the chat reply short and point to the guide.
