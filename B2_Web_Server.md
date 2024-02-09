# B2 Web Server

## 1. Apache2

> Apache2 allows for quick and easy configuration of a web server on Linux machines, while being extensible with many easy to use modules. It is a quick first step, if you need to host a website on your own computer for example, but is also usable in a production environment.
>
> Install Apache2 on lab2. The modules used later for serving user directory contents, rewriting URLs and setting up SSL should come with Apache2 by default. Set up SSH port forwarding for HTTP and HTTPS so that you can test the server on your local machine (localhost) with your favourite web browser. Note that VirtualBox port forwarding is not the way to do this! Instead, look into the ssh(1) manual page and use an ssh command to link the ports on the virtual machines to ports on the host.

### 1.1 Serve a default web page using Apache2 1p

On lab2

```shell
sudo apt update 
sudo apt install apache2 

curl localhost 
```

### 1.2 Show that the web page can be loaded on local browser (your machine or Niksula) using SSH port forwarding. 1p

On lab2

```shell
exit
# port forwarding
# 8888 is host port
# 80 is http port on guest
# 2200 is host ssh port
ssh -L 8888:localhost:80 vagrant@localhost -p 2200
```

In broswer

```shell
localhost:8888 
```

## 2. Serve a web page using Node.js

> Node is a cross platform browser-free JavaScript runtime environment, which can also be used as a HTTP server, when configured with the appropriate modules. The purpose of this assignment is to familiarize yourself with the increasingly popular and simple method of serving web applications using Node.js. Although javascript skills are not really needed in this assignment, we strongly recommend that you take a deeper look at javascript and also node.js
>
> Install nodejs on lab3 from package manager and create an HTTP application helloworld.js listening on port 8080 that serves a web page with the text "Hello world!".
>
> The web pages served by Node.js are written in javascript, but you do not actually need to know how to write it, because there's plenty of hello world examples on the internet. You do need to understand the contents.

### 2.1 Provide a working web page with the text "Hello World!" 2p

On lab3

```shell
mkdir test-server 
touch helloworld.js 
```

In `helloworld.js`

```javascript
const http = require("http");

const hostname = '0.0.0.0';
const port = 8080;

const server = http.createServer((req, res)=>{
        res.statusCode = 200;
        res.setHeader("Content-Type", "text/html");
        res.end("<h1>Hello, World!</h1>\n");

});

server.listen(port, hostname, ()=>{
        console.log('Server is running at http://${hostname}:${port}/');
});

```

### 2.2 Explain the contents of the helloworld.js javascript file. 1p

Just explain.

### 2.3 What does it mean that Node.js is event driven? What are the advantages in such an approach? 2p

In this paradigm, the program responds to events as they occur, rather than executing code in a linear manner.
The basic components of an Event-Driven Program are:

- A callback function ( called an event handler) is called when an event is triggered.
- An event loop that listens for event triggers and calls the corresponding event handler for that event.

Advantages of Event-Driven Programming:

- **Flexibility:** It is easier to alter sections of code as and when required.
- **Suitability for graphical interfaces:** It allows the user to select tools (like radio buttons etc.) directly from the toolbar
- Programming simplicity: It supports predictive coding, which improves the programmer’s coding experience.
- **Easy to find natural dividing lines:** Natural dividing lines for unit testing infrastructure are easy to come by.
- A good way to model systems: Useful method for modeling systems that must be asynchronous and reactive.
- **Allows for more interactive programs:** It enables more interactive programming. Event-driven programming is used in almost all recent GUI apps.
- Using hardware interrupts: It can be accomplished via hardware interrupts, lowering the computer’s power consumption.
- Allows sensors and other hardware: It makes it simple for sensors and other hardware to communicate with software.

## 3. Configuring SSL for Apache2

> This next part introduces you to SSL certificates by configuring your Apache server with one that you create.
> On lab2, use the Apache ssl module to enable https connections to the server. Create a 2048-bit RSA-key for the Apache2 server. Then create a certificate that matches to the key. The suggested method for achieving these two steps is using openssl. Configure Apache2 to use this certificate for HTTPS traffic. Set up again SSH port forwarding for the HTTPS port to test the secure connection using your local browser (if it is not active already).
> Note: Taking a shortcut with CA.pl is not accepted, you need to understand the process! Only a few commands are needed, though. Both the key and certificate can be created simultaneously using a single shell command.

