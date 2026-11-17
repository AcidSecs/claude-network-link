# Network Link for Claude (vn-network-link)

Created by Soroosh Hedayat

Designed in Collaboration by: VectorNine Systems Inc. & Acid Secs Productions Inc.

Have a second PC at home and want Claude to work on it? This plugin makes a very detailed tick-box guide that links two Windows PCs on the same home network using SSH, a secure remote door. It uses keys instead of passwords and works on your local network only. After every step there is a check, so you know it worked before you go on.

## What it does
- Asks a few plain questions, then builds the guide with every command ready to copy.
- Runs only harmless check commands after you say a step is done.

## What you do yourself
Every admin and security step: installing the SSH server, the firewall setting and turning password logins off. Claude never types those and never edits the SSH server's settings or the firewall.

## What it needs
Two Windows 10 or 11 PCs on the same network, an administrator account on the second PC, and the Desktop Commander connector on the PC Claude is linked to. A cloud session cannot reach your home network.

## What it sends
Nothing leaves your network. Private keys and passphrases never go in chat or in files shared with Claude.

Switch it on or off with claude-setup-wizard.
