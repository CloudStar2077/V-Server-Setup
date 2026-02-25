# V-Server-Setup
A Nginx webserver and Git should be installed on the machine. To harden the server, asymmetric encryption using a SSH key pair will be configured, and password-based login will be disabled.


# Table of Contents

1. [Connecting to the Virtual Machine](#connecting-to-the-virtual-machine)
2. [Creating and Deploying the SSH Key Pair](#creating-and-deploying-the-ssh-key-pair)
3. [Disable Password Authentication](#disable-password-authentication)
4. [Installation and Configuration Nginx Webserver](#installation-and-configuration-nginx-webserver)
5. [Git Installation and Login](#git-installation-and-login)


## 1. Connecting to the Virtual Machine

To begin setting up the virtual machine, a SSH connection is established to the target host. For this, the server IP address and user credentials are required.

`ssh vladimir-ivic@167.235.27.211`

Since this is the first connection and the server’s public key is not yet stored in the local known_hosts file, the following message appears:
  
  The authenticity of host '... (IP address)' can't be established.
  ECDSA key fingerprint is SHA256:....
  Are you sure you want to continue connecting (yes/no/[fingerprint])?

This must be confirmed by typing: `yes`

After logging into the server, I first get an overview of the environment:

`whoami` → shows the current user

`id` → displays group memberships

<img width="1058" height="225" alt="2026-02-19_17-19" src="https://github.com/user-attachments/assets/acb699ba-cbc1-46f8-8dfe-c7698f7cbffa" />

This allows me to verify that I am part of the sudo group, which enables the execution of commands with elevated privileges.

The `hostname` command displays the machine name.

## 2. Creating and Deploying the SSH Key Pair

Next, the key pair must be generated on the local machine:

`logout`

`ssh-keygen -t ed25519`
   
This sets the encryption type to ed25519, which is currently one of the most modern and secure algorithms.
For this project, no passphrase is set.
In the hidden directory ~/.ssh, both the private and public keys are now stored.
The public key is copied to the target host using:

`ssh-copy-id -i /home/user/.ssh/demo_ed25519.pub vladimir-ivic@167.235.27.211`

The `-i` option specifies the identity file (our public key).

<img width="1059" height="385" alt="xyz" src="https://github.com/user-attachments/assets/108f49d5-9931-4a0a-8d8a-8f5496cef497" />   

## 3. Disable Password Authentication

In the file /etc/ssh/sshd_config, we edit the line PasswordAuthentication from yes to no and remove the # at the beginning of the line to disable the comment function. 
You can open the text editor with "vim".

`vim /etc/ssh/sshd_config`

<img width="815" height="68" alt="2026-02-19_21-46" src="https://github.com/user-attachments/assets/c0eedefa-9c41-490b-a745-2c9a5fd59ef3" />

## 4. Installation and configuration Nginx webserver

Before installing the web server, the packages should be updated:

`sudo apt update && sudo apt install nginx -y`

The double ampersand (&&) allows you to chain two commands together. However, the second command will only be executed if the first one was successful (exit code 0).
The `-y` option automatically confirms the installation.

After installation, the service status is checked:

`systemctl status nginx.service`

<img width="1256" height="431" alt="2026-02-19_22-39" src="https://github.com/user-attachments/assets/768b5828-639e-465e-ac0d-b88f64470071" />

If the preset shows enabled, this indicates that the service will automatically start after a reboot.

Now, entering the server IP address in a web browser displays the default Nginx welcome page, which will be modified in the next step.

For this purpose, a new directory is created under /var/www/alternatives

`sudo mkdir /var/www/alternatives`

Because normal users do not have permission to make edits in this directory, the command "sudo" is used beforehand, which means SuperUserDo.

In this newly created directory, the text file alternate-index.html will now be created; this is achieved using the "touch" command.

`sudo touch /var/www/alternatives/alternate-index.html`

To configure the web server to load the newly defined alternative website, the following lines are added to /etc/nginx/sites-enabled/alternatives.

`sudo vim /etc/nginx/sites-enabled/alternatives`

<img width="667" height="295" alt="2026-02-23_19-36" src="https://github.com/user-attachments/assets/7abbadf3-374c-4600-9eca-1da9953d0b06" /> <br>

Of course, the alternate-index.html file also needs to be filled with HTML code. 

`sudo vim /var/www/alternatives/alternate-index.html`

<img width="999" height="397" alt="2026-02-23_19-43" src="https://github.com/user-attachments/assets/666f2dcd-0c98-4a02-bad5-c73d307f86ac" /> <br>

To ensure that the default nginx website is no longer displayed and users are directed to the alternative website, port 80 is disabled and a redirect to port 8081 is set up.

Comment out the following entry (listen):

`sudo vi /etc/nginx/sites-available/default`

<img width="676" height="153" alt="2026-02-24_18-10" src="https://github.com/user-attachments/assets/58234a56-d112-4aae-b224-f9099a751948" /> <br>

Create a new file and add the following server block 

`sudo vim /etc/nginx/sites-available/redirect_80.conf`

<img width="682" height="160" alt="2026-02-24_18-39" src="https://github.com/user-attachments/assets/d4d6b14e-927f-465a-8b45-bf1a73e43178" /> <br>


Configure and test nginx

`sudo nginx -t`       
`sudo systemctl restart nginx`

To verify, open a web browser and access the target IP address, in this case 167.235.27.211

<img width="689" height="166" alt="2026-02-24_18-54" src="https://github.com/user-attachments/assets/57863877-e987-4c64-ba1e-8d0349fea39e" /> <br>

## 5. Git Installation and Login

To install Git, use the command:

`sudo apt install git -y` .

The installation is then verified with `git --version`. The output should display the Git version.

<img width="456" height="58" alt="2026-02-23_18-59" src="https://github.com/user-attachments/assets/93eded8a-477b-49ce-a435-3a2c223b37d5" /> <br>

Now, the credentials need to be entered using the following commands:

`git config --global user.name "Your Name"`

`git config --global user.email "your.email@example.com"`

To check the configuration:

`git config --list`

Then Create a new local Branch in the home directory

`mkdir -p ~/Git/V-Server-Setup`

After that the repository needs to be initialize

`cd ~/Git/V-Server-Setup`
`git init`

A key pair is also created on the virtual server.

`ssh-keygen -t ed25519`

Copy the public key.

`cat ~/.ssh/github-key.pub`

Then add it to the GitHub web interface under SSH and GPG Keys settings.

Test if it works 

`ssh -T git@github.com`

Now its time to make a readme.md on the local machine and add it to the github remote host 

`touch README.md`
`git add README.md`
`git remote add origin git@github.com:"username"/"repo"`
`git push -u origin main`

### THE END 


   
