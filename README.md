# fedora-docker-nextcloud-aio (using custom storage) using tailscale for access outside your network
Nextcloud Docker on Fedora 44 or greater (should work lower using proper repos)

Start by downloading and installing Fedora 44 Server window.open <a href="https://fedoraproject.org/server/" target="_blank">here</a>

Once installed it should have all the basics like wget ssh etc! configure ssh to your custom port rather than the default for best security!

SSH Configuration

1. Configure SSH Daemon (For this example we will use 2926 you can use whatever you like!)
printf '%s\n' 'Port 22' 'Port 2926' | sudo tee /etc/ssh/sshd_config.d/03-custom-port.conf > /dev/null

2. Update SELinux Policy
sudo dnf install policycoreutils-python-utils -y

sudo semanage port -a -t ssh_port_t -p tcp 2926

3. Verify the change with
sudo semanage port -l | grep ssh_port_t

4. Configure the firewall
sudo firewall-cmd --permanent --add-port=2926/tcp

sudo firewall-cmd --reload

5. Restart and Verify the Firewall
sudo sshd -t

6. Verifydaemon is listening on port
sudo sshd -T | grep '^port '

sudo systemctl restart sshd.service

8. Finalize settings
ssh -p 2222 (userid@server-ip)


A. Install Fail2Ban
sudo dnf install fail2ban fail2ban-firewalld

sudo systemctl enable --now fail2ban firewalld

B. Create a local configuration file at /etc/fail2ban/jail.d/local.conf to define the SSH jail with your custom port
sudo nano /etc/fail2ban/jail.d/local.conf

C. Add the following configuration to the file
[DEFAULT]
backend = systemd
bantime = 3600
findtime = 600
maxretry = 5
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled = true
port = 2926
backend = systemd
maxretry = 5
findtime = 600
bantime = 3600

D. Validate the configuration and restart Fail2ban to apply the changes
sudo fail2ban-client -t

sudo systemctl restart fail2ban

E. Verify that the sshd jail is active and monitoring the correct port using
sudo fail2ban-client status sshd

F. Ensure that your Firewalld zone allows traffic on port 2926 for SSH to function correctly
sudo firewall-cmd --zone=FedoraWorkstation --add-port=2926/tcp --permanent

sudo firewall-cmd --reload

1. Finally we can Start with installing Docker(Doesn't sjip in fedora repos have to add docker repo)
sudo dnf install -y dnf-plugins-core

sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo

2. Install Docker CE Install the engine, CLI, containerd, and the Compose plugin
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

3. Start and Enable Docker Start the daemon and configure it to launch at boot
sudo systemctl enable --now docker

sudo systemctl status docker

4. Verify Installation by checking the versions
docker --version

docker compose version

5. To gain non root access (Optional)
sudo usermod -aG docker $USER

6. Create a directory to store your compose.yaml file
mkdir nextcloud-aio-install

7. Now navigate to nextcloud-aio-install
cd nextcloud-aio-install

8. Now download the compose.yaml from this repo and just modify it for the mounted directory your planning to use. For this example i'm using /media/data.

wget https://raw.githubusercontent.com/qcmobiltech/fedora-docker-nextcloud-aio/refs/heads/main/compose.yaml

9. Edit the file using nano
nano compose.yaml

10. Replace /media/data with your directory or disk of choice and save and exit the file!

11. Run the file by typing sudo Docker compose up -d

12. Next setup docker in fedoras firewall
sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0

sudo firewall-cmd --permanent --zone=FedoraWorkstation --add-masquerade

sudo firewall-cmd --reload

13. now install tailscale 

14. Expose docker in the fedora firewall 
sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0

sudo firewall-cmd --permanent --zone=FedoraWorkstation --add-masquerade

sudo firewall-cmd --reload

15. Install Tailscale and login once you create a machine in your account, See their website for that it's pretty basic.
curl -fsSL https://tailscale.com/install.sh | sh

It will give you a url to login to talescale

16. Then start tailscale
sudo tailscale up

17. start serving on tailscale
tailscale serve --bg http://127.0.0.1:11000
