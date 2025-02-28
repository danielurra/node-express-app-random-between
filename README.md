# Set up a simple Express server that listens on port 3044
![rest-api-node-plus-express_js-Dani](https://github.com/danielurra/node-express-app-random-between/assets/51704179/189b70c2-5122-406d-b6cf-95c23d80cfc9)<br>
[Introduction](https://github.com/danielurra/node-express-app-random-between#set-up-a-simple-express-server-that-listens-on-port-3044)<br>
* [Simple Express server](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#nodejs-express-random-between-app)<br>
  - [Random number App](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#nodejs-express-random-between-app)<br>
* [Project initialization](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#initialize-and-install-dependencies)<br>
  - [Package.json](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#packagejson)<br>
* [Server code](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#serverjs)<br>
  - [explanation](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#setting-up-a-route)<br>
  - [Export function](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#randomjs)<br>
* [Web server NginX - server block file](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#web-server-nginx-reverse-proxy)<br>
* [Run your app](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#run-the-server-app-good-for-dev-phase---permanent-run-use-pm2)<br>
* [Animated Gif](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#animated-gif)<br>
* [PM2](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#using-a-process-manager-recommended-for-development-and-production)<br>
* [Enhanced version](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#updated-code)<br>
* [Explanation of the changes](https://github.com/danielurra/node-express-app-random-between/blob/main/README.md#explanation-of-changes)<br>

## Node.js, express, random-between App
Node, express app, random number between 0 and your input, a number specifed in the route.<br> 
For example, http://localhost:3044/10 should return a random number between 1 and 10.<br>
![curl-random-echo](https://github.com/danielurra/node-express-app-random-between/assets/51704179/d21924c5-b4a6-462f-8739-1fe30f8f2181)<br>
When a user visits a URL like `http://localhost:3044/<some-number>`, the server generates a random number between 1 and `<some-number>` and sends it back as the response.<br>
The random number generation logic is encapsulated in the random function, which is **imported** from a separate file `(./random)`.<br>

## Initialize and install dependencies
* `express`: This is a popular **Node.js** framework used to create web servers and handle HTTP requests.
* `app`: The app variable is an **instance** of the Express application.
* `random`: This line imports a function from a **local file** named random.js.
```javascript
npm init --yes
npm install express
touch server.js random.js
```
## server.js 
The server listens on **port 3044** for incoming `HTTP GET requests`.
```javascript
const express = require("express");
const app = express();
const random = require("./random");

app.get("/:number", (req, res) => {
  const number = req.params.number;
  res.send(random(number).toString());
});

app.listen(3044, () => {
 console.log("Server listening on port 3044");
});
```
### Setting Up a Route
* `app.get("/:number", ...)`: This sets up a route for handling GET requests. The `:number` part is a route parameter, meaning that whatever value is passed after the / in the URL will be captured as number.
For example, if you visit http://localhost:3044/10, the number parameter will be "10".
* Inside the callback function:
`req.params.number`: This retrieves the value of the number parameter from the URL.
`random(number)`: Calls the imported random function with the number as an argument. We'll explain the random function shortly.
`res.send(...)`: Sends the result of `random(number)` back to the client as a response.<br>
* The `.toString()` converts the number to a string before sending it
## random.js 
This part of the code **exports a function** that generates a random number between 1 and n (inclusive).<br>
* `Math.random()`: Generates a random floating-point number between 0 (inclusive) and 1 (exclusive).<br>
* `Math.random() * n`: Scales the random number to be between 0 and n.<br>
* `Math.floor(...)`: Rounds the number down to the nearest integer.<br>
* `+ 1`: Ensures that the result is at least 1.<br>
```javascript
module.exports = (n) => {
  const randomNumber = Math.floor(Math.random() * n) + 1;
  return randomNumber;
};
```
## package.json
```javascript
{
  "name": "nest.comprones.com",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```
## Web server Nginx reverse proxy
Our VPS server is a **Ubuntu 22.10** running `nginx` as a web server and also as a **reverse proxy**.<br>
The SSL Certificates are provided by Let's Encrypt and installed using the **certbot** CLI tool.
```bash
server {
        root /var/www/nest.comprones.com;
        index index.html index.js;
        server_name nest.comprones.com www.nest.comprones.com;
        location / {
     try_files $uri @nodejs;
     #try_files $uri $uri/ =404;
        }
# The try_files directive takes a list of files and a location as the last argument.
# the @ modifier, defines a named location block
# the last option of the directive try_files is an internal redirect to the specified location in this case to Node.js reverse proxy

location @nodejs { 
 proxy_pass http://localhost:3044;
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection 'upgrade';
 proxy_set_header Host $host;
 proxy_cache_bypass $http_upgrade;
}

listen 443 ssl; # managed by Certbot
ssl_certificate /etc/letsencrypt/live/comprones.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/comprones.com/privkey.pem; # managed by Certbot
include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = www.nest.comprones.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = nest.comprones.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        server_name nest.comprones.com www.nest.comprones.com;
    return 404; # managed by Certbot
}
```
## Run the server App (good for dev phase - permanent run use PM2)
Finally, run the node server with the following command:<br>
```javascript
node server.js
```
`Output`:<br>
"Server listening on port 3044"<br>
## Animated GIF
![node-express-app-random-num](https://github.com/danielurra/node-express-app-random-between/assets/51704179/b77d713e-cc84-4579-8b90-e525ac0d48c4)<br>

## Using a Process Manager (Recommended for Development and Production)
Process managers like `PM2` or `Forever` are widely used to keep **Node.js** applications running in the background.<br>
They also provide features like automatic restarts, logging, and monitoring.<br>
### Using PM2
`PM2` is one of the most popular process managers for **Node.js** applications.<br>
* Use the Node Packet Manager `npm` to install PM2 globally :
  ```bash
  npm install -g pm2
  ```
* Start your app with PM2 (let's say it is "server.js"):
  ```bash
  pm2 start server.js
  ```
* Keep PM2 running on reboot :<br>
  To ensure your app starts automatically when the server reboots, run:
  ```bash
  pm2 startup
  ```
* Save the current PM2 process list :<br>
  This ensures that PM2 restores your app on reboot:
  ```bash
  pm2 save
  ```
* Monitor your app :<br>
  You can monitor your app's status with:
  ```bash
  pm2 list  
  ```
* View logs :<br>
  PM2 stores logs for your app, which you can view with:
  ```bash
  pm2 logs
  ```
* Restart a Specific Application :<br>
  If you want to restart a specific application managed by PM2, use the pm2 restart command followed by the application name or ID.:
  ```bash
  pm2 restart <app_name_or_id>
  ```
* Use the App ID number :<br>
  if you know the application's ID (you can find it using `pm2 list`), you can restart it by ID:
  ```bash
  pm2 restart 0
  ```
  ![2025-02-06_12-53-47](https://github.com/user-attachments/assets/fb05ddd9-ecaf-4f9e-b3cc-8be1ab8b4b0d)<br>

* Restart All Applications :<br>
  If you have multiple applications running under PM2 and want to restart all of them at once, use:
  ```bash
  pm2 restart all
  ```

## Updated code
This modification makes the server respond with an HTML page instead of plain text.<br>
The random number is displayed inside an **\<h1\>** tag along with the descriptive text **"Random number between 1 and n (inclusive):"**.<br>
This way, when users visit the URL, they see a nicely formatted webpage with the random number displayed prominently.<br>

```javascript
const express = require("express");
const app = express();
const random = require("./random");

app.get("/:number", (req, res) => {
  const number = req.params.number;
  
  // Generate the random number using the imported 'random' function
  const randomNumber = random(number);
  
  // Create an HTML response with the random number in an <h1> tag
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Random Number Generator</title>
    </head>
    <body>
      <h1>Random number between 1 and ${number} (inclusive): ${randomNumber}</h1>
    </body>
    </html>
  `);
});

app.listen(3044, () => {
  console.log("Server listening on port 3044");
});
```
## Explanation of Changes
* **HTML Response**:
Instead of sending just the random number as plain text, we now send a full HTML document.<br>
The HTML document contains an **\<h1\>** element that displays the text **Random number between 1 and n (inclusive)** followed by the generated random number.<br>
* **Template Literals**:
We use template literals (enclosed in `backticks` " \` ") to construct the HTML string. This allows us to embed variables like `${number}` and `${randomNumber}` directly into the string.<br>
* **HTML Structure**:
The HTML structure includes a `<!DOCTYPE html>` declaration, `<html>`, `<head>`, and `<body>` tags for proper formatting.<br>
The `<h1>` tag contains the desired text and the random number.<br>


