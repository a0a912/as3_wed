# Linux a3p2 Adding a Firewall

Step 1: Check if ufw is installed and if ssh is allowed. This is important as you don’t want to be locked out your Instance

### 1a: Update existing repositories with pacman -Syu

```bash
sudo pacman -Syu
```

### 1b: Install ufw with pacman:

```bash
sudo pacman -S ufw
```

### 1c: Check what ufw is currently doing:

```bash
sudo ufw status #see what Ufw is currently doing with the firewall
sudo ufw show added #See what rules ufw are following
```

It should say inactive and no rules because we haven’t enabled it yet. DON’T ENABLE before allowing ssh otherwise you will be locked out:

```bash
sudo ufw allow ssh
```

### 1d: We also need to enable http and port 8080 since we will be using that for the assignment:

```bash
sudo ufw allow http
sudo ufw allow 8080
```

Check with show added:

```bash
sudo ufw show added
```

You should get the following output confirming ssh is allowed:

```bash
Added user rules (see 'ufw status' for running firewall):
ufw allow 22
ufw allow 80
ufw allow 8080
```

### 1e: Now we can safely enable ufw:

```bash
sudo ufw enable
```

Press y to enable it.

use status to check what ufw is doing now:

```bash
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
8080                       ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
8080 (v6)                  ALLOW       Anywhere (v6)
```

Part 2: Server File work

The next task is to use the hello-server file the prof gave us. The file is in the prof’s notes under attatchments. At this link specifically: [https://gitlab.com/cit2420/2420_notes_w24/-/blob/main/attachments/hello-server?ref_type=heads](https://gitlab.com/cit2420/2420_notes_w24/-/blob/main/attachments/hello-server?ref_type=heads) 

### 2a: Download this file and make sure you note where you download it. Then make sure you open powershell in that exact location (or cd to it)

### 2b: With your powershell in that directory, use sftp to connect to your droplet and upload the file using the put command through sftp (Secure File Transfer Protocol). Here is my powershell so far with comments:

```bash
 cd "C:\Users\waliz\OneDrive\Documents\Uni Stuff\BCIT\Year 1\Term 2\Linux ACIT 2420\as3p2"
 as3p2  ll #cd to where I downloaded the hello-server file and check if the file is even there

    Directory: C:\Users\waliz\OneDrive\Documents\Uni Stuff\BCIT\Year 1\Term 2\Linux ACIT 2420\as3p2

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2024-04-10   9:48 AM       12310304   hello-server

 as3p2  sftp linux #Connect to your droplet via sftp 
Connected to linux.

sftp> put hello-server #uploads to my home server
Uploading hello-server to /home/ahmed/hello-server #File Uploaded successfully
```

### 2c: Move the file to the correct location:

```bash
sudo mv hello-server /usr/local/bin/backend
```

### 2d: Make the file executatble:

```bash
sudo chmod u+x /usr/local/bin/backend
```

## 3: Setting up Service File

### 3a: Vim into the Service for the backend file so that it can properly run in the background:

```bash
sudo vim /etc/systemd/system/backend.service
```

Copy and paste the following into the file:

```bash
[Unit] #This gives info on the service
Description=Backend Service #Name of the service
After=network.target

[Service]
Type=Simple
ExecStart=/usr/local/bin/backend #Location of the File
Restart=always #Reload after changes

[Install]
WantedBy=multi-user.target #Works for all users
```

### 3b: Start and enable the backend:

```bash
archlinuxx86-64cloudimg20240103204422qcow2-s-1vcpu-1gb-sfo3-01 /usr/local/bin Γ₧£
sudo systemctl start backend
sudo systemctl enable backend
Created symlink /etc/systemd/system/multi-user.target.wants/backend.service ΓåÆ /etc/systemd/system/backend.service.
```

## Step 4: Reverse Proxy

We will now add a reverse proxy to the backend

### 4a: Open the configuration file you made in sites-available last assignment:

```bash
sudo vim /etc/nginx/sites-available/nginx-2420
```

### 4b: We are going to make the following additions to the file. We are going to add these lines to the server block to make the 2 new proxies work:

```bash
location /hey {
    proxy_pass http://127.0.0.1:8080;
}

location /echo {
    proxy_pass http://127.0.0.1:8080;
}
```

Your file should look like this now:

```bash
server {
    listen 80;
    listen [::]:80;
    server_name _;
    root /web/html/nginx-2420;
    location / {
        index index.html index.htm;
    }
location /hey {
    proxy_pass http://127.0.0.1:8080;
}

location /echo {
    proxy_pass http://127.0.0.1:8080;
}
}
```

### 4c: Check if the syntax is correct with sudo nginx -t:

```bash
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
If all goes well, use postman to test if they work. You might need to reload ufw and/or reboot your instance one more time to ensure it works. Here is a screenshot of my postman demonstrating it.

![Untitled](Linux%20a3p2%20Adding%20a%20Firewall%20504d40b7970e46a18152f23f726af00c/Untitled.png)
