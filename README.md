# V-Server-Setup

An Nginx web server and Git will be deployed on the target system. To enhance security, asymmetric encryption will be implemented through SSH key-based authentication, and password-based authentication will be permanently disabled.

# Table of Contents

1. [Connecting to the Virtual Machine](#connecting-to-the-virtual-machine)
2. [Creating and Deploying the SSH Key Pair](#creating-and-deploying-the-ssh-key-pair)
3. [Disable Password Authentication](#disable-password-authentication)
4. [Installation and Configuration Nginx Webserver](#installation-and-configuration-nginx-webserver)
5. [Git Installation and Login](#git-installation-and-login)


## 1. Connecting to the Virtual Machine

To begin setting up the virtual machine, a SSH connection is established to the target host. For this, the server IP address and user credentials are required.

```bash 
ssh "username"@"target-ip"`
```
If this is the first connection, the server’s public key is not yet stored in the local `known_hosts` file, the following message appears:
```bash
  The authenticity of host ... (IP address)' can't be established.
  ECDSA key fingerprint is SHA256:....
  Are you sure you want to continue connecting (yes/no/[fingerprint])?
  ```
This must be confirmed by typing: `yes`

After logging into the server, you can get an overview of the environment by typing:

- `whoami` → shows the current user

- `id` → displays group memberships

This allows you to verify if you are part of the sudo group, which enables the execution of commands with elevated privileges.

## 2. Creating and Deploying the SSH Key Pair

Generate the SSH Key pair on the local machine:

- `logout` --> logout from V-Server

- `ssh-keygen -t ed25519` --> generates a SSH Key Pair
   
This sets the encryption type to ed25519, which is currently one of the most modern and secure algorithms.
For this project, no passphrase is set.
In the hidden directory `~/.ssh`, both the private and public keys are now stored.
Copy the public key to the target host using:
```bash
ssh-copy-id -i /home/user/.ssh/"yourkey.pub" "username"@"target-ip"
 ```
The `-i` option specifies the identity file (public key).

## 3. Disable Password Authentication

In the file /etc/ssh/sshd_config, edit the line PasswordAuthentication from yes to no and remove the # at the beginning of the line to disable the comment function. 
You can open the text editor with `vim`.
```bash
vim /etc/ssh/sshd_config
  ```
## 4. Installation and configuration Nginx webserver

Before installing the web server, the packages should be updated:
```bash
sudo apt update && sudo apt install nginx -y
 ```
The double ampersand (&&) allows you to chain two commands together. However, the second command will only be executed if the first one was successful (exit code 0).
The `-y` option automatically confirms the installation.

After installation, check the service status :
```bash
systemctl status nginx.service
  ```
<img width="1256" height="431" alt="2026-02-19_22-39" src="https://github.com/user-attachments/assets/4d275c6b-0a2d-4a29-a0e1-5804b825eab0" />

If the preset shows enabled, this indicates that the service will automatically start after a reboot.

Entering the server IP address in a web browser will open the default Nginx welcome page, which will be modified in the next step.

For this purpose, create a new directory under /var/www/alternatives
```bash
sudo mkdir /var/www/alternatives
  ```
Because normal users do not have permission to make edits in this directory, the command "sudo" is used beforehand, which means SuperUserDo.

In this new directory, create the text file alternate-index.html, this is achieved by using the `touch` command.
```bash
sudo touch /var/www/alternatives/alternate-index.html
 ```

Configure the web server to load the newly defined alternative website, add the following lines to /etc/nginx/sites-enabled/alternatives.
```bash
sudo vim /etc/nginx/sites-enabled/alternatives
 ```
<img width="611" height="293" alt="2026-02-26_14-30" src="https://github.com/user-attachments/assets/16ea08b1-427d-4e0b-aad7-96d7ea08d5a7" /> <br>

Write the following HTML Code to the alternate-index.html file. 
```bash
sudo vim /var/www/alternatives/alternate-index.html
  ```
<img width="976" height="378" alt="2026-02-26_14-31" src="https://github.com/user-attachments/assets/ce166273-f045-4780-af09-dd6f7b5c2875" /> <br>


To ensure that the default nginx website is no longer displayed and users are directed to the alternative website, disable port 80 and set up a redirect to port 8081.

Comment out the following entry (listen):
```bash
`sudo vim /etc/nginx/sites-available/default`

# Default server configuration
#
server {
       #        listen 80 default_server;
       #        listen [::]:80 default_server;
  ```

Create a new file and add the following server block 
```bash
`sudo vim /etc/nginx/sites-available/redirect_80.conf`

Server {
    listen 80;
    server_name "ip-address";

    return 301 http://$host:8081$request_uri;
}
  ```

Configure and test nginx                    
```bash
sudo nginx -t       
sudo systemctl restart nginx
  ```
To verify, open a web browser and access the target IP address

<img width="583" height="142" alt="2026-02-27_21-21" src="https://github.com/user-attachments/assets/1d005e9f-dda5-4b90-9edd-d60f6d533032" /> <br>

## 5. Git Installation and Login

Generate a key pair on the V Server.
```bash
ssh-keygen -t ed25519
  ```
Copy the public key.
```bash
cat ~/.ssh/github-key.pub
 ```
Then add it to the GitHub web interface under SSH and GPG Keys settings.

To install Git, use the command 
```bash
sudo apt install git -y .
 ```
To verify the installation type 
```bash
git --version
 ```
The output should display the Git version.

<img width="456" height="58" alt="2026-02-23_18-59" src="https://github.com/user-attachments/assets/202b1177-7987-4179-b58b-308cbded5e1b" /> <br>

Enter the credentials using the following commands:

```bash
git config --global user.name "Your Name"

git config --global user.email "your.email@example.com"
 ```
Check the configuration:

```bash
git config --list
 ```
Then Create a new local Branch in the home directory
```bash
mkdir -p ~/Git/V-Server-Setup
 ```
Initialize the repository
```bash
cd ~/Git/V-Server-Setup
git init
 ```
Test if it works 
```bash
ssh -T git@github.com
 ```
<img width="1261" height="77" alt="2026-02-25_17-37" src="https://github.com/user-attachments/assets/85c08a35-1976-4b15-a270-9c90b22e123d" /> <br>

Push the local repo to the github remote host 

```bash
touch README.md 

git add README.md

git push -u origin main

#In order to execute pull requests, a feature branch is created.

git checkout -b feature/update-readme

git push -u origin feature/update-readme
  ```


   
