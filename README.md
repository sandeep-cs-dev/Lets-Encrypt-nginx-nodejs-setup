# Let's Encrypt

Examples of getting certificates from [Let's Encrypt](https://letsencrypt.org/) working on NGINX and Node.js servers.

## Obtain certificates
I am using the manual method, you have to make a file available to pass let's encryp acme-challenge. Follow the commands from running

# express server for acme challenge
```javascript
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
```shell
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto certonly --manual --email admin@example.com -d example.com
```

This creates a directory: `/etc/letsencrypt/live/example.com/` containing certificate files:

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
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
}
```
