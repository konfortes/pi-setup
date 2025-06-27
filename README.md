# Raspberry Pi 4 Setup with Ansible

This guide will help you set up your Raspberry Pi 4 (Ubuntu Server) using Ansible from your Mac.

## Prerequisites
- A Mac with Homebrew
- Your Raspberry Pi is accessible via SSH (e.g., `raspberrypi.local`)
- You have a Cloudflare account

## 1. Install Ansible on your Mac
```sh
brew install ansible
```

## 2. (Optional, Recommended) Generate and Copy SSH Key to Raspberry Pi
By default, this setup uses SSH password authentication. You can add your SSH key later for easier access:
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id ubuntu@raspberrypi.local
```

## 3. Clone this repository
```sh
git clone <this-repo-url>
cd <repo-folder>
```

## 4. Update Inventory
Edit `inventory.ini` if your Pi's hostname or user is different.

## 5. Run the Playbook (with password authentication)
When prompted, enter your Pi user's password:
```sh
ansible-playbook -i inventory.ini site.yml -k
```
- The `-k` flag tells Ansible to prompt for the SSH password.

## 6. Next Steps
- Update the git repo URL in the playbook (see comments in `site.yml`).
- Configure Cloudflared tunnel as needed (see comments in the playbook).
- (Optional) Set up SSH key authentication for future runs.

---

## Playbook Features
- Installs Docker & docker-compose
- Installs Cloudflared
- Clones your docker-compose repo
- Installs Ollama (dockerized)
- Sets up systemd services for Ollama, Cloudflared, and docker-compose 