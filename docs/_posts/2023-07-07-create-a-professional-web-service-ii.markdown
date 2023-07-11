---
layout: post
title:  "Create a professional web service - Part II - Logging"
date:   2023-07-07 07:55:56 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

## Adding logging capabilities

We want to log all errors and traces to the console in order to monitor the service and detect malfunctions in both development and production modes.

To achieve this, we need to use a logger that displays human-readable messages in development mode and messages that can be ingested by a logger agent for centralized processing, analysis, and generating alarms.

We will do with [Winston](https://www.npmjs.com/package/express-winston) combined with [Morgan](https://www.npmjs.com/package/morgan), a package already installed in the base Express project.

First, install Winston and configure it to enable logging messages:

```bash
# From the project root folder
npm install winston --save

# Create a configuration file for the project
mkdir config
cd config
touch config.js

# Create a configuration file for Winston and Morgan
cd ..
mkdir api
cd api
touch logger.js
```

/config/config.js

```javascript
module.exports = {
    client: "TEST",
    service: "PROJECTNAME"
}
```

/api/logger.js

```javascript
var config = require('../config/config.js');
const { createLogger, format, transports } = require("winston");

const level = process.env.LOG_LEVEL || "debug";
 
function formatParams(info) {
  const { timestamp, level, message, ...args } = info;
 
  return `${timestamp} ${level}: ${message} ${Object.keys(args).length
    ? JSON.stringify(args, "", "")
    : ""}`;
}
 
const developmentFormat = format.combine(
  format.colorize(),   // Indiquem que volem colors
  format.timestamp(),
  format.align(),
  format.printf(formatParams)
);
 
const productionFormat = format.combine(
  format.timestamp(),
  format.align(),
  format.printf(formatParams),
  format.json()
);
 
let logger;

const transportsCustom = [
  // Use the console to print the messages => PM2 and Docker saves to file
  new transports.Console(),
  // Print all the error level messages inside the error.log file
  //new transports.File({ filename: 'logs/error.log', level: 'error' }),
  // Print all the error message inside the all.log file
  // (also the error log that are also printed inside the error.log(
  //new transports.File({ filename: 'logs/all.log' }),
] 

if (process.env.NODE_ENV !== "production") {
  logger = createLogger({
    level: level,
    format: developmentFormat,
    //transports: [new transports.Console()]
    transports: transportsCustom
  });
 
} else {
  logger = createLogger({
    level: level,
    format: productionFormat,
    // Set the client and service name to easily identify the logs
    defaultMeta: {
      service: config.service,
      client: config.client,
    },
    transports: transportsCustom
  });
}
module.exports = logger;
```

Update the the file `/app.js`:

1. Change `var logger = require('morgan');` for `var morgan = require('morgan');`
2. Add `const logger = require("./api/logger");`
3. Delete `app.use(logger('dev'));`
4. Add after `var app = express();` and install body-parser if needed `npm install body-parser --save`:

```javascript
// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }));
// parse application/json
app.use(bodyParser.json());
```

5. Add before routes in `app.js`:

```javascript
const morganFormat = process.env.NODE_ENV !== "production" ? "dev" : "combined";

app.use(
  morgan(morganFormat, {
    skip: function(req, res) {
      return res.statusCode < 400;
    },
    stream: {
      write: (message) => logger.http(message.trim()),
    },
  })
);
```

Finally, to test our new logger we need to add som logs into the code, for example, in the `./app.js` file add:

```javascript
// Add this constant at the begining of the file
const logger = require("../api/logger");

// Add this logs at the end of the file
logger.error('This is an error log');
logger.warn('This is a warn log');
logger.info('This is a info log');
logger.http('This is a http log');
logger.debug('This is a debug log');
```

To test this changes start the service in development mode `npm start` and then in production mode `NODE_ENV=production npm start`
