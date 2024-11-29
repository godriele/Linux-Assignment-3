# Linux-Assignment-3 Part 1

## Introduction:


- Welcome to my Linux Assignment #3 where I will be setting up a Bash script that will generate a static html file and it will contain some system information. The script I'll be making, wil be configured to run at 5 am using a **Systemd** `Service` and `Timer`.

- Inisde the **HTML** document, the script will be served with an `nginx` web server tht will run on a Arch Linux Droplet along with a firewall setup using `ufw` to help secure the server

---

# Task 1: Create a System Account User, Give permission, and Create a file directory 

### 1.  Git Repository 
- Before starting I made sure to clone the repository given because it contained the necessary file to start


- Clone this repository
```
https://git.sr.ht/~nathan_climbs/2420-as3-p2-start
```

### 2. Create the user 
- I made the user to be a system account because creating a system account user instead of a regular user can help limit access and permissions, which can help protect the system from security risks. System accounts are typically used for system service which will be needed for this script

```
sudo useradd -r -d /var/lib/webgen
```

This will create the user "webgen" as a system account(`-r`) and its home directory to be (`-d`) at /var/lib/webgen

Double check that that user you created is a system account
  -  After creating the user "webgen", a way to double check that it is a system account, ran:
   ```
   sudo cat /etc/passwd | grep webgen
   ```

  - This should output: the following
  ```
   webgen:x:999:1001::/var/lib/webgen:/usr/sbin/nologin
  ```
   **webgen** = user
   
   **UID** = 999 -> typical for system accounts 

   **GID** = 1001 -> Group ID

   **/var/lib/webgen** = Home Directory

   **/usr/sbin/nologin** = The default shell for system accounts 

### 3. Change Ownership 
- We need to give the user webgen and group webgen ownership (since its a system account a group will be automatically be createed named webgen)

```
sudo chown -R webgen:webgen /var/lib/webgen 
```
`-R` option recursively changes the ownership of all files

- Ref = https://askubuntu.com/questions/693418/use-chown-to-set-the-ownership-of-all-a-folders-subfolders-

### 4. Home Directory Structure
  We want want our File structure to look like
  
  ```
  /var/lib/webgen/ 
├── bin/ 
│ 	└── generate_index 
└── HTML/ 
	└── index.html
  ```
  - Make the directories
  ```
  sudo mkdir /var/lib/webgen/bin /var/lib/webgen/HTML
  ```
 I copied cloned the repository contetn inside /var/lib/webgen/bin, I then moved the generate_index one level up to be inside /bin, and deleted the cloned repo files

  - Make the generate_index excutable
  ```
  sudo chmod 700 /var/lib/webgen/bin/generate_index
  ```
   `700` will only grant the onwer full permissions and denys it to groups and others

  ---

  # Task 2: Service and Timer File 
  ### 1. Service File
  We would need to first create the service file called "generate-index.service"

  Since custom services are located in `/etc/systemd/system`, to create the file:

  ```
   sudo nvim /etc/systemd/system/generate-index.service
  ```
  
  I go more indepth on whats inside in the Service File in this repo
  
  ### 1.2 Reload, Enable, Start
  After creating the service file and adding in the necessary stuff inside run:
  ```
  sudo systemctl daemon-reload
  # Reload systemd to know that there was a new service file

  sudo systemctl enable generate-index.service
  # This will make the service start automatically on boot

  sudo systemctl start generate-index.service
  # Start the service
  ```
  ### 1.3 Check the status 
  ```
  sudo systemctl status generate-index.service
  # This will show if the service is active or inactive
  ```
  - To check the logs of you service run:
  ```
  journalctl -u generate-index.service
  # -u is used to filter logs by a specifif unit 
  ```

  ### 2. Timer file
  We need to create a timer file called "generate-index.timer"
  
  Same with the service file, timers are located in **/etc/systemd/system** to create the file:
  ```
   sudo nvim /etc/systemd/system/generate-index.timer
  ```
    - I go more indepth on wahts inside in the Timer File in this repo

  ### 2.1 Reload, Enable, Start
  After creating the timer file and adding in the necessary stuff inside run 
  ```
  sudo systemctl daemon-reload
  # Reload systemd to know there was a new timer file

  sudo systemctl enable generate-index.timer
  # Start the timer automatically on boot

  sudo systemctl start generate-index.timer
  # Start the timer
  ```
  ### 2.3 Check the status
  ```
   sudo systemctl status generate-index.timer
  ```

  Ref: 
  - Services = https://wiki.archlinux.org/title/Systemd
  - Timers = https://wiki.archlinux.org/title/Systemd/Timers

