# Let's Encrypt

Examples of getting certificates from [Let's Encrypt](https://letsencrypt.org/) while working on NGINX and Node.js servers.

## Obtain certificates
I am using the manual method. You have to make a file available through express to pass let's encrypt acme-challenge. 

# express server for acme challenge
```javascript
var express = require("express");
var app = express();
var path = require("path");
var fs  =  require("fs");
var server = require('http').createServer(options,app)

app.get('/.well-known/acme-challenge/{path-key}'
, function(req, res) {
    res.sendFile(path.join(__dirname + '/path_to_key_file'));
});
server.listen(3000,function(err){
 if(!err) {
 	console.log("server listening oat port 3000");
 }
 else {
 	console.log("something went wrong",err);
 }
});

```
# Running Let's encrypt script
```shell
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto certonly --manual --email admin@example.com -d example.com
```
 Before pressing key to continue, replace actual path key and and file with challenge key in express server.
 Make sure your express server is accessible from the domain for which you want to optain the certificate.

This creates a directory: `/etc/letsencrypt/live/example.com/` containing below certificate files:

- cert.pem
- chain.pem
- fullchain.pem
- privkey.pem

## express js (https server)

```javascript
var express = require('express');
var app = express();
var path = require("path");
var fs  =  require("fs");

var options = {
  key: fs.readFileSync('/etc/letsencrypt/live/example.com/privkey.pem'),
  cert: fs.readFileSync('/etc/letsencrypt/live/example.com/cert.pem'),
  ca: fs.readFileSync('/etc/letsencrypt/live/example.com/chain.pem')
};

var server = require('https').createServer(options,app);
//app.use(express.static(path.join(__dirname, 'public')));
server.listen(3000,function(err){
 if(!err) {
 	console.log("server listening oat port 3000");
 }
 else {
 	console.log("something went wrong",err);
 }
});

```

## NGINX
```Nginx
server {
    listen 8080;
    listen 443 default ssl;
    server_name example.com;
    ssl_certificate      /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/example.com/privkey.pem;
    
    location / {
        proxy_pass  https://localhost:3000;
        proxy_set_header   Connection "";
        proxy_http_version 1.1;
        proxy_set_header        Host            $host;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    }

gzip on;
gzip_comp_level 4;
gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;


location /public {
    alias /var/app/current/public;
}

}


```
