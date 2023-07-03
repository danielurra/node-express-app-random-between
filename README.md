# Node.js, express, random-between App
Node, express app, random number between 0 and your input, a number specifed in the route.<br> 
For example, http://localhost:3044/10 should return a random number between 1 and 10.<br>
![curl-random-echo](https://github.com/danielurra/node-express-app-random-between/assets/51704179/d21924c5-b4a6-462f-8739-1fe30f8f2181)

## Initialize and install dependencies

```javascript
npm init --yes
npm install express
touch server.js random.js
```
## Copy the following code inside server.js 
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
## Copy the following code inside random.js 
```javascript
module.exports = (n) => {
  const randomNumber = Math.floor(Math.random() * n) + 1;
  return randomNumber;
};
```
