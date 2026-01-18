# Jellyfin Security Hardening
**Securing my Jellyfin server using Caddy reverse proxy and Fail2Ban brute force prevention.**

## Project Objective
I initially setup a Caddy reverse proxy to provide secure remote access over HTTPS. However, I wanted additional peace of mind against brute force sign-in attempts so that scanners and crawlers couldn't guess my password. While Jellyfin has built-in settings to lock an account after failed sign-ins, I wanted something more robust that would block traffic from an attacker’s IP entirely at the firewall level.

## The Stack
* **Caddy:** Modified to forward the real client IP address to the Jellyfin server so it could properly log the correct source information.
* **Windows SMB:** Used to share the Jellyfin log folder from my Windows host with my Raspberry Pi.
* **Fail2Ban:** Runs on the Raspberry Pi to monitor those shared logs.
* **Iptables:** The Linux firewall used by Fail2Ban to block all traffic from an attacker's IP once a match is found.

## The "Big Fixes" (Lessons Learned)

### 1. The Polling Issue
While testing, I was initially unable to trigger the jail rule. I was simulating bad passwords with my phone on a cellular connection, and while the Jellyfin logs were recording the denied attempts, Fail2Ban was not finding the match. I verified with the fail2ban-regex tool that my filter was correct, and then learned that I needed to set the log parsing backend to "polling" rather than "auto." This was necessary to continuously update Fail2Ban's view of the logs since they were being hosted over a network share.

### 2. Content Security Policies (CSP)
I also learned about Content Security Policies. In an updated Caddyfile, I set a policy that was too extreme. I became unable to interact with the web application because the policy was blocking necessary CSS and JavaScript from loading. I had to walk these changes back to restore functionality to the Jellyfin web interface.

### 3. Session Layer and "Established" Connections
Finally, I learned about session layer issues. While testing with my phone, my cellular IP was successfully blocked in the firewall, but I could still communicate with the server. I found that this was because my mobile browser still had an existing "Established" session. Once I opened a new private tab and navigated to my server's address, I received a connection timeout. This verified that the traffic blocking was working as intended for any new connection attempts.

---

### How to use this repo:
* See /configs for sanitized examples of my Caddyfile and Fail2Ban jail settings.
* See /docs for my detailed troubleshooting notes.
