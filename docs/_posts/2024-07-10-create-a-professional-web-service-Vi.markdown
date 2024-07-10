---
layout: post
title:  "Create a professional web service - Part VI - Git hooks to run tests automatically"
author: Jordi Dalmau Heras
date:   2024-07-10 08:12:05 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

## Pre-commit Git hook

A Git pre-commit hook is a script that runs automatically every time you run `git commit` before the commit actually happens. It is a type of Git hook, which are scripts that Git executes before or after events such as commit, push, and receive.

A pre-commit hook is located in the .git/hooks/ directory of a Git repository and is typically named pre-commit (without an extension). When you run git commit, Git will execute this script before finalizing the commit. The script can be written in any programming or scripting language, as long as it is executable.

### Why is useful?

Code Quality Assurance:

1. Linting: Automatically run linters to ensure code adheres to defined style guidelines.
2. Static Analysis: Perform static code analysis to catch common bugs and vulnerabilities.
3. Unit Tests: Run unit tests to make sure new commits do not break existing functionality.

Consistency:

1. Formatting: Automatically format code according to project standards (e.g., using tools like Prettier or Black).
2. Metadata Checks: Ensure commit messages follow a certain format or include necessary information.

Preventing Mistakes:

1. File Validations: Ensure no sensitive files (e.g., configuration files with credentials) are committed.
2. Dependency Checks: Verify that dependencies are correctly updated and committed.
3. Prevention of Large Files: Prevent accidental commits of large files which should not be in the repository.

Efficiency:

1. Automated Tasks: Automatically update documentation, run scripts, or perform other routine tasks without manual intervention.
2. Minimizing CI/CD Failures: By catching issues early, pre-commit hooks help reduce the chances of failed CI/CD builds.

### Installation and configurartion in ExpressJS

Install and configure the git pre-commit hook with the package `https://www.npmjs.com/package/pre-commit`

```bash
npm install pre-commit --save-dev

```

Add to the package.json file the tests that the pre-commit must run

```json
  "scripts": {
    "start": "node ./bin/www",
    "test": "node ./node_modules/mocha/bin/mocha",
    "test-apis": "node ./node_modules/mocha/bin/mocha ./test/api/",
    "test-routes": "node ./bin/www & P1=$! && sleep 2 && node ./node_modules/mocha/bin/mocha ./test/routes/v1/ && kill $P1",
    "test-eslint": "node ./node_modules/.bin/eslint app.js bin/www api/*.js routes/*.js"
  },
  "pre-commit": [
    "test-apis",
    "test-routes",
    "test-eslint"
  ],
```
