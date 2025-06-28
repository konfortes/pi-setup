# Raspberry Pi 4 Setup with Ansible

This guide will help you set up your Raspberry Pi 4 (Ubuntu Server) using Ansible from your Mac.

## Prerequisites
- A Mac with Homebrew
- Your Raspberry Pi is accessible via SSH (e.g., `raspberrypi.local`)
- You have a Cloudflare account (optional - only if you want to set up Cloudflared tunnel)

## 1. Install Ansible on your Mac
```sh
brew install ansible
```

## 2. Set Up SSH Key Authentication (Recommended)
Using SSH keys is the most secure and convenient way to connect to your Raspberry Pi with Ansible.

### a. Generate an SSH key (if you don't have one)
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```
- Press Enter to accept the default file location and set a passphrase if you wish.

### b. Copy your public key to the Raspberry Pi
```sh
ssh-copy-id -i ~/.ssh/pi_ed25519.pub pi@raspberrypi.local
```
- Enter your Pi user's password when prompted.

### c. Test your SSH connection
```sh
ssh pi@raspberrypi.local
```
- You should connect without being prompted for a password.

## 3. (Optional) Use an SSH Config File for Custom Keys
If you are using a non-default SSH key or want to simplify your SSH commands, you can create an SSH config file:

1. Open (or create) the SSH config file:
   ```sh
   vim ~/.ssh/config
   ```
2. Add the following block (replace the path with your key if needed):
   ```
   Host pi.local
       HostName raspberrypi.local
       User pi
       IdentityFile ~/.ssh/pi_ed25519
   Host pi.remote
       HostName ssh.konfortes.uk
       IdentityFile ~/.ssh/pi_ed25519
       User pi
       ProxyCommand cloudflared access ssh --hostname %h
   ```
3. Now you can connect with:
   ```sh
   # in local network
   ssh pi.local

   # Or through Cloudflare Access (setup in later phase) with:
   ssh pi.remote
   ```

## 4. Update Inventory
Edit `inventory.ini` if your Pi's hostname or user is different.

## 5. Cloudflared Tunnel Setup (Optional)

**Note**: Cloudflared tunnel setup is now optional. You can skip it by pressing Enter when prompted for the tunnel ID.

If you want to set up Cloudflared tunnel, follow these steps:

### Manual Setup (Required Before Playbook)

Some Cloudflared tunnel steps require manual setup due to browser authentication. Complete these steps on your Mac before running the playbook:

a. **Install cloudflared locally**
   ```sh
   brew install cloudflared
   ```

b. **Authenticate and create your tunnel**
   ```sh
   cloudflared tunnel login
   cloudflared tunnel create konfortes-pi
   ```
   - This will open a browser for authentication and create a credentials file (e.g., `~/.cloudflared/<TUNNEL_ID>.json`).

c. **Create a config file**
   - `vim ~/.cloudflared/config.yml`
   
   ```yaml
   tunnel: <TUNNEL_ID>
   credentials-file: ~/.cloudflared/<TUNNEL_ID>.json

   ingress:
      - hostname: n8n.konfortes.uk
        service: http://localhost:5678
      - hostname: ssh.konfortes.uk
        service: ssh://localhost:22
        originRequest:
          connectTimeout: 30s
          noTLSVerify: true
      - service: http_status:404
   ```

d. **Copy credentials and config files to your Ansible project**
   - Place them in `files/cloudflared/` (create this directory if it doesn't exist).
   - Example:
     ```sh
     mkdir -p files/cloudflared
     cp ~/.cloudflared/<TUNNEL_ID>.json files/cloudflared/
     cp ~/.cloudflared/config.yml files/cloudflared/  # if you have one
     ```

After these steps, you can run the playbook and Cloudflared will be fully configured on your Raspberry Pi. 

## 6. Run the Playbook
```sh
ansible-playbook -i inventory.ini site.yml -K
```

When prompted for the Cloudflared Tunnel ID:
- Enter your tunnel ID (without .json) to set up Cloudflared
- Press Enter to skip Cloudflared setup entirely

## 7. Next Steps
- Update the playbook as needed for your future phases (e.g., git repo clone, docker-compose systemd service).
- Configure Cloudflared tunnel as needed (see comments in the playbook).


---

## Troubleshooting


---

## Playbook Features
- Installs Docker & docker-compose
- Installs Cloudflared
- Sets up systemd services for Cloudflared

