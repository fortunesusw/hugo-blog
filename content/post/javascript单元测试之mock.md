---
title: "Javascript单元测试之mock"
date: 2017-11-27T21:56:18+08:00
subtitle: ""
tags: ["unit test", "javascript", "sinon"]
---

在真实的项目中，各种各样的功能会使代码很难被测试。不论是浏览器中的Ajax请求，定时器，日期时间等功能，还是Node.js中的数据库，网络，文件操作。

这些功能之所以很难被测试是因为你不能在代码中控制它们。如果你使用了Ajax，为了让你的代码通过测试，你需要有一个服务器来响应请求。如果你使用了setTimeout，你的测试代码将不得不等待定时器被触发。如果你访问了数据库或是网络，也是同样的道理，你需要一个含有正确数据的数据库或是一个网络服务器。

通过使用Sinon，我们可以让测试复杂代码变得不复杂。

<!--more-->

## 安装

```
npm install sinon
var sinon = require('sinon');
```

## 常规说明

* `Spy`: 它提供有关函数调用的信息，而不影响它们的行为
* `Stub`: 像`Spy`，但完全取代了函数功能。这使得可以做任何你喜欢的stubbed函数 - 抛出一个异常，返回一个特定的值，等等
* `Mock`: 通过结合`Spy`和`Stub`来更换整个对象
* `Fake timers`: 可以用来穿越时间，例如触发`setTimeout`
* `Fake XMLHttpRequest and Server`: 可以用来伪造`Ajax`请求和响应

## Spy使用

* 收集函数的调用信息
* 原始函数会运行
* 适合验证函数是否被调用， 调用了几次等

```javascript
var sinon = require('sinon');
var spy = sinon.spy();

//We can call a spy like a function
spy('Hello', 'World');

//Now we can get information about the call
console.log(spy.firstCall.args); 

//output: 
//['Hello', 'World']
```

```javascript
var sinon = require('sinon');

var User = {
    setName: function (name) {
        this.name = name;
    }
};

//Create a spy for the setName function
var setNameSpy = sinon.spy(User, 'setName');

//Now, any time we call the function, the spy logs information about it
User.setName('ABD');

//Which we can see by looking at the spy object
console.log(setNameSpy.callCount);

//Important final step - remove the spy
setNameSpy.restore();

//output: setName被调用一次
//1 
```

```javascript
//使用mocha框架测试
var sinon = require('sinon');
var assert = require('assert');

function myFucntion(condition, callback) {
    if(condition)
        callback();
}

describe('myFunction', function () {
    it('should call the callback function', function () {
        var callback = sinon.spy();
        myFucntion(true, callback);

        assert(callback.calledOnce);
    })
});

//output
//  myFunction
//    ✓ should call the callback function
//
//  1 passing (7ms)

```

## Stub使用

* 拥有`Spy`所有功能
* 原始函数不会运行， 直接替换

适合场景：

* 替换测试变慢或者难以测试的外部调用
* 根据函数返回值来触发不同的操作
* 测试异常情况， 如抛出异常等

```javascript
var sinon = require('sinon');
var stub = sinon.stub();

stub('hello');

console.log(stub.firstCall.args);

//output
//['hello']
```

```javascript
var sinon = require('sinon');
const jsdom = require("jsdom");
const { JSDOM } = jsdom;
var $ = require('jquery')((new JSDOM(``, { runScripts: "outside-only" })).window);

function saveUser(user, callback) {
    $.post('/user', {
        first: user.first,
        last: user.last
    }, callback);
}

describe('saveUser', function () {
    it('should call callback after saving', function () {
        var post = sinon.stub($, 'post');

        var expectedUrl = '/user';
        var expectedParams = {
            first: 'firstname',
            last: 'lastname'
        };

        var user = {
            first: 'firstname',
            last: 'lastname'
        };

        var callback = sinon.spy();

        saveUser(user, callback);

        post.restore();
        sinon.assert.calledWith(post, expectedUrl, expectedParams);
    })
});

//output
//  saveUser
//    ✓ should call callback after saving
//
//
//  1 passing (11ms)

```

