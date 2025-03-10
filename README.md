# assignment2
#!/bin/bash

notify_action() {
    echo "Action: $1"
}

notify_action "Checking network configuration..."
netplan_file="/etc/netplan/00-installer-config.yaml"
if ! grep -q "192.168.16.21/24" "$netplan_file"; then
    notify_action "Updating network configuration..."
    sed -i 's/dhcp4: true/dhcp4: false/g' "$netplan_file"
    echo -e "network:\n  version: 2\n  renderer: networkd\n  ethernets:\n    eth0:\n      dhcp4: false\n      addresses: [192.168.16.21/24]" >> "$netplan_file"
    sudo netplan apply
    notify_action "Network configuration updated."
else
    notify_action "Network already configured."
fi

notify_action "Checking /etc/hosts file..."
if ! grep -q "192.168.16.21 server1" /etc/hosts; then
    notify_action "Updating /etc/hosts..."
    sed -i '/server1/d' /etc/hosts
    echo "192.168.16.21 server1" >> /etc/hosts
    notify_action "/etc/hosts updated."
else
    notify_action "/etc/hosts already correct."
fi

notify_action "Checking Apache2 installation..."
if ! systemctl is-active --quiet apache2; then
    sudo apt update -y
    sudo apt install apache2 -y
    sudo systemctl enable apache2
    sudo systemctl start apache2
    notify_action "Apache2 installed and started."
else
    notify_action "Apache2 already running."
fi

notify_action "Checking Squid installation..."
if ! systemctl is-active --quiet squid; then
    sudo apt install squid -y
    sudo systemctl enable squid
    sudo systemctl start squid
    notify_action "Squid installed and started."
else
    notify_action "Squid already running."
fi

users=("dennis" "aubrey" "captain" "snibbles" "brownie" "scooter" "sandy" "perrier" "cindy" "tiger" "yoda")
for user in "${users[@]}"; do
    if ! id "$user" &>/dev/null; then
        sudo useradd -m -s /bin/bash "$user"
        sudo mkdir -p /home/$user/.ssh
        sudo touch /home/$user/.ssh/authorized_keys
        sudo chmod 700 /home/$user/.ssh
        sudo chmod 600 /home/$user/.ssh/authorized_keys
        sudo chown -R $user:$user /home/$user/.ssh
        notify_action "User $user created."
    else
        notify_action "User $user already exists."
    fi
done

notify_action "Adding SSH key for dennis..."
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG4rT3vTt99Ox5kndS4HmgTrKBT8SKzhK4rhGkEVGlCI student@generic-vm" > /home/dennis/.ssh/authorized_keys
sudo chmod 600 /home/dennis/.ssh/authorized_keys
sudo chown dennis:dennis /home/dennis/.ssh/authorized_keys
notify_action "SSH key added for dennis."

notify_action "Script completed."


