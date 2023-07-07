---
layout: post
title:  "Create a professional web service - Part II"
date:   2023-07-07 07:55:56 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

## Adding logging capabilities

We want log to console all the errors and traces to monitor the service and detect malfunction.
To do this we ned to use a logger that display messages readable by humans in development mode and messages readable by a logger agent that can ingest our logs and centralize and process to analize and generate alarms.

We can do this with [Winston](https://www.npmjs.com/package/express-winston) combined with [Morgan](https://www.npmjs.com/package/morgan), a package already installed in the base Express project.

So, we install Winston and configure to allow logging messages:

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
  // Allow the use the console to print the messages => PM2 and Docker saves to file
  new transports.Console(),
  // Allow to print all the error level messages inside the error.log file
  //new transports.File({ filename: 'logs/error.log', level: 'error' }),
  // Allow to print all the error message inside the all.log file
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
    // En entorn de producció indiquem el servei i el client
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
4. Add:

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
app.use(
  morgan(morganFormat, {
    skip: function(req, res) {
      return res.statusCode >= 400;
    },
    stream: {
      write: (message) => logger.http(message.trim()),
    },
  })
);
```

Finally, to test our new logger we need to add som logs in the code, for example, in the `./ap.js` file we can add some logs:

```javascript
// Add this comnstant at the beginning of the file
const logger = require("../api/logger");

// Add this logs at the end of the file
logger.error('This is an error log');
logger.warn('This is a warn log');
logger.info('This is a info log');
logger.http('This is a http log');
logger.debug('This is a debug log');
```

To test this changes start the service in development mode `npm start` and then in production mode `NODE_ENV=production npm start`
