# Notes (Opinionated)
Mastering Modern Mobile Front-End: Ionic, Angular, CSS3/SCSS, HTML5, ..., Angular 2.0, postCSS, ECMA2015, and maybe TypeScript.  
Only Modern browsers are considered. Fuck IE10-.

TODO:
1. $q
2. waterline
3. custom $http using $q and waterline syntax to query data from client, high abstraction to by pass thinking of query made from server.
3. SASS
4. Ionic custom sass
5. dig Ionic / Angular

#TOC
## CSS
[Flexbox Examples](#flexbox)
## Angular 1.x
[$q promises](#$q-promise)

--------------------------------------------------------------------
#CSS
##Flexbox
### Properties 
```css
.row {
  display: flex;

  flex-direction: row | row-reverse | column | column-reverse;
  flex-wrap: nowrap | wrap | wrap-reversea;
  /* shorthand */
  flex-flow: @flex-direction || @flex-wrap;

  /* main axis */
  justify-content: flex-start | flex-end | center | space-between | space-around;

  /* cross-axis useful only when flex-wrap is wrap*/
  align-content: flex-start | flex-end | center | space-between | space-around;

  /* cross-axis per line */
  align-items: flex-start | flex-end | center | baseline | stretch
}

.col {

  order: <integer>;

  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto | <length>
  /* shorthand (use short)*/
  flex: @flex-grow @flex-shrink @flex-basis;  /* default none */ 

  align-self: flex-start | flex-end | center | space-between | space-around;

  /* float, clear, vertical-align does not work on flex item, but it does work on flex container. */
}

```
### Centering single content
#### Method 1
```css
.row {
  display: flex;
  height: 50vw; /* if specified, content will be centered vertically  */
}

.col {
  flex: 0 0 20vw;
  height: 20vw;
  line-hieght: 20vw;
  text-align: center;
  margin: auto;
}
```

#### Method 2
```css
.row {
  display: flex;
  height: 50vw;
  justify-content: center;
  align-items: center;
}
.col {
  flex: 0 0 20vw
  height: 20vw;
  line-hieght: 20vw;
  text-align: center;
  /* no margin auto */
}
```


Q: would .rowOuter>.rowInner>.col work?  
i.e. .rowInner both display: flex, and flex: 0 0 90vw;






[Back to TOC](#toc)

--------------------------------------------------------------------
# Angular 1.x
## $q promises
```js
// Style 1
$q.fcall(function(){ ... })
.then( function(input){
  
}, function(reason){
  
})
.then( function (value){
  
})
.catch( function (error) {
  
} )
.done()

// Style 2
var func1= function (input){
  var deferred = $q.defer();
  funcUsingCb( function (value){
    if(noError){
      deferred.resolve(data)
    } else {
      deferred.reject(data)
    }
  return deferred.promise;
};

func1(input)
.then(function (value){
  
}, function (error){
  
})
.then(function(value){
  
}, function (error){
  
})
```
If the value returned to fullfilment handler in `.then` is promise object, then instead of `input` being the promise object, `.then` will use the returned promise, and resolved value of passed promise will be passed to arguement of fulfillment handler (in our case, `input`).

If the promise gets fulfilled but there is no fulfillment handler in `.then`, AND if you assigned promise to a variable, then resolved value will be assigned to the variable.  

Similarly if no rejection handler then error object will be assigned to the variable (note noes that `throw`, just pass error obj); recommend using `.catch()` to handle error only rather than `.then(null, function (error) {})`

A handler in `.finally()` does not accept arguments. While the value will be ignored, error will be assigned to the variable. If promise is passed in then, it will wait for promosise to resolve(fulfill or reject)

### Array of independent promises
Turn an array of functions (that return values and/or promises) into **a** promise;  
#### $q.all
`$q.all` resolves to a promise when all promises are resolved in the array. Note that if the function just returns value then that function is considered resolved immediately but still has to defer until others to resolve.  
If one of the function is rejected then retuned promise is immediately rejected not waiting for the rest of the batch.
```js
//
$q.all([funcReturnPromise1, funcReturnPromise2])
.spread( function (value1, resolved1){ ... })
```
Use `.spread` is like array of `.then`s; i.e. Conceptually;
`[.then, .then]`. where each then accounts for value or promise object.

#### $q.allSettled
Does not reject immediately if one of the resolved value is an error.  
A single array of objects with properties `state`, `value`, and `reason` are passed to a fulfillment handler. There is no rejection handler, rejection is done inside fulfillment handler by checking `array.state !== 'fulfilled'`. since it passes single argument it uses `.then`.
```js
$q.allSettled([func1, func2])
.then(function (resultsArray){
  var values = [];
  var reasons = [];
  resultsArray.forEach(function(result){
    if(result.state === 'fulfilled'){
      values.push(result.value);
    } else {
      reasons.push(result.reason)
    }
  });
  return values;
})
```

#### $q.any
If fulfilled then only first value is considered.
If rejected then all the rejected are considered.
```js
$q.any(promise)
.then(function (first){})
.then(function (error){});
```


### Array of dependent promises (like piping)
As passed in value is promise the `.then` applies to returned promise we can nest and unnest `.then` as we please. Only time we will nest `.then` within `.then` is when we wanna use multiple variables and closures;

If we have number of **Promise-Producing** functions we can convert;
```js
return foo(initialVal).then(bar).then(baz).then(qux)
```
to
```js
var funcs = [foo, bar, baz, qux];
return funcs.reduce($q.when, $q(initialVal));
```
Note, returned value is promise object.


## Handling errors;
```js
// Style 1
$q.fcall(foo.bind(null, arg1))
.then(function (value){
  ...
}, function (error){
  ...
})
.then(function (value){
  ...
}, function (error){
  ...
})

// Style 2 ?? does this work?
$q.fcall(foo.bind(null, arg1))
.then(function (value){
  ...
})
.catch(function (error){
  ...
})
.then(function (value){
  ...
})
.catch(function (error){
  ...
})

//Style 3
$q.fcall(foo.bind(null, arg1))
.then(function (value){
  ...;
  if(someCondition){
    return new Error(1)
  } else {
    return value;
  }
})
.then(function (value){
  ...;
  if(someCondition){
    return new Error(2)
  } else {
    return value;
  }
})
.catch(function (error){
  if(error == 1){
    ...
  } else if(error == 2) {
    ...
  }
})
```
If don't care which `.then` throws an error then use one `.catch` style at the end.

`.done()` makes sure that the error will be handled if `.then` before it didn't handle errors. It will explicitly re-throw it rather than passing the error object.

[Back to TOC](#toc)

---------------------------------------------------------------------

## Digression / Ideas

Wouldn't it be nice to have a db querying capability from client side?
eg) `Posts.findOne({attr1: "content1"})` do http request, upon receiving the request from server it runs the exact command.

당신이 사람이 아니라는것을 증명해보세요.
