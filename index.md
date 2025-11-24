# Wireguard Project

## Create an account on digitalocean.com
1. sign up for a new account using: https://m.do.co/c/d33d59113ab6
2. I used gmail to log in
3. Click on "billing" on the left panel to attach a credit card to your account
4. Select "droplet" when asked what you want to create
## Create an Ubuntu droplet
- Settings:
	-Image: Ubuntu
	-Version: 24.04
	-Droplet type: basic
	-CPU option: regular -> $6/month
	-Create a root password
	-Datacenter: any is good, but I chose San Francisco - Datacenter 2

## Installing Docker using the Droplet
1. open your computer's terminal and ssh into your droplet:
```
ssh root@xxx.xxx.xxx.xxx
```
- since I set a password, I have to use that to log in
#### Setting up Docker's apt repository
- Add Docker's GPG key
```
apt update
```
```
apt install ca-certificates curl
```
```
install -m 0755 -d /etc/apt/keyrings
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
```
chmod a+r /etc/apt/keyrings/docker.asc
```
- Add the repository to the Apt sources
```
tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
- update the system again
#### Install Docker and Docker Compose
```
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- verify the installation was successful
```
docker run hello-world
```

#### Set up Wireguard
1. make a directory to store it
```
mkdir -p ~/wireguard/
```
```
mkdir -p ~/wireguard/config
```
2. Create docker compose yml file
```
nano ~/wireguard/docker-compose.yml
```
3. Paste contents into the file
```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Hong_Kong
      - SERVERURL=157.245.183.54 (use your ip)
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0/24
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

#### Generate QR Code
1. Before turning on your vpn, go to ipleak.net and screenshot your current ip address
[phone ip before](phone ip addr before.jpg)
2. Next, in the wireguard directory, run
```
docker compose up -d
```
```
docker logs -f wireguard
```
3. You will be presented with three QR codes: PEER pc1, PEER pc2, PEER phone1. I am using my phone to connect to the vpn
###### Connecting to the VPN on a phone
1. Download the Wireguard app on your mobile device
2. Click "add tunnel" and choose "create from Qr code"
3. Scan your QR code
4. Allow the app to make configurations and toggle the VPN to on in the app
5. Revisit ipleak.net and capture your new ip address
[phone ip after](phone ip addr after.jpg)

###### Connecting to the VPN on your computer
1. Visit ipleak.net and capture your current IP address
[computer ip before](Screenshot_23-11-2025_11243_ipleak.net.jpeg)
2. See where the config files are located
```
ls -R config
```
3. Download peer_pc1.conf onto your computer (exit ssh session)
```
scp root@157.245.183.54:~/wireguard/config/peer_pc1/peer_pc1.conf .
```
4. Download Wireguard on your pc using https://www.wireguard.com/install/
5. After it downloads, click "add tunnel" and find your config file
6. Activate the VPN
7. Revisit ipleak.net and capture your new IP address
[computer ip after](Screenshot_23-11-2025_113516_ipleak.net.jpeg)
#### Destroy your Droplet
1. Go to your account settings on DigitalOcean and deactivate your account. Make sure to purge your data before submitting the deactivation.
