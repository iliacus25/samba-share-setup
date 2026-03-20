# Samba for Linux and Windows File Sharing

This project documents the setup of a Samba share on a Linux laptop to enable seamless file transfer with a Windows laptop on a home network.

## Goal

Replace use of USB drives for file sharing of notes and documentation with direct network sharing. While studying for my IT certifications, it is useful to be able to transfer study notes, command-line outputs, and so on between the machine used for learning Linux on and the Windows machine I currently use a daily driver.

This project provided the opportunity for applying learnt knowledge of networking, permissions, firewall settings, and Linux command line. The project focuses ona simple, single-user, local-network setup rather than an enterprise-scale deployment.

## Environment

- Windows 11 laptop used as daily driver.
- Linux laptop running Ubuntu as a learning machine.
- Home network with router-managed DHCP.
- Samba (SMB protocol).
- UFW firewall.

## Outcome

- Instant, reliable file transfers between machines.
- Share mapped as a drive.
- UFW firewall enabled.
- DHCP address reservations to prevent IP changes from breaking connection.

## Future Improvements

Set up and integrate Pi-hole for private DNS lookups and improved security, and set up Tailscale for secure remote access without port forwarding.

---
