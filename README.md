# Let's Encrypt

Examples of getting certificates from [Let's Encrypt](https://letsencrypt.org/) working on NGINX and Node.js servers.

## Obtain certificates
I am using the manual method, you have to make a file available to pass let's encryp acme-challenge. Follow the commands from running

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

## Node.js
```javascript
var https = require('https');
var fs = require('fs');

var options = {
  key: fs.readFileSync('/etc/letsencrypt/live/example.com/privkey.pem'),
  cert: fs.readFileSync('/etc/letsencrypt/live/example.com/cert.pem'),
  ca: fs.readFileSync('/etc/letsencrypt/live/example.com/chain.pem')
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
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
