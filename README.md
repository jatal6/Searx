# 🔎 What is Searx?

Searx is a search engine that allows you to search multiple different engines at once (also known as a metasearch engine) and removes all kinds of personal information from your queries. In this project we setup searx on an AWS EC2 instance.

# 📝 Setting up our AWS EC2 Instance

In our project, we utilized an Ubuntu server using version 20.04 LTS (t2.micro type).

![](https://files.readme.io/2560f91-image.png)

In order to enhance security for server root access, we used the amazons key pair to securely connect to our instance using OpenSSH with RSA encryption.

![](https://files.readme.io/58dca75-image.png)



Once the instance has started running, we used SSH to connect to the instance on an ubuntu based virtual machine using the following command `ssh -i ~/.ssh/searxEc2.pem ubuntu@my.instance.ip`

# 📝 Setting up the security groups

Security groups act as a firewall that are setup to only allow traffic from specified ports as well as limiting who may access the EC2 instance. In the "Security Groups" tab under Network & Security, we are able to configure our specific security groups. Based on the image below, we only allow inbound traffic from ports 4041, 80, 20, 4040, 443, and 8080.

![](https://files.readme.io/58d517c-image.png)

# :house: Setting up domain

In our project we used AWS's Route 53 service to setup a custom domain and point our instance's public IP address to redirect and route traffic to it.

![](https://files.readme.io/f0e0adb-image.png)

Once the type A DNS record is created with our instance public IP, we waited for Route 53 to propagate so we can utilize the server through the domain[` https://team1search.com/`](https://team1search.com/).

# Instillation of Docker + Searx

In our instance, we finally install Searx using the docker service using the following commands

1. Installing docker

`sudo apt install docker docker-compose`

2. Cloning the searx docker container

`git clone https://github.com/searx/searx-docker.git`

3. Use sed to replace the string in the .env file “ReplaceWithARealKey!” with a random 33 digit string using the openssl command

`sed -i "s|ReplaceWithARealKey\!|$(openssl rand -base64 33)|g" .env`

We edited the .env file and inserted our custom domain after the SEARX_HOSTNAME= variable and added an email after the LETSENCRYPT_EMAIL= variable (required to get a valid SSL certificate)

![](https://files.readme.io/6a8471e-image.png)

We then ran the 1. startup script and 2. allowed the docker container to always start on system boot, and 3. started the docker container

1. `./start.sh`
2. `systemctl enable $(pwd)/searx-docker.service`
3. `systemctl start searx-docker.service`

After this setup the team1search.com website should direct you to our server and show the searx search engine.

![](https://files.readme.io/ea749b7-image.png)

# Notes:

# Specific Service Uses

- Caddy: Reverse proxy which is what is used to create the LetsEncrypt SSL certificate
- Filtron: Filtering reverse HTTP proxy and bot abuse protection
- Morty: Privacy aware web content sanitizer proxy

# Modifying Docker-compose Files

- Modified the caddy service entity  to use our specific domain name and SSL cert email ("SEARXHOSTNAME=" & "SEARX_TLS="
- Modified the filtron service entity to listen on all ports "0.0.0.0:4040" & "0.0.0.0:4041"
- Modified the searx service entity to use our domain name and morty url ("BASE_URL=" & "MORTY_URL=")
