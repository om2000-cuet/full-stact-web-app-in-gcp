# full-stack-web-app-in-gcp
<img src="https://cdn-images-1.medium.com/max/1600/1*Vq-yTKUmoxLwGr2pOqK1Kw.png"/><br/><br/>
Image Courtesy: Poridhi Team, Umar F Chy<br/><br/>
Deploy a full-stack application consisting of a frontend and a backend application within a Virtual Private Cloud (VPC)
on Google Cloud Platform (GCP).<br/><br/>
• There should be one virtual machine (VM) for the frontend application and two VMs for the backend application.<br/><br/>
• Additionally, set up a front-facing load balancer that will serve as a reverse proxy for the system and distribute traffic evenly between the two backend VMs.<br/><br/>
• Lastly, you need to demonstrate fetching data from<br/><br/>

Exam 6<br/><br/>


At first created a VPC named cls-vpc-1 , also subnect named cls-subnet-1 was created under the region us-central-1 the ip v4 range we used is 192.168.0.0/24
created a VM cls-lb-1  which will act as a front end Load balancer which supports external IP the public IP which is created by default was 34.170.228.55
<br/><br/>
Created another VM cls-frontend for front end, where the initial front end webpages will reside. This VM dont have support for external IP
<br/><br/>
Configuring  Load Balancer VM and Front End VM
<br/><br/>
in LoadBalancer VM<br/><br/>
sudo apt update -y 
as 34.170.228.55 is not showing anything so we install Nginx under LoadBalancer VM<br/><br/>
sudo apt install nginx<br/><br/>
after installing nginx if we visit 34.170.228.55 it will show Welcome to Nginx <br/><br/>
to check if nginx status<br/><br/>
sudo systemctl status nginx <br/><br/>

Adding NAT Gateway to VPC<br/><br/>

As we didnt add public ip to front end VM, so we are adding  NAT Gateway  (cls-gw-1 ) to vpc using which front end VM will be able to exgress to outside world
<br/><br/>

Modifying the index file of Nginx server ( in LoadBalancer VM)<br/><br/>
cd /var/www/html/<br/><br/>
ls<br/><br/>
sudo vim index.nginx-debian.html<br/><br/>
modify the html and go to http://34.170.228.55 to see the changes otherwise use this code<br/><br/>
sudo nginx -s reload <br/><br/>

Configuring FrontEnd VM<br/><br/>

as CloudNat is added to VPC, so now FrontEnd VM can Exgress to outsideworld and we can do configuration to it<br/><br/>

sudo apt update -y <br/><br/>
sudo su

curl -fsSL https://deb.nodesource.com/setup_19.x | bash - &&\
apt-get install -y nodejs

exit

node -v

Check Yarn is installed or not<br/><br/>
yarn -v
if yarn is not installed use this code 
sudo corepack enable
yarn -v  (it will show yarn version)

installing Ract App using Vite<br/><br/>

yarn create vite
cd our-frontend
yarn
yarn run build
yarn preview
Ctrl+c (to exit preview)<br/><br/>
now we have to expose host ip and need to tell port

sudo yarn preview —host —port 80


Configuring Load Balancer VM<br/><br/>
sudo apt install -y telnet net-tools
telnet 192.168.0.3 80
(it will connect the ip as both are under same vpc) 
curl 192.168.0.3

Configuring Nginx server configuration file 
cd /etc/nginx/
ls
sudo vim nginx.conf

:%d (remove all lines in the config file)
:wq




events {
    # empty placeholder
}
http {
    server {
        listen 80;
        location / {
            proxy_pass http://frontend;
        }<br/><br/>
location /api/ {<br/><br/>
            rewrite ^/api/(.*)$ /$1 break;<br/><br/>
            proxy_pass http://backend;<br/><br/>
        }<br/><br/>
          }
<br/><br/>
    upstream frontend {<br/><br/>
        server  192.168.0.3:80;<br/><br/>
    }<br/><br/>
upstream backend{<br/><br/>
server 192.168.0.4;<br/><br/>
server 192.168.0.5;<br/><br/>
}
 <br/><br/>
}



to validate our configuration use this code<br/><br/>
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

Reload the server as we have changed configuration<br/><br/>
sudo nginx -s reload 

now you will see that, visiting http://34.170.228.55 will show the react app installed in FrontendVM<br/><br/>


Configuring Backends VM  ( create and Configuration) <br/><br/>
create  2 VM cls-be-1 and cls-be-2 for backend process purpose( both of them  dont need public IP) 


Configuring Backend 1 VM  <br/><br/>

sudo apt update -y
sudo su
curl -fsSL https://deb.nodesource.com/setup_19.x | bash - &&\
apt-get install -y nodejs
exit




then check if the node installed properly or not using<br/><br/>
node -v 
mkdir be1
cd be1

sudo corepack enable
npm init -y 

installing Express server<br/><br/>
yarn add  express


vim index.js

const express = require('express');

// Create an instance of Express
const app = express();

// Define a route for the root URL
app.get('/', (req, res) => {
  res.send('Hello, World from Backend 1!’);
});

// Start the server
const PORT = process.env.PORT || 80;
app.listen(PORT, () => {
  console.log(‘Server is running on port ${PORT}’ +PORT);
});

sudo node index.js

to check it is working or not, from Backend 1 VM<br/><br/>
curl 192.168.0.4

Configuring Backend 2 VM  <br/><br/>


sudo apt update -y
sudo su
curl -fsSL https://deb.nodesource.com/setup_19.x | bash - &&\
apt-get install -y nodejs
exit


then check if the node installed properly or not using<br/><br/>
node -v 
mkdir be2
cd be2
sudo corepack enable
npm init -y 

installing Express server
yarn add express



vim index.js

const express = require('express');

// Create an instance of Express
const app = express();

// Define a route for the root URL
app.get('/', (req, res) => {
  res.send('Hello, World from Backend 2!’);
});

// Start the server
const PORT = process.env.PORT || 80;
app.listen(PORT, () => {
  console.log(‘Server is running on port ${PORT}’ +PORT);
});

sudo node index.js

to check it is working or not, from Backend 1 VM<br/><br/>
curl 192.168.0.5
 
Configuring  Load Balancer again <br/><br/>


Configuring Nginx server configuration file <br/><br/>
cd /etc/nginx/
ls
sudo vim nginx.conf

:%d (remove all lines in the config file)
:wq


events {
    # empty placeholder
}
http {
    server {
        listen 80;
location / {<br/><br/>
            proxy_pass http://frontend;<br/><br/>
        }<br/><br/>
location /api/ {<br/><br/>
            rewrite ^/api/(.*)$ /$1 break;<br/><br/>
            proxy_pass http://backend;<br/><br/>
        }
         <br/><br/>
    }
<br/><br/>
    upstream frontend {<br/><br/>
        server  192.168.0.3:80;<br/><br/>
    }
upstream backend{<br/><br/>
server 192.168.0.4;<br/><br/>
server 192.168.0.5;<br/><br/>
}
 <br/><br/>
}



to validate our configuration use this code<br/><br/>
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

Reload the server as we have changed configuration<br/><br/>
sudo nginx -s reload 

