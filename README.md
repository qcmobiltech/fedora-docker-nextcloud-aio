# fedora-docker-nextcloud-aio (using custom storage)
Nextcloud Docker on Fedora 44 or greater (should work lower using proper repos)

Start by downloading and installing Fedora 44 Server window.open <a href="https://fedoraproject.org/server/" target="_blank">here</a>

once installed it should have all the basics like wget ssh etc! configure ssh to your custom port rather than the default for best security!

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
