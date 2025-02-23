# NodeJs-Deployment on AWS with EC2 instance, Nginx as reverse proxy and SSL certificate.

## > Steps to deploy a Node.js app to AWS using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Create Free AWS Account
Create free AWS Account at https://aws.amazon.com/

## 2. Create and Lauch an EC2 instance and SSH into machine
Choose one of the instance type as per your need - (for free-tier choose t2.micro instance type).

## 3. Associate Elastic IP with the instance
Instance created does not have fixed public IP. To associate the instance with a fixed public IP we use elastic IP. Connect your domain name with the generated elastic IP.  

## 4. Connect with the instance created using AWS server or using command prompt using key-pair. Steps to connect to the instance using command prompt
```
# Run the following command to change the directory with key-pair file exists
cd C:\Users\YourUsername

#Connect to EC2 Instance
ssh -i "your-key.pem" ubuntu@your-instance-public-ip
```

## 5. install Node and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs

node --version
```

## 6. Clone your project from Github and install dependencies
```
git clone https://github.com/username/project_name
cd project_name
npm install
```
## 7. Add environment variables
```
sudo nano .env
# paste .env file as it is.
```

## 8. Install pm2 (process manager) and test app
```
# A process manager is used to manage applications by handling tasks such as starting, stopping, restarting, and monitoring logs.
sudo npm i pm2 -g
pm2 start src/index.js

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream bydefault shows last 15 lines of logs)
pm2 logs --lines 100 (shows last 100 lines of logs)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```

## 9. Install NGINX and configure
```

# nginx is a reverse proxy which allow us to directly use the domain name or public IP to access the service without explicitly mentioning the port number.
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com

    location / {
        proxy_pass http://localhost:8000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo nginx -s reload
```

## 10. Add SSL with LetsEncrypt
```
# An SSL certificate enables a secure connection for a website by encrypting data and allowing it to be accessed over HTTPS
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
sudo certbot renew --dry-run
```
