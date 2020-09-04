
# Installation Guide
This document can be used as a quick start guide to setup the Dev Environment. Please review the system requirements listed below before moving forward with the SDO installation and deployment.
## Table of Contents
[1. System Requirements](#system-requirements)
[2. Docker Installation](#docker-installation)
[3. Docker-compose Installation](#docker-compose-installation)
[4. Other Development Tools](#other-development-tools)
[5. References](#references)

## System Requirements

| Component | Recommended |
|------- |------|
| Operating System | Ubuntu 18.04 / Windows 10 |
| Docker Engine | 18.09 |
| Docker-compose | 1.21.2 |
| maven | 3.5.4 |

## Docker* Installation
1 . Removing the older versions of Docker. If these are installed, uninstall them:
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
2 . Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:
```
 sudo apt-get update
 sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
```

!!! NOTE 
    If you are working behind a proxy, ensure to set proper proxy variables.

3 . Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
4 . Use the following command to set up the **stable** repository.
```
sudo add-apt-repository \ "deb [arch=amd64] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) \ stable"
```
5 . Update the `apt` package index and install the docker engine 18.09
```
sudo apt-get update
sudo apt-get install docker-ce=5:18.09.9~3-0~ubuntu-bionic docker-ce-cli=5:18.09.9~3-0~ubuntu-bionic containerd.io
```
6 . To run the docker system behind a proxy server, the configuration is done by the following steps:

 1. Directory `docker.service.d` is to be created in systemd directory as shown below.

    `mkdir -p /etc/systemd/system/docker.service.d`

2. For HTTP proxy, create a file **_http-proxy.conf_** in the above created directory and add the following content to this file.
	```
	[Service]
	Environment=”HTTP_PROXY=<Proxy IP/URL:Port>”
	```
3. For HTTPS proxy, create a file **_https-proxy.conf_** in the above created directory and add the following content to this file.
	```
	[Service]
	Environment=”HTTPS_PROXY=<Proxy IP/URL:Port>”
	```
4. Next, create a directory named **_.docker_** in the user home path (**~/**) and a create a file named **_config.json_** if not present, add the following content.

	```
	 { "proxies":
		 { "default":
			 {
			 "httpProxy": "<Proxy IP/URL:Port>",
			 "httpsProxy": "<Proxy IP/URL:Port>"
			 }
		 }
	}
	```
5. After configuring the above, the docker needs to be restarted.

	```
	sudo systemctl daemon-reload
	sudo systemctl restart docker
	```
6. To ensure that the proxies are set successfully, run the following command

	`sudo systemctl show --property Environment docker`

7 . Verify that Docker Engine is installed correctly by running the `hello-world` image.

	 sudo docker run hello-world

## Docker-compose Installation
To install a specific version of docker-compose (for example **_1.21.2_**) follow these steps:

1 . download the specific version **(1.21.2)** of docker-compose
```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/bin/docker-compose
```
2 . To apply executable permissions, run the following command
```
sudo chmod +x /usr/bin/docker-compose
```
3 . To ensure that the required version is installed, run ` docker-compose --version` command

## Other Development Tools

1 . To install OpenJDK*

  `sudo apt install openjdk-11-jdk-headless`

2 . To install Maven*

  `sudo apt install maven`

3 . To set the correct system time

  ```
  sudo date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"
  ```
  Ensure that the system time is correct, else you will receive the certificate expiration error.
  
  Change Google* domain according to your location.

## References

[https://docs.docker.com/engine/install/ubuntu/#installation-methods](https://docs.docker.com/engine/install/ubuntu/#installation-methods)
[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
[https://docs.docker.com/network/proxy/](https://docs.docker.com/network/proxy/)
