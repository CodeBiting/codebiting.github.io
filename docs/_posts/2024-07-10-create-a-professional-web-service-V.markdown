---
layout: post
title:  "Create a professional web service - Part V - Static analysis with ESLint"
author: Jordi Dalmau Heras
date:   2024-07-10 17:48:24 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

## Static Analysis

ESLint is a tool for a static code analysis and code style enforcing <https://eslint.org/docs/latest/use/getting-started>.

### Installation and configurartion in ExpressJS

Execute the installation and configure with project preferences.

```bash
npm init @eslint/config

# Choose:
# To check syntax, find problems, and enforce code style
# What type of modules does your project use? CommonJS (require/exports)
# Which framework does your project use? None of these
# Does your project use TypeScript? No
# Where does your code run? Node
# Use a popular style guide Standard: https://github.com/standard/standard
# What format do you want your config file to be in? JavaScript
# Which package manager do you want to use? npm
```

Configure the rules of the eslint, edit the file `.eslintrc.js`:

For example, to add a rule to use 4 space identation to get the code more compact and in switch-case ident case <https://eslint.org/docs/latest/rules/indent#switchcase>

```yaml
    indent: ['error', 4, { SwitchCase: 1 }],
```

Or use semicolons to make the code easier to read:

```yaml
    semi: ['error', 'always']
```

Add to the package.json file the eslint tests:

```json
  "scripts": {
    "start": "node ./bin/www",
    "test": "node ./node_modules/mocha/bin/mocha",
    "test-apis": "node ./node_modules/mocha/bin/mocha ./test/api/",
    "test-routes": "node ./bin/www & P1=$! && sleep 2 && node ./node_modules/mocha/bin/mocha ./test/routes/v1/ && kill $P1",
    "test-eslint": "node ./node_modules/.bin/eslint app.js bin/www api/*.js routes/*.js"
  }
```

Analyze and correct the style of the javascript files

```bash
node ./node_modules/.bin/eslint app.js
node ./node_modules/.bin/eslint app.js --fix
node ./node_modules/.bin/eslint bin/www --fix
node ./node_modules/.bin/eslint .eslintrc.js --fix
```
