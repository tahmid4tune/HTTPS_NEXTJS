# HTTPS_NEXTJS

## What this is about?
This repository contains the set of instructions to run a NextJS application on Ubuntu/Linux PC's localhost using HTTPS protocol instead of HTTP. 

## Why is this important?
With growing popularity of HTTPS (for obvious reasons) there are different components nowadays which do not support HTTP protocol anymore. For instance, (mobile browser) google map's current location tracking system is not supported when the application runs on HTTP protocol.

## Steps to run NextJS application on HTTPS
### Generate certificate for localhost
- Execute following command from command prompt:
```sh
openssl req -x509 -out localhost.crt -keyout localhost.key \
  -days 365 \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=localhost' -extensions EXT -config <( \
   printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```

- After execution of the command you should be able to locate following 2 files in your home directory: 
1. localhost.crt
2. localhost.key

### Add generated certificate as trusted
1. Copy the certificate to directory: /usr/local/share/ca-certificates/ using following command:
```sh
sudo cp localhost.crt /usr/local/share/ca-certificates/localhost.crt
```
2. Update CA store using following command:
```sh
sudo update-ca-certificates
```
### Project configuration
- Create server configuration file named `server.js` in the root directory of your project. 
- Paste following content in `server.js` file:

```sh
const { createServer } = require("https");
const { parse } = require("url");
const next = require("next");
const fs = require("fs");
const dev = process.env.NODE_ENV !== "production";
const app = next({ dev });
const handle = app.getRequestHandler();
const httpsOptions = {
  key: fs.readFileSync("./certificates/localhost.key"),
  cert: fs.readFileSync("./certificates/localhost.crt"),
};
app.prepare().then(() => {
  createServer(httpsOptions, (req, res) => {
    const parsedUrl = parse(req.url, true);
    handle(req, res, parsedUrl);
  }).listen(3000, (err) => {
    if (err) throw err;
    console.log("> Server started on https://localhost:3000");
  });
});
```
- This `server.js` file looks for `localhost.key` and `localhost.crt` files inside of certificates directory. Hence, create a directory named certificates in root directory of the folder and paste `localhost.key` and `localhost.crt` files there.
- Update the `.gitignore` file as well in order to avoid putting our own localhost setup related changes to be pushed into the main repository. Add following lines in `.gitignore` file

```sh
#Localhost HTTPS setup config
/certificates
server.js
```
- In order to run the project using this `server.js` configuration defined above, we need to update the `scripts` section of the `package.json` file. Add following script there.

```sh
"scripts": {
  ...
  "secure-dev": "node server.js",
  ...
}
```
### Test the whole setup
- Open command prompt in the root directory of the project and execute following command:

```sh
npm run secure-dev
```

or for yarn users

```sh
yarn secure-dev
```

## References
- https://medium.com/@greg.farrow1/nextjs-https-for-a-local-dev-server-98bb441eabd7
- https://support.kerioconnect.gfi.com/hc/en-us/articles/360015200119-Adding-Trusted-Root-Certificates-to-the-Server
