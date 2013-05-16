---
layout: post
title: "Understanding (and manipulating) `this` in JavaScript"
date: 2013-05-14 23:19
comments: true
categories: 
---

Knowing about `this` in JavaScript is not only very basic but also very important.

A lot of good articles have been written which describe how `this` in JavaScript works, for example:  
- [Mozilla MDN](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Operators/this)  
- [Yehuda Katz](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)  
- [impressivewebs.com](http://www.impressivewebs.com/javascript-this-different-contexts/)  

Knowing how you can manipulate `this` gives you great power and you no more have to write `var self = this`.   

See for example this code:  
``` javascript
var myClass = function() {
    this.files = [];
}

myClass.prototype.addFile = function(file) {
    this.files.push(file);
}

myClass.prototype.loadFilesFromFolder = function() {
    fs.readdir('/tmp', function(err, files) {
        if(err) throw new Error(err);
        files.forEach(this.addFile);
    });
}

var myInstance = new myClass();
myInstance.loadFilesFromFolder();
```

Now if you try to run that code in your node.js environment, it will give you the following error:
``` text 
files.forEach(this.addFile);
      ^
TypeError: undefined is not a function
    at Array.forEach (native)
```

This comes because the addFile method was not found on the this keyword. So what is then exactly the this keyword now? Lets try it and log it to the console:
``` text
{ ArrayBuffer: [Function: ArrayBuffer],
  Int8Array: { [Function: Int8Array] BYTES_PER_ELEMENT: 1 },
  Uint8Array: { [Function: Uint8Array] BYTES_PER_ELEMENT: 1 },
  Uint8ClampedArray: { [Function: Uint8ClampedArray] BYTES_PER_ELEMENT: 1 },
  Int16Array: { [Function: Int16Array] BYTES_PER_ELEMENT: 2 },
  Uint16Array: { [Function: Uint16Array] BYTES_PER_ELEMENT: 2 },
  Int32Array: { [Function: Int32Array] BYTES_PER_ELEMENT: 4 },
  Uint32Array: { [Function: Uint32Array] BYTES_PER_ELEMENT: 4 },
  Float32Array: { [Function: Float32Array] BYTES_PER_ELEMENT: 4 },
  Float64Array: { [Function: Float64Array] BYTES_PER_ELEMENT: 8 },
[...]
```  
The readdir method obviously uses an ArrayBuffer which then executes the callback function we defined - clearly manipulating the context we had expected.

So how can we manipulate this to use the this we expected?

## The bind() method

The `bind` method([https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/bind](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/bind)) takes an infinite number of arguments, where the first one is the `this` value and all the others can be used to pre-specify arguments to pass.

The bind method then creates a copy of the function.

For example:
``` javascript
var helloConsole = console.log.bind(null, 'Hello!');

helloConsole() // outputs "Hello!"
```

In this example i set the `this` keyword to `null` as we don't really need it and instead i passed a default argument to the console.log method.

To come back to our example we can also simply use it to set the this keyword to a value we want:
``` javascript
myClass.prototype.loadFilesFromFolder = function() {
    fs.readdir('/tmp', function(err, files) {
        files.forEach(this.addFile);
    }.bind(this));
}
```
Well if we run the code again the error no more occurs but instead there is a new error:
```
      this.files.push(file);
                 ^
TypeError: Cannot call method 'push' of undefined
```

The problem again is again that the `files` array does not exist on the `this` instance. Advanced methods like `Array.forEach`, `Array.map` or `Array.filter` all accept a second argument which should be used as the `this` value in the passed callback function.

``` javascript
myClass.prototype.loadFilesFromFolder = function() {
    fs.readdir('/tmp', function(err, files) {
        files.forEach(this.addFile, this);
    }.bind(this));
}
```
Now our code actually does what it was supposed to do.