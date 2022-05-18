# Vaultwarden Secure Setup 
### Server Setup
- Rocky Linux
- Add Unprivileged user as root
```bash
useradd -g users -G wheel vault
```
- Change vault's password
```bash
passwd vault
```

### SSH (Disable and use console login or use below instructions)
  - Cat authroized_keys file to copy
  ```bash
  cat ~/.ssh/authorized_keys 
  ```
  - Switch to user vault and run below
```bash
mkdir ~/.ssh && touch ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys && vim ~/.ssh/authorized_keys
```
  - Paste keys into vim and save
  - SSHD_Config
    - Yes PubKey Auth
    - No pass auth
    - No root login
  - Exit terminal session and ssh in as vault
 
### Docker/Fail2ban Setup
- Install pre-reqs Command
```bash
sudo dnf upgrade --refresh -y && sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && sudo dnf install epel-release -y && sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin fail2ban git -y && sudo usermod -aG docker $USER && sudo systemctl enable docker && sudo reboot
```
- After reboot clone below to home directory 
```bash
git clone https://github.com/BrianTipton1/vaultwarden_stack.git
```
>Paths in compose and env file will work with /home/vault/vaultwarden_stack/

### Selinux Issues
- Copy fail2ban files to appropriate folders (/etc/fail2ban/dir)
```bash
sudo cp /home/vault/vaultwarden_stack/fail2ban/filter.d/* /etc/fail2ban/filter.d && sudo cp /home/vault/vaultwarden_stack/fail2ban/jail.d/* /etc/fail2ban/jail.d
```
- Start fail2ban service
- Look for the error messages using below command
```bash
sudo cat /var/log/audit/audit.log | grep fail2ban-server | grep AVC
``` 
- To fix a error in the log, use the above output in the below commands grep input
```bash
sudo grep 'type=AVC msg=audit(1652842936.740:275)' /var/log/audit/audit.log | audit2allow -M new_policy
```
- Keep repeating by restarting the fail2ban service and fixing the permission errors one by one with below command using the file created above
```bash
sudo semodule -i new_policy.pp
```
- Use below command to check and make sure the fail2ban service is actually not erroring even if systemd service successfully started
```bash
sudo cat /var/log/fail2ban.log
```

#### Change password/host info environment variables in stack and run stack
#### Test fail2ban setup on login and admin page
#### Disable and stop SSH when finished setting up if not already disabled
