# HNG13 stage0 devops task

**Name**: Emmanuel Karingi Kariithi

**Slack Username**: @Kariithi

**Domain/Address**: http://emmanuelkariithi.com/

## Project Description
This project was completed as part of the [HNG 13 Internship](https://hng.tech/internship) – DevOps Track (Stage 0). It involves deploying a web server using NGINX on an AWS EC2 instance to host a static website. The configuration serves a custom `index.html` file from `/var/www/html` on the default HTTP port (`80`). A custom domain managed in AWS Route 53 is mapped to the instance for public access.

The project demonstrates key DevOps and cloud fundamentals — provisioning cloud infrastructure, configuring a web server, managing DNS, and deploying a live web application accessible over the internet.
## How to deploy a web server in AWS

### Requirements:
- Deploy a server using NGINX.
- Place index file at: `/var/www/html/index.html`
- Run on default HTTP port `(80)`
- Display custom content when accessed via browser.

### Deployment rules:
- Only port `80` allowed (no :`8080`, :`3000`, etc.)

### 1. Launch an Ubuntu EC2 instance + open HTTP
1. On AWS Console, go to EC2 → Instances → Launch instances.
2. Choose a name for the instance e.g. `HNG13-stage0`
3. Choose an AMI Ubuntu Server 24.04 LTS (That is free tier eligible).
4. Choose an Instance Type: t3.micro (free-tier eligible).
5. Create a Key pair and download the `.pem` so as to SSH into the instance later.

**Network settings / Security group**:
6. Create a security group that allows:
    - SSH (TCP 22) with Source: `your IP address` for security purposes.
    - HTTP (TCP 80) with Source: `0.0.0.0/0` (Anywhere) so the web is reachable.
7. Click on `Launch Instance`
8. Allocate and associate an Elastic IP to the instance so the public IP won’t change.

### 2. Connect to the EC2 instance via SSH
1. Open your terminal and navigate to where you stored the `.pem` key.

    `cd Downloads`

2. Change the file permissions so it’s secure to prevent SSH errors

    `chmod 400 HNG13-KEY.pem`

3. Connect to the instance

    `ssh -i your-key-name.pem ubuntu@<your-public-ip>` e.g. `ssh -i HNG13-KEY.pem ubuntu@44.177.34.183`

**You’ll know you’re connected when your terminal prompt changes to something like:**

    `ubuntu@ip-172-xx-xx-xxx:~$`

### 3. Install and start NGINX
1. In the same terminal, update the package lists:

    `sudo apt update`
2. Install NGINX:

    `sudo apt install nginx -y`

3. Start and enable NGINX:

    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
4. Check that it’s running:

    `systemctl status nginx`

5. Test in your browser:
    - Open your EC2 public IP (or Elastic IP) in a browser and you should see the default NGINX welcome page.

### 4. Upload custom `index.html` to the server
We can upload the custom `index.html` file via `scp` or edit the index file directly in the server.

**For SCP**:
1. If you have already opened VS Code and in the same directory where the HTML file is, open the terminal in VS Code and run:

    `scp -i /path/to/key-name.pem index.html ubuntu@<your-public-ip>:/tmp/`

To copy the local `index.html` file to the `/tmp/` folder on the server.

2. In your server terminal, run the following commands to move the `index.html` file into place.

    ```bash
    sudo mv /tmp/index.html /var/www/html/index.html
    sudo chmod 644 /var/www/html/index.html
    sudo chown www-data:www-data /var/www/html/index.html
    ```

**For directly editing the `index.html` file in the server**

1. In the terminal connected to the server, run:

    `sudo nano /var/www/html/index.html`

Then paste your HTML, save with `CTRL+O`, then `ENTER`, and exit with `CTRL+X`.

**Visit `http://<your-public-ip>` again, you should see your custom webpage content instead of the default NGINX page.**

### 5. Point your domain to your EC2 instance
I am using a domain that I had acquired from AWS (Route 53) so that's what thi guide will focus on.

1. In the AWS Console, go to `Route 53` → `Hosted zones`.
2. Click on your domain name.
3. Click Create record.
4. Choose:
    - **Record name**: leave empty for the root domain (or enter `www` for a subdomain).
    - **Record type**: A – IPv4 address.
    - **Value**: enter your instance’s Elastic IP address.
    - **TTL**: keep the default (e.g., 300 seconds).

5. Click Create records.

**If you haven’t yet assigned an Elastic IP, do that now:**
    - Go to EC2 → Elastic IPs → Allocate Elastic IP
    - Associate it with your running instance
    - Then use that Elastic IP in the A record above


6. Then test in your browser: It may take a few minutes for DNS changes to propagate. Visit your custom domain in your browser and you should see your same custom webpage, now accessible through your domain.

