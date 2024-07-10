---
layout: post
title:  "Create a professional web service - Part I - Server skeleton"
author: Jordi Dalmau Heras
date:   2023-07-06 09:58:12 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

Creating a professional web service has become more than just a technological endeavor—it's an art. Enter Node.js and Express.js, the dynamic duo that empowers developers to sculpt robust and performant web services while harnessing the prowess of the Linux operating system for hosting. If you're eager to learn the ropes of building and deploying a top-tier web service, you're in the right place.

This blog post embarks on a journey through the intricacies of crafting a professional web service using Node.js and Express.js, with a focus on deploying it on a Linux web server.

## Build a simple web server

Let's start with installing Node.js, npm, and Express on Ubuntu 22.04 LTS.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source ~/.bashrc

nvm install v18.14.2
nvm list

# Check versions
node -v
npm -v
```

First, we will create a very simple web service that listens for requests and responds with a message. All the development will be done using Node.js.

```javascript
const http = require("http");

const host = 'localhost';
const port = 8000;

const requestListener = function (req, res) {
  const { headers, method, url } = req;
  let body = [];
  req.on('error', (err) => {
    console.error(err);

    res.setHeader("Content-Type", "application/json");
    res.writeHead(500);
    res.end(`{"message": "Internal server error ${err.message}"}`);
  }).on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    console.log(`Received request [${method}] ${url}`);
    console.log(`Received request with headers`, headers);
    console.log(`Received request with body data`, body);

    res.setHeader("Content-Type", "application/json");
    res.writeHead(200);
    res.end(`{"message": "This is a JSON response"}`);
  });
};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});
```

Some important points about this web server are:

- All the received requests are handled by the requestListener function. There are no defined routes or HTML 404 errors. If we want these features, we need to program them within the server.
- The response is sent using the res.end() function. There are no views or HTML pages available.

## Build a web server with Express

Now, let's utilize a web framework to avoid implementing routing and templating from scratch. There are several frameworks available, but we will use Express because it is fast, minimalist, extensible, and widely used. To install Express:

```bash
npm install express -g
npm install -g express-generator
```

To generate the project type:

```bash
express --view=pug MyFirstProject

   create : MyFirstProject/
   create : MyFirstProject/public/
   create : MyFirstProject/public/javascripts/
   create : MyFirstProject/public/images/
   create : MyFirstProject/public/stylesheets/
   create : MyFirstProject/public/stylesheets/style.css
   create : MyFirstProject/routes/
   create : MyFirstProject/routes/index.js
   create : MyFirstProject/routes/users.js
   create : MyFirstProject/views/
   create : MyFirstProject/views/error.pug
   create : MyFirstProject/views/index.pug
   create : MyFirstProject/views/layout.pug
   create : MyFirstProject/app.js
   create : MyFirstProject/package.json
   create : MyFirstProject/bin/
   create : MyFirstProject/bin/www

   change directory:
     $ cd MyFirstProject

   install dependencies:
     $ npm install

   run the app:
     $ DEBUG=myfirstproject:* npm start

# Goto the project folder
cd MyFirstProject

# Install default dependiencies
npm install

# Start the servive
npm start
```

Now, we have a project with routing and templates using Pug. [Pug](https://github.com/pugjs/pug) is a template engine that simplifies the readability of the template code. However, you can choose any other template engine integrated with Express from the [Express template engines](https://expressjs.com/en/resources/template-engines.html).

To start working with Pug, you can use the website <https://pug2html.com/>.

Now that we have built and running the project with Express, let's discuss other features required for a professional service:

- Logging
- Documented APIs
- HATEOAS
- Consistent error handling

We will cover these topics in the upcoming posts.
