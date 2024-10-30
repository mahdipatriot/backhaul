#!/bin/bash

# Colors for beautification
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[1;34m'
NC='\033[0m' # No Color

function install_backhaul() {
    echo -e "${BLUE}Creating backhaul directory and downloading Backhaul...${NC}"
    sudo mkdir -p /root/backhaul && cd /root/backhaul
    sudo wget https://github.com/Musixal/Backhaul/releases/download/v0.5.1/backhaul_linux_amd64.tar.gz
    sudo tar -xzvf backhaul_linux_amd64.tar.gz
    sudo rm backhaul_linux_amd64.tar.gz

    read -p "Is this an Iran server (y/n)? " ir_server
    read -p "Are you using IPv4 or IPv6 (4/6)? " ip_version
    if [[ "$ip_version" == "4" ]]; then
        ip="0.0.0.0"
    else
        ip="[::]"
    fi

    echo -e "${YELLOW}Enter ports (comma-separated):${NC}"
    read ports_input
    ports=($(echo $ports_input | tr ',' ' '))
    
    read -p "Enter token: " token
    read -p "Enable nodelay (true/false): " nodelay
    read -p "Enter tunnel port: " tunnelport

    if [[ "$ir_server" == "y" ]]; then
        config="[server]\nbind_addr = \"$ip:$tunnelport\"\ntransport = \"tcp\"\ntoken = \"$token\"\nkeepalive_period = 75\nnodelay = $nodelay\nheartbeat = 40\nchannel_size = 2048\nsniffer = false\nweb_port = 2060\nsniffer_log = \"/root/backhaul.json\"\nlog_level = \"info\"\nports = ["
        
        for port in "${ports[@]}"; do
            config+="\"$port=$port\","
        done
        config=${config%,}"]"
        echo -e "${GREEN}Config generated for Iran server:${NC}"
        echo -e "$config"
        echo -e "$config" | sudo tee /root/backhaul/config.toml

    else
        read -p "Enter Iran server IP: " ipkharej
        if [[ "$ip_version" == "6" ]]; then
            ipkharej="[$ipkharej]"
        fi
        read -p "Enter connection pool number: " cpool

        config="[client]\nremote_addr = \"$ipkharej:3080\"\ntransport = \"tcp\"\ntoken = \"$token\"\nconnection_pool = $cpool\nkeepalive_period = 75\ndial_timeout = 10\nnodelay = $nodelay\nretry_interval = 3\nsniffer = false\nweb_port = 2060\nsniffer_log = \"/root/backhaul.json\"\nlog_level = \"info\""
        echo -e "${GREEN}Config generated for non-Iran server:${NC}"
        echo -e "$config"
        echo -e "$config" | sudo tee /root/backhaul/config.toml
    fi

    echo -e "${BLUE}Creating systemd service...${NC}"
    service_content="[Unit]\nDescription=Backhaul Reverse Tunnel Service\nAfter=network.target\n\n[Service]\nType=simple\nExecStart=/root/backhaul/backhaul -c /root/backhaul/config.toml\nRestart=always\nRestartSec=3\nLimitNOFILE=1048576\n\n[Install]\nWantedBy=multi-user.target"
    echo -e "$service_content" | sudo tee /etc/systemd/system/backhaul.service

    sudo systemctl daemon-reload
    sudo systemctl enable backhaul.service
    sudo systemctl start backhaul.service
    echo -e "${GREEN}Backhaul service installed and started.${NC}"
}

function uninstall_backhaul() {
    read -p "Are you sure you want to uninstall Backhaul (y/n)? " confirm
    if [[ "$confirm" == "y" ]]; then
        sudo systemctl daemon-reload
        sudo systemctl disable backhaul.service
        sudo systemctl stop backhaul.service
        sudo rm /etc/systemd/system/backhaul.service
        sudo rm -r /root/backhaul
        echo -e "${RED}Backhaul uninstalled.${NC}"
    else
        echo -e "${YELLOW}Uninstall cancelled.${NC}"
    fi
}

function update_backhaul() {
    sudo rm /root/backhaul/backhaul_linux_amd64.tar.gz
    sudo wget https://github.com/Musixal/Backhaul/releases/download/v0.5.1/backhaul_linux_amd64.tar.gz
    sudo tar -xzvf backhaul_linux_amd64.tar.gz
    sudo rm backhaul_linux_amd64.tar.gz
    sudo systemctl restart backhaul.service
    echo -e "${GREEN}Backhaul updated and service restarted.${NC}"
}

function restart_backhaul_service() {
    sudo systemctl restart backhaul.service
    echo -e "${GREEN}Backhaul service restarted.${NC}"
}

function setup_cronjob() {
    echo -e "${YELLOW}Choose a time interval for the cronjob to restart backhaul.service:${NC}"
    echo "1) Every 1 hour"
    echo "2) Every 3 hours"
    echo "3) Every 6 hours"
    echo "4) Every 12 hours"
    echo "5) Every 24 hours"
    read -p "Choose an option (1-5): " cron_option

    case $cron_option in
        1) interval="0 * * * *" ;;
        2) interval="0 */3 * * *" ;;
        3) interval="0 */6 * * *" ;;
        4) interval="0 */12 * * *" ;;
        5) interval="0 0 * * *" ;;
        *) echo -e "${RED}Invalid option.${NC}" && return ;;
    esac

    croncmd="systemctl restart backhaul.service"
    (sudo crontab -l 2>/dev/null; echo "$interval $croncmd") | sudo crontab -
    echo -e "${GREEN}Cronjob set to restart backhaul.service every $interval.${NC}"
}

function main_menu() {
    clear
    echo -e "${BLUE}=== Backhaul Script Menu By eshghe Meysoo koso  ===${NC}"
    echo -e "${RED}=== https://github.com/mahdipatriot/backhaul  ===${NC}"
    echo -e "${RED}=== Credits : https://github.com/Musixal/Backhaul  ===${NC}"
    echo "1) Install Backhaul"
    echo "2) Uninstall Backhaul"
    echo "3) Update Backhaul"
    echo "4) Restart Backhaul Service"
    echo "5) Setup Cronjob to Restart Backhaul Service"
    echo "6) Exit"
    echo "=========================="
    read -p "Choose an option: " option

    case $option in
        1) install_backhaul ;;
        2) uninstall_backhaul ;;
        3) update_backhaul ;;
        4) restart_backhaul_service ;;
        5) setup_cronjob ;;
        6) echo -e "${GREEN}Exiting...${NC}" && exit 0 ;;
        *) echo -e "${RED}Invalid option!${NC}" && main_menu ;;
    esac
}

# Main loop
while true; do
    main_menu
done
