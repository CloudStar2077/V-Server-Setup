# V-Server-Setup
An Nginx webserver and Git should be installed on the machine. To harden the server, asymmetric encryption using a SSH key pair will be configured, and password-based login will be disabled.


# Table of Contents

1. [Connecting to the Virtual Machine](#connecting-to-the-virtual-machine)
2. [Creating and Deploying the SSH Key Pair](#creating-and-deploying-the-ssh-key-pair)
3. [Disable Password Authentication](#disable-password-authentication)
4. [Installation and Configuration Nginx Webserver](#installation-and-configuration-nginx-webserver)
5. [Git Installation and Login](#git-installation-and-login)


## 1. Connecting to the Virtual Machine

To begin setting up the virtual machine, a SSH connection is established to the target host. For this, the server IP address and user credentials are required.

`ssh "username"@"target-ip"`

Since this is the first connection and the server’s public key is not yet stored in the local known_hosts file, the following message appears:
  
  The authenticity of host '... (IP address)' can't be established.
  ECDSA key fingerprint is SHA256:....
  Are you sure you want to continue connecting (yes/no/[fingerprint])?

This must be confirmed by typing: `yes`

After logging into the server, I first get an overview of the environment:

`whoami` → shows the current user

`id` → displays group memberships

This allows me to verify that I am part of the sudo group, which enables the execution of commands with elevated privileges.

## 2. Creating and Deploying the SSH Key Pair

Next, the key pair must be generated on the local machine:

`logout`

`ssh-keygen -t ed25519`
   
This sets the encryption type to ed25519, which is currently one of the most modern and secure algorithms.
For this project, no passphrase is set.
In the hidden directory ~/.ssh, both the private and public keys are now stored.
The public key is copied to the target host using:

`ssh-copy-id -i /home/user/.ssh/demo_ed25519.pub "username"@target-ip`

The `-i` option specifies the identity file (our public key).

## 3. Disable Password Authentication

In the file /etc/ssh/sshd_config, we edit the line PasswordAuthentication from yes to no and remove the # at the beginning of the line to disable the comment function. 
You can open the text editor with "vim".

`vim /etc/ssh/sshd_config`

## 4. Installation and configuration Nginx webserver

Before installing the web server, the packages should be updated:

`sudo apt update && sudo apt install nginx -y`

The double ampersand (&&) allows you to chain two commands together. However, the second command will only be executed if the first one was successful (exit code 0).
The `-y` option automatically confirms the installation.

After installation, the service status is checked:

`systemctl status nginx.service`

<img width="1256" height="431" alt="2026-02-19_22-39" src="https://github.com/user-attachments/assets/4d275c6b-0a2d-4a29-a0e1-5804b825eab0" />

If the preset shows enabled, this indicates that the service will automatically start after a reboot.

Now, entering the server IP address in a web browser displays the default Nginx welcome page, which will be modified in the next step.

For this purpose, a new directory is created under /var/www/alternatives

`sudo mkdir /var/www/alternatives`

Because normal users do not have permission to make edits in this directory, the command "sudo" is used beforehand, which means SuperUserDo.

In this newly created directory, the text file alternate-index.html will now be created; this is achieved using the "touch" command.

`sudo touch /var/www/alternatives/alternate-index.html`

To configure the web server to load the newly defined alternative website, the following lines are added to /etc/nginx/sites-enabled/alternatives.

`sudo vim /etc/nginx/sites-enabled/alternatives`

<img width="611" height="293" alt="2026-02-26_14-30" src="https://github.com/user-attachments/assets/16ea08b1-427d-4e0b-aad7-96d7ea08d5a7" /> <br>

Of course, the alternate-index.html file also needs to be filled with HTML code. 

`sudo vim /var/www/alternatives/alternate-index.html`

<img width="976" height="378" alt="2026-02-26_14-31" src="https://github.com/user-attachments/assets/ce166273-f045-4780-af09-dd6f7b5c2875" /> <br>


To ensure that the default nginx website is no longer displayed and users are directed to the alternative website, port 80 is disabled and a redirect to port 8081 is set up.

Comment out the following entry (listen):

`sudo vim /etc/nginx/sites-available/default`

```
# Default server configuration
#
server {
       #        listen 80 default_server;
       #        listen [::]:80 default_server;

  ```

Create a new file and add the following server block 

`sudo vim /etc/nginx/sites-available/redirect_80.conf`

```
Server {
    listen 80;
    server_name "ip-address";

    return 301 http://$host:8081$request_uri;
}
  ```

Configure and test nginx                    
```
sudo nginx -t       
sudo systemctl restart nginx
```
To verify, open a web browser and access the target IP address

<img width="478" height="181" alt="2026-02-27_20-42" src="https://github.com/user-attachments/assets/468338cc-d5a5-4d28-973b-868dbd87c4ff" /> <br>


## 5. Git Installation and Login

A key pair is also created on the virtual server.

`ssh-keygen -t ed25519`

Copy the public key.

`cat ~/.ssh/github-key.pub`

Then add it to the GitHub web interface under SSH and GPG Keys settings.

To install Git, use the command `sudo apt install git -y` .

The installation is then verified with `git --version`. The output should display the Git version.

<img width="456" height="58" alt="2026-02-23_18-59" src="https://github.com/user-attachments/assets/202b1177-7987-4179-b58b-308cbded5e1b" /> <br>

Now, the credentials need to be entered using the following commands:
```
git config --global user.name "Your Name"

git config --global user.email "your.email@example.com"
  ```
To check the configuration:

`git config --list`

Then Create a new local Branch in the home directory

`mkdir -p ~/Git/V-Server-Setup`

After that the repository needs to be initialize

`cd ~/Git/V-Server-Setup`
`git init`

Test if it works 

`ssh -T git@github.com`

<img width="1261" height="77" alt="2026-02-25_17-37" src="https://github.com/user-attachments/assets/85c08a35-1976-4b15-a270-9c90b22e123d" /> <br>

Finally its time to push the local repo to the github remote host 

```console
touch README.md 

git add README.md

git push -u origin main

#In order to execute pull requests, a feature branch is created.

git checkout -b feature/update-readme

git push -u origin feature/update-readme
  ```


   