### 3.1 Provide and explain your solution. 1p

On lab2

1. Create private key

    ```shell
    openssl genpkey - algorithm [algorithm name] -out [keyfile name] -pkeyopt rsa_keygen-bits: [bit count] 

    openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
    ```

2. extract public key from private key

    ```shell
    openssl rsa -pubout -in [private key file] -out [public key file]

    openssl rsa -pubout -in private_key.pem -out public_key.pem
    ```

3. Create Certificate Signing Request (CSR)

    ```shell
    openssl req -new -key [privage key file] -out [csr file] 

    openssl req -new -key private_key.pem -out csr.pem 
    ```

4. Request Self-signed Certificate

    ```shell
    open x509 -req -days [expire time] -in [csr file] -signkey [private key file] -out [file name] 

    openssl x509 -req -days 365 -in csr.pem -signkey private_key.pem -out certificate.pem 
    ```

5. enable apache2 ssl mod

    ```shell
    sudo a2enmod ssl 
    ```

6. Modify default-ssl.conf
In `/etc/apache2/sites-available/default-ssl.conf`

    ```default-ssl.conf
    SSLEngine on  
    SSLCertificateFile /path/to/your/certificate.pem  
    SSLCertificateKeyFile /path/to/your/private_key.pem 
    ```

7. Open browser and enter the HTTPS address

### 3.2 What information can a certificate include? What is necessary for it to work in the context of a web server? 1p

#### Information included in a certificate

**Subject:** The entity (person, organization, device, or website domain) the certificate represents.
**Issuer:** The Certificate Authority (CA) that issued the certificate. The issuer's signature verifies the certificate's authenticity.
Validity Period: The start date and expiry date, which define the timeframe during which the certificate is considered valid.
**Public Key:** The public key corresponding to the private key owned by the subject. The public key is used by clients to encrypt data that can only be decrypted by the corresponding private key.
Signature Algorithm: The algorithm used by the CA to sign the certificate, ensuring its integrity.
**Serial Number:** A unique identifier for the certificate issued by the CA.
Subject Alternative Names (SANs): Additional domain names or IP addresses covered by the certificate.
**Extensions:** Additional fields that provide various functionalities and constraints, such as key usage (digital signatures, key encipherment, etc.), enhanced key usage, and constraints on the certificate's use.

#### What is necessary

- The server must have a private key that corresponds to the public key in the certificate.
- The certificate must be properly installed and configured on the web server. This involves configuring the web server software (such as Apache or Nginx) to use the certificate and private key for SSL/TLS connections.
- The server should also provide the intermediate certificates that link the server's certificate to a trusted root CA certificate.
- The server typically listens on port 443 for HTTPS connections, as opposed to port 80 for HTTP.

### 3.3 What do PKI and requesting a certificate mean? 1p

PKI (Public Key Infrastructure): PKI is a framework used to create, manage, distribute, use, store, and revoke digital certificates. It is based on public key cryptography, where keys come in pairs (public and private).

## 4. Enforcing HTTPS

> Next, you will enforce the use of an SSL encrypted connection for one page on your server, while still allowing http on another. You will also learn to serve a directory from the user's home directory.
> On lab2, create a “public_html” directory and subdirectory called "secure_secrets" in your user’s home directory. Use the Apache userdir module to serve public_html from users' home directories.
> Enforce access to the secure_secrets subdirectory with HTTPS by using rewrite module and .htaccess file, so that Apache2 forwards "<http://localhost/~user/secure_secrets>" to "<https://localhost/~user/secure_secrets>". Please note that this is a bit more complicated to test with the ssh forwarding, so you can test it locally with lynx or netcat at the virtual machine. If your demo requires, you may hard-code your port numbers to the forwarding rules.

### 4.1 Provide and explain your solution. 2p

On lab2

1. create directories and index files

    ```shell
    mkdir public_html 

    cd public_html 

    touch index.html #(echo “<html><h1>hello world</h1></html>” >> index.html) 

    mkdir secure_secrets 

    touch index.html #(echo “<html><h1>You’ve found the secret!</h1></html>”) 
    ```

2. enable userdir mod

    ```shell
    sudo a2enmod userdir 
    ```

