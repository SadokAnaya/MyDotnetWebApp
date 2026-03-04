# MyDotnetWebApp

This repository contains a small **ASP.NET Core** web application that I deployed on a **hardened Ubuntu 24.04 LTS** server running in a **VirtualBox VM**. The goal was to demonstrate a complete, repeatable deployment flow using **SSH**, **Git**, **systemd**, and **Nginx**.

## What I built (end-to-end)
- Created a public GitHub repository for a .NET web app and ensured it targets **.NET 8 (LTS)** for stable Linux deployment.
- Provisioned an Ubuntu server in VirtualBox and applied basic hardening:
  - Installed system updates (`apt update/upgrade`)
  - Enabled **UFW** with a default-deny inbound policy and only allowed **22/tcp (SSH)** and **80/tcp (HTTP)**
- Exposed the VM from the Windows host using **VirtualBox NAT port forwarding**:
  - `2222 → 22` for SSH access
  - `8080 → 80` for HTTP access

## Runtime architecture
The application runs on **Kestrel** internally and is exposed through **Nginx**:
- Kestrel binds to: `http://127.0.0.1:5000` (not publicly exposed)
- Nginx listens on port **80** and reverse-proxies requests to Kestrel on `127.0.0.1:5000`
- From the Windows host the app is reachable at: `http://127.0.0.1:8080` (forwarded to VM port 80)

## Deployment and operations
The app is managed as a **systemd service** (`mydotnetwebapp.service`) so it starts automatically and restarts on failures. I implemented a simple deployment script on the server (`~/deploy.sh`) that performs:
1. `git pull` to update the application source
2. `dotnet restore` and `dotnet publish` (Release) to produce deployable artifacts
3. `systemctl restart mydotnetwebapp` to apply the update

To demonstrate remote automation, the deployment script can be triggered from the Windows host over SSH:
```bash