---

# Task 3: Nginx Configuration Modification 
  ### 1. Modify Nginx.configuration 
  We dont want to make our changes inside the main Nginx configuration because we may change some important settings. 
  We will use a separate server block file because it will help the configuration more organized and makes updating more easier 

 The Nginx Configuration file is located in **/etc/nginx/nginx.conf** 
 
 We can make our changes by running 
 ```
 sudo nvim /etc/nginx/nginx.conf
 ```

 The change: 
 ```
  user webgen; <---- change the default value to webgen 
  worker_processes 1 
 ```
 ### Personal Change
 I was having issues when loading my nginx web page, it showed the Welcome to your nginx page, but it wasnt showing my computers information. 

 Inisde the nginx configruation file, I had to add:
 ```
  http{
    types_hash_max_size 4096;
    # This makes the hash table for file types larger, so it can handle more types without running into issues
    types_has_bucket_size 64;
    # This will adjust how the hash tables is split up to make it more efficient 
  }
 ```

Im guessing I had to add ths because my nginx setup wasn't properly configured to handle certain file types. After researching this, I figured out taht Nginx uses somethign called a has table to match file types to their correct format.
So if this hash table isnt big enoguh or isn't set up properly, Nginx might not know how to handle certain files and will just show its default page

Ref = https://wiki.archlinux.org/title/Nginx

 ### 2. Create Server Blcock
  Since we dont want to modify the main nginc configuration file, We will create a separate one

  We need to make two directories in `/etc/nginx`
  
**- website**
  ```
   sudo mkdir /etc/nginx/website
  ```
 - Create a file called **server_block** in this directory 
  
**- sites-enabled**
  ```
   sudo mkdir /etc/nginx/sites-enables
  ```
 ### 3. Server Block
 I go more indepth in my Server Block file in this repo 
 After making the changes save and quit.

 ### 4. Enable our site, Symlink
 We have to modify our Nginx configuration file
 ```
  sudo nvim /etc/nginx/nginx,conf
 ```

 Inside the http function, we will have to add:
 ```
  http{
   types_hash_max_size 4096;
   types_hash_bucket_size 64;

   include /etc/nginx/sites-enabled <--- this line 
  }
 ```
 This will tell Nginx to look in the sites-enabled folder and load all the configuration files inside it

 - Now we can enable our site by running: 
```
sudo ln -s /etc/nginx/website/server_block /etc/nginx/sites-enabled/
```
 - To validate and test the changes we created we can run
```
sudo nginx -t 
```
- Lastly we have to have restart Nginx to enable everything
```
sudo systemctl restart nginx.service
```
Ref = 
- https://documentation.suse.com/smart/systems-management/html/systemd-management/index.html,
- https://wiki.archlinux.org/title/Nginx
  
---
# Task 4 Setting up ufw, ssh and our firewall
  ### 1. Install UFW 
  ```
  sudo pacman -S ufw
  ```
  - **ufw** will configure our firewall to set up rules to allow or block network traffic

  ### 2. Allow SSH and HTTP 
  ```
  sudo ufw allow ssh
  sudo ufw allow http 
  ```
  - We are allowing incoming SSH connections to remotely access the server
  - We are allowing incoming HTTP traffic to enable our web server to serve websites to the public
    
  ### 3. Enable SSH Rate Limiting 
  ```
  sudo ufw limit ssh
  ```
  - This will limits the nunmber of failed login attempts from a specific IP address within a short time period. Will help prevent brute-force attacks

  ### 4. Enable UFW Firewall 
  ```
   sudo ufw enable 
  ```
  - This will make sure that the rules we just made are applied to our system

  ### 5.Verify the Firewall 
  ```
   sudo ufw status verbose 
  ```
   - This will show the current firewall status, including which ports are allowed or denied 

# Task 5: Our Website!!!
### 1. Check our nginx
  We first have to check our nginx to see that everythign is working 
  ```
  sudo systemctl status nginx
  ```
### 2. Enable and Start Nginx (If not already enabled) 
  ```
   sudo systemctl enable nginx
   sudo systemctl start nginx 
  ```
### 3. Website
 To see your website vist -> http://<ip address> 
 
 but here is mine 

<img width="607" alt="Screenshot 2024-11-28 at 4 00 10 PM" src="https://github.com/user-attachments/assets/377ebe4f-61bf-4591-8fd7-6f196e189375">

 