3. modify folder and file permission level

    ```shell
    sudo chmod /path 755 
    ```

4. create directories and index files

    Every component of the path should be chmod 755.

5. create directories and index files

    Then it should be accessible to <http://localhost/~vagrant/> & <http://localhost/~vagrant/secure_secrets/>

6. set up enforcement access to the secure_secrets subdirectory with HTTPS

    ```shell
    sudo a2enmod rewrite

    cd ~/public_html

    nano .htaccess
    ```

7. write in .htaccess

    ```config
    RewriteEngine On 
    RewriteCond %{HTTPS} off 
    RewriteCond %{REQUEST_URI} secure_secrets 
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301] 
    ```

Go to browser or curl to send a GET request to <http://localhost/~vagrant/secure_secret/>, it would shoe 301 moved permanently to https site.

### 4.2 What is HSTS? 1p

HTTP Strict Transport Security (HSTS) is a simple and widely supported standard to protect visitors by ensuring that their browsers always connect to a website over HTTPS.

### 4.3 When to use .htaccess? In contrast, when not to use it? 1p

The server must execute the .htaccess file to use it. This means that the server will need to use extra RAM, CPU power, and computing time to process a .htaccess file. This means each request that runs through a .htaccess file will take more resources and time.
Only use it when the main configuration file is not accessible

## 5. Install nginx as a reverse proxy

> Nginx is a third commonly used way of serving webpages, and also allows functioning as a proxy. Next, you are going to serve both Apache2 and Node.js hello world from lab1 using nginx as a reverse proxy.
> Install nginx on lab1 and configure it to act as a gateway to both Apache2 at lab2 and Node.js at lab3 the following way:
> HTTP requests to <http://lab1/apache> are directed to Apache2 server listening on lab2:80 and requests to <http://lab1/node> to Node.js server on lab3:8080.

### 5.1 Provide a working solution serving both web applications from nginx. 2p

1. Install Nginx

    ```shell
    sudo apt update
    sudp apt install nginx
    ```

2. Set up proxy

    ```shell
    sudo nano /etc/nginx/sites-available/lab_proxy
    ```

3. In lab_proxy file

    ```conf
    server {
        listen 80;
        server_name lab1;

        location /node {
                proxy_pass http://192.168.20.11:8080/;
        }

        location /apache {
                proxy_pass http://192.168.10.11/;
        }

    }
    ```

4. Create symbolic link in /etc/nginx/sites_enabled/

    ```shell
    sudo ln -s /etc/nginx/sites-available/lab2-proxy /etc/nginx/sites-enabled/
    ```

5. Check nginx configuration correctness

    ```shell
        sudo nginx -t
    ```

6. Reload nginx

    ```shell
    sudo systemctl reload nginx
    ```

7. Testing

    ```shell
    curl http://lab1/apache/
    curl http://lab1/node/
    ```

### 5.2 Explain the contents of the nginx configuration file. 1p

Just Explain

### 5.3 What is commonly the primary purpose of an nginx server and why? 1p

NGINX stands out as an efficient load balancer, adept at distributing incoming web traffic across multiple servers.

Implementing NGINX as a reverse proxy protects back-end servers from direct exposure to the internet.

## 6. Test Damn Vulnerable Web Application

> For security purposes, security professionals and penetration testers set up a Damn Vulnerable Web Application to practice some of the most common vulnerabilities. To achieve this goal, you can download the file in <https://github.com/digininja/DVWA/archive/master.zip> and install it on lab2. Video link: <https://youtu.be/WkyDxNJkgQ4> for guidance.
> You can delete the default Apache index.html and install the web application to be served as the default webpage. Finally, install Nikto tool which is an open-source web server scanner on lab 1 and scan your vulnerable web application.

### 6.1 Using Nmap, enumerate the lab2, and detect the os version, php version, apache version and open ports 2p

On lab1

```shell
nmap 192.168.10.11
```

Service version detection

```shell
nmap -sV 192.168.10.11
```

OS detection (sudo level)

```shell
sudo nmap -O 192.168.10.11
```

Agressive Scan (sudo level)

```shell
sudo nmap -A 192.168.10.11
```

### 6.2 Using Nikto, to detect vulnerabilities on lab2 2p

On lab1

```shell
nikto http://lab1/apache
```
