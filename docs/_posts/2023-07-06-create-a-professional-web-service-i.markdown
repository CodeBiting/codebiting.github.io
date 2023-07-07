---
layout: post
title:  "Create a professional web service - Part I"
date:   2023-07-06 09:58:12 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

First of all install NodeJS, npm and Express. There are several methos of installation and multiple tutorials that explain, in this case we will install node version manager over a Ubuntu 22.04LTS:

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

First we will make a very simple web service with node:

```javascript
const http = require("http");

const host = 'localhost';
const port = 8000;

const requestListener = function (req, res) {
  console.log(`Received request with body data ${JSON.strifigy(req.body)}`);
  res.setHeader("Content-Type", "application/json");
  res.writeHead(200);
  res.end(`{"message": "This is a JSON response"}`);
};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});
```

Some important things about this web server are:

- All the requests received are handled by the `requestListener` function, there are no routes defined, there are no html 404 errors. If we want this we need to program inside the server.
- The response is sent by the function `res.end()`, we don't have any view template or html pages.

Now lets use a web framework to avoid implement routing and templating. There are several frameworks, but we will use Express because is fast, minimialits, extensible and widely used. To install Express:

```bash
npm install express -g
npm install -g express-generator
```

To generate the project:

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

With this we have a project with routing, and templates with pug. [Pug](https://github.com/pugjs/pug) is a template engine that simplifies the readabilty of the templates code. But you can choose any other template engine integrated with Express [Express template engines](https://expressjs.com/en/resources/template-engines.html).

To begin to work with pug you can use this page <https://pug2html.com/>

Ok, we have the project build and running with Express but we need other features to have a professional service:

- Logging
- Documented APIs
- HATEOAS
- Return consistent errors

We will see in the nexts posts