## mock使用
`Mocks`是使用`stub`的另一种途径。如果你曾经听过"mock对象'这种说法，这其实是一码事 ---- `Sinon`的`mock`可以用来替换整个对象以改变其行为，就像函数`stub`一样。

基本上只有需要针对一个对象的多个方法进行stub时才需要使用`mock`。如果只需要替换一个方法，使用`stub`更简单。

使用`mock`时你要很小心。由于`mock`强大的功能，它很容易导致你的测试过于具体 ---- 测试了太多，太细节的内容 ---- 这很容易在不经意间导致你的测试变得脆弱

与`spy`和`stub`不同的是，`mock`有内置的断言。你需要预先定义好`mock`对象期望的行为并在测试结束前执行验证函数。

```javascript
var sinon = require('sinon');
var store = require('store');

function incrementStoredData(value, amount){
    var total = store.get(value) || 0;
    var newtotal = total + amount;
    store.set(value, newtotal);
}

describe('incrementStoredData', function() {
    it('should increment stored value by one', function() {
        var storeMock = sinon.mock(store);
        storeMock.expects('get').withArgs('data').returns(0);
        storeMock.expects('set').once().withArgs('data', 1);

        incrementStoredData('data', 1);

        storeMock.restore();
        storeMock.verify();
    });
});

//output
//  incrementStoredData
//    ✓ should increment stored value by one
//
//
//  1 passing (12ms)
```

## 最佳实践： 使用sinon.test()

在前面的示例中，我们使用了`stub.restore()`或`mock.restore()`来执行清理操作。这个操作是必要的，否则测试替身会一直存在并给其它测试带来负面影响或是导致错误。

但是直接使用`restore()`方法是有问题的。因为有可能在`restore()`执行之前测试代码就因为错误提前结束执行了。

有两种方法可以解决这个问题：我们可以把所有的代码放在一个`try catch`块中，这样就可以在`finally`块中执行`restore()`而不用担心测试代码是否报错。

还有一种更好的方式，就是把测试代码包裹在`sinon.test()`中：

```javascript
it('should do something with stubs', sinon.test(function() {
    
    //注意： 使用this而非sinon
    var stub = this.stub($, 'post');

    doSomething();

    sinon.assert.calledOnce(stub);
    
    //没有stub.restore()处理了
});
```

把测试代码包裹在`sinon.test()`之中后，我们就可以使用`Sinon的沙盒特性`了。它允许我们通过`this.spy()`、`this.stub()`和`this.mock()`来创建`spy`，`stub`和`mock`。任何使用沙盒特性创建的测试替身都会被自动清理。

注意上边的例子中没有`stub.restore()`操作，因为在沙盒特性下的测试里它变得不必要了。

如果在所有地方都使用了`sinon.test()`，那么你就可以避免由于某个测试未能清理它内部的测试替身而导致后续测试随机失败的情况。

## Sinon工作原理浅析

为了更好的理解Sinon的工作原理，让我们看一些和它工作原理有关的例子。这将有利于我们更好的理解Sinon究竟做了哪些工作并在不同的场景中更好的利用它。

#### spy本质上是一个函数包装器

```javascript
function createSpy(targetFunc) {
    const spy = function () {
        spy.args = arguments;
        spy.returnValue = targetFunc.apply(this, arguments);
        return spy.returnValue;
    };

    return spy;
}

function sum(a, b) {
    return a + b;
}

const spiedSum = createSpy(sum);

spiedSum(10, 5, 1);

console.log(spiedSum.args);
console.log(spiedSum.returnValue);

// output 
// { '0': 10, '1': 5, '2': 1 }
// 15
```

#### stub是把一个函数替换成另一个

```javascript
var stub = function() { };

var original = thing.otherFunction;
thing.otherFunction = stub;

// 现在开始，所有对thing.otherFunction的调用都会被stub的调用所取代
```


