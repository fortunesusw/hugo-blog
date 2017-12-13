---
title: "Javascript 使用jest单元测试"
date: 2017-12-13T12:31:53+08:00
subtitle: "mock部分"
tags: ['javascript', 'jest', 'unit test']
---

> [官方文档](https://facebook.github.io/jest/docs/en/getting-started.html)， 文档很详细， 内容也比较丰富

`jest`是`facebook`出品，是一个完整的`js`单元测试框架，有比较完善的测试方案，也可以配合其他断言库、 `mock`库等

<!--more-->

## mock方式

1. mock functions
2. mock module 

## mock functions

mock函数， 比较灵活， 可以灵活地`mock`我们想要的预期场景

比如我们要测试`redis`:

```javascript
//redis.js
const redis = require('redis');
const EXPIRED = 30 * 24 * 3600;

function createClient() {
    return redis.createClient()
}

let set = function (key, value) {
    return new Promise((resolve, rejected) => {
        let client = createClient();
        client.set(key, value, 'EX', EXPIRED, (err, result) => {
            if (!err) {
               resolve(result);
            }
            client.quit();
        });
    });
};

module.exports = set;
```

```javascript
//resid.test.js
let mockRedisClient = {
    on: jest.fn(),
    quit: jest.fn()
};
let mockRedis = {
    createClient: jest.fn(() => mockRedisClient),
};
jest.mock('redis', () => mockRedis);
let redis = require('../redis');

describe('redis string', () => {
    test('normal set', () => {
        let set = jest.fn((k, v, ex, timeout, callback) => {
            expect(k).toMatch('key');
            expect(v).toMatch('value');
            expect(ex).toMatch('EX');
            expect(timeout).toBe(24 * 30 * 3600);

            callback(null, 'OK')
        });
        mockRedisClient.set = set;

        return redis.set('key', 'value').then(v => {
            expect(mockRedis.createClient.mock.calls.length).toBe(1);
            expect(set.mock.calls.length).toBe(1);
            expect(mockRedisClient.quit.mock.calls.length).toBe(1);
            expect(v).toMatch('OK');
        });
    });
});
```

## mock module
mock整个js文件， 比较不灵活， 场景比较固定
比如我们在单元测试的时候，不生成日志， 就可以把日志模块mock掉，只要保持api不变即可。

```javascript
//log.js
const fs = require('fs');
const {createLogger, format, transports} = require('winston');
const {combine, timestamp, printf, splat} = format;
// winston3.0版本不支持按日期滚动
// const DailyRotateFile = require('winston-daily-rotate-file');

 //使用morgan记录express的访问日志
const morgan = require('morgan');

const path = require('path');
const logDir = path.join(__dirname, '../logs');

const myFormat = printf(info => {
    return `${info.timestamp} [${info.level}] ${info.message}`;
});

const ts = {
    console: new transports.Console(),
    debug: new transports.File({filename: logDir + '/debug.log', level: 'debug'}),
    info: new transports.File({filename: logDir + '/info.log', level: 'info'}),
    warn: new transports.File({filename: logDir + '/warn.log', level: 'warn'}),
    error: new transports.File({filename: logDir + '/error.log', level: 'error'}),
};

const debugLogger = createLogger({
    exitOnError: false,
    format: combine(
        timestamp(),
        myFormat
    ),
    transports: [
        ts.debug, ts.console
    ],
});

const infoLogger = createLogger({
    exitOnError: false,
    format: combine(
        timestamp(),
        myFormat
    ),
    transports: [
        ts.info, ts.console
    ],
});

const warnLogger = createLogger({
    exitOnError: false,
    format: combine(
        timestamp(),
        myFormat
    ),
    transports: [
        ts.warn, ts.console
    ],
});

const errorLogger = createLogger({
    exitOnError: false,
    format: combine(
        timestamp(),
        splat(),
        myFormat
    ),
    transports: [
        ts.error, ts.console
    ],
});

/**
 * https://github.com/expressjs/morgan
 */
const custom_format = '[:date[iso]] :remote-addr ":method :url HTTP/:http-version" :status ' +
    ':res[content-length] :response-time ":referrer" ":user-agent"';
const accessLogStream = fs.createWriteStream(path.join(logDir, 'access.log'), {flags: 'a'});
const accessLogger = morgan(custom_format, {stream: accessLogStream});

const Logger = function () {

};

const DefaultLogger = new Logger();

Logger.prototype.debug = function (message) {
    debugLogger.debug(message);
};
Logger.prototype.info = function (message) {
    infoLogger.info(message);
};
Logger.prototype.warn = function (message) {
    warnLogger.warn(message);
};
Logger.prototype.error = function (message, err) {
    if (err)
        errorLogger.error("%s ### %o", message, err);
    errorLogger.error("%s", message);
};
Logger.prototype.fatal = function (message, err) {
    if (err)
        errorLogger.error("%s ### %o", message, err);
    errorLogger.error("%s", message);
};

function getLogger(category) {
    if (!category)
        return DefaultLogger;
    switch (category) {
        case 'access':
            return accessLogger;
            break;
        default:
            return DefaultLogger;
            break;
    }
}

module.exports = {
    getLogger,
};
```

```javascript
//mock log.js
const Logger = function () {

};

const DefaultLogger = new Logger();

Logger.prototype.debug = function (message) {
};
Logger.prototype.info = function (message) {
};
Logger.prototype.warn = function (message) {
};
Logger.prototype.error = function (message, err) {
};
Logger.prototype.fatal = function (message, err) {
};

function getLogger(category) {
    return DefaultLogger;
}

module.exports = {
    getLogger
};
```

