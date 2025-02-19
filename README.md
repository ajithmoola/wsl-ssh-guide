# How to Connect to WSL through SSH from a Mac

Windows Subsystem for Linux (WSL) is a powerful feature that allows you to run a Linux environment directly on Windows. This guide will walk you through the process of setting up SSH access to your WSL environment from a Mac.

## Step 1: Set Up SSH Server in WSL

1. Open your WSL distribution.
2. Update your package list:
   ```bash
   sudo apt update
   ```
3. Install the OpenSSH server:
   ```bash
   sudo apt install openssh-server
   ```
4. Edit the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
5. Ensure the following lines are present and uncommented:
   ```
   Port 2222
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```
6. Save and exit the file (Ctrl+X, then Y, then Enter).
7. Start the SSH service:
   ```bash
   sudo service ssh start
   ```

## Step 2: Set Up Port Forwarding on Windows

1. Open PowerShell as Administrator on Windows.
2. Get your WSL IP address and set up port forwarding:
   ``` powershell
   netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=2222 connectaddress=$((wsl hostname -I).trim())
   ```

## Step 3: Configure Windows Firewall

1. Open Windows Defender Firewall with Advanced Security.
2. Click on "Inbound Rules" and then "New Rule".
3. Choose "Port" and click Next.
4. Select "TCP" and enter "2222" for the port number.
5. Allow the connection and apply the rule to all profiles.
6. Name the rule (e.g., "WSL SSH") and finish the wizard.

## Step 4: Generate SSH Key on Mac

1. Open Terminal on your Mac.
2. Generate a new SSH key:
   ```bash
   ssh-keygen -t rsa -b 4096
   ```
3. Follow the prompts, using the default file location and adding a passphrase if desired.

## Step 5: Copy SSH Key to WSL

1. Display your public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
2. Copy the output.
3. In your WSL terminal:
   ```bash
   mkdir -p ~/.ssh
   nano ~/.ssh/authorized_keys
   ```
4. Paste your public key into this file, save, and exit.
5. Set correct permissions:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

## Step 6: Configure SSH on Mac

1. Edit your SSH config file:
   ```bash
   nano ~/.ssh/config
   ```
2. Add the following:
   ```
   Host wsl
       HostName <WINDOWS_IP>
       User <WSL_USERNAME>
       Port 2222
       IdentityFile ~/.ssh/id_rsa
   ```
   Replace `<WINDOWS_IP>` with your Windows machine's IP address and `<WSL_USERNAME>` with your WSL username.

## Step 7: Connect from Mac to WSL

1. In your Mac's terminal, connect using:
   ```bash
   ssh wsl
   ```

You should now be connected to your WSL environment!

## Common Issues and Resolutions

Despite following the steps above, you might encounter some issues. Here are some common problems and how to resolve them:

### 1. Connection Timeout

**Symptom:** SSH connection attempt results in a timeout.

**Possible causes and solutions:**
- WSL SSH service is not running. 
  - Solution: In WSL, run `sudo service ssh start`
- Port forwarding is not set up correctly. 
  - Solution: Check with `netsh interface portproxy show v4tov4`
  - If empty, set up port forwarding as described in Step 2
- Windows Firewall is blocking the connection. 
  - Solution: Verify the firewall rule for port 2222 is active

### 2. Authentication Failure

**Symptom:** Connection establishes, but you get a "Permission denied (publickey)" error.

**Possible causes and solutions:**
- The public key is not properly added to `authorized_keys`.
  - Solution 1: Verify the content of `~/.ssh/authorized_keys` in WSL
  - Solution 2: use `ssh-copy-id` to update `~/.ssh/authorized_keys` on linux:
     1. Enable password login from Linux by updating `/etc/ssh/sshd_config`, set `PasswordAuthentication` to  `yes`
     2. Run `ssh-copy-id` from your mac
        ``` bash
        ssh-copy-id -i ~/.ssh/id_rsa -n -p 2222 <your user name>@<your windows machine name or ip address>
        ```
- Incorrect permissions on SSH files.
  - Solution: In WSL, run:
    ```bash
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    ```
- Incorrect filename for authorized keys.
  - Solution: Ensure the file is named `authorized_keys`, not `authorized_users` or anything else

### 3. SSH Server Refuses Connection

**Symptom:** You get a "Connection refused" error.

**Possible causes and solutions:**
- SSH server is not running on the specified port.
  - Solution: Check SSH config in WSL (`/etc/ssh/sshd_config`) to ensure it's set to use port 2222
- WSL instance is not running.
  - Solution: Open a WSL terminal on your Windows machine to start the instance

### 4. Host Key Verification Failed

**Symptom:** You get a "Host key verification failed" error.

**Possible causes and solutions:**
- The host key has changed (common if you've reinstalled WSL).
  - Solution: Remove the old key from your Mac's `known_hosts` file:
    ```bash
    ssh-keygen -R "[your_windows_ip]:2222"
    ```

### 5. WSL IP Address Changes

**Symptom:** Connection worked before, but suddenly stops working.

**Possible causes and solutions:**
- WSL IP address has changed after a restart.
  - Solution: Update the port forwarding rule with the new IP:
    1. In WSL, get the new IP: `ip addr show eth0`
    2. In Windows PowerShell (as admin), update the rule:
       ```powershell
       netsh interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0
       netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=2222 connectaddress=<NEW_WSL_IP>
       ```

### 6. Incorrect Default Shell

**Symptom:** You connect successfully but get an unexpected shell environment.

**Possible causes and solutions:**
- Windows OpenSSH is not using the WSL shell.
  - Solution: Set the default shell for SSH connections:
    1. In Windows PowerShell (as admin):
       ```powershell
       New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\wsl.exe" -PropertyType String -Force
       ```
    2. Restart the SSH service: `Restart-Service sshd`

Remember, when troubleshooting SSH connections, the verbose mode (`ssh -v wsl`) can provide helpful diagnostic information. Don't hesitate to use it when you encounter issues.

By being aware of these common issues and their solutions, you'll be better prepared to troubleshoot any problems that arise when setting up SSH access to your WSL environment from your Mac.

## Contributing

If you have suggestions for improving this guide, please feel free to create an issue or submit a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
