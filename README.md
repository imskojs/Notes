# Notes (Opinionated)
Mastering Modern Mobile Front-End: Ionic, Angular, CSS3/SCSS, HTML5, ..., Angular 2.0, postCSS, ECMA2015, and maybe TypeScript.  
Only Modern browsers are considered. Fuck IE10-.

TODO:  
2. dig Ionic /Angular  
3. SASS  
4. Ionic custom sass  

#TOC
## CSS
[Flexbox Examples](#flexbox)
## Angular 1.x
[$q promises](#$q-promises)
[$emit, $broadcast](#$emit-$broadcast)
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
## Using directives for dom manipulation
MUST use directive if you manipulate(get, set or both) a DOM more than once.
Don't need to use directive if you manipulate a dom element only once, just use controller.
Custom directives's link function load before methods inside $ionView.beforeEnter. Only way for custom directive to load after the methods in controller has called is to not use $ionView.<whatever>. Of course, if all else fails use $timeout(...,0) in link function, but that can cause problems later when additional things has to be done.  
When controller is defined in ui-router, in config, directives do not get the same $scope, it inherits, i.e. scope.$parent == $scope, where scope is directive linkFn scope, and $scope is Ctrl that was defined in ui-router scope.






##$q promises
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

##$emit, $broadcast
When we talk about $rootScope.$emit, this only lets other $rootScope listeners to catch it. Good to use when we don't want every $scope to get it. Generally a high level communication between each other, like some top level management talking to each other in cabin so the other employee can't hear them.
 When we talk about $rootScope.$broadcast, this is a method that lets pretty much everything hear it. This would be the equivalent of teacher yelling that open your notebook so everyone in the classroom hears it.
When we talk about $scope.$emit, this is when you want that $scope and all its parents and $rootScope to hear the event. This is a child whining to their mother at home (but not in market where other kids can hear).
When we talk about $scope.$broadcast, this is for the $scope itself and its children. This is a child whispering to its stuffed animals so their parents can't hear.

So now we understood these functionality now discuss important point which create issues when we use these functionality.

 1. $rootScope live forever in application scope so if our controller is run multiple time then all $rootScope.$on inside  it will run all time whenever controller called and caught events will result in callback invoked multiple times. If we use $scope.$on instead of $rootscope.$on, the callback will be surly destroyed along with our controller implicitly by AngularJS application. This is a common cause of bugs in an AngularJS application.


2. If we must need to use $rootscope.$on in our application, in that case we need to manually destroy that on controller  destroy event.


sample code: 

var cleanListeneEvents = $rootScope.$on('someEvent', function(event, paramValue) { console.log(paramValue); });

$scope.$on('$destroy', function() {

//this will deregister that listener
  cleanListeneEvents();
});

3. Always prefer $rootScope.$emit() instead of $rootScope.$broadcast(). Because whenever we  broadcast on rootScope, event is bubled down to every scope currently existing. That can be make slow our application. It's not necessary to do that, if  our listeners are hooked to rootScope.


4. In rare case we should use these events, instead of this we can use service and inject with controller/ directive to communicate with each other. If we need to communicate changes all over our application use a service to maintain the values and load the values when the various views are active.

5. If we know that there is only one listener, and we have already encountered it, we can stop further propagation of the event by calling event.stopPropagation() on the event object passed to the event listener function.

6. The worst part is that an event based approach can get really messy to track, maintain and debug. It makes it very asynchronous, with no real easy way to track the flow of an application 

7. If we talk about performance, then $emit() is faster than $broadcast().

- See more at: http://firstcrazydeveloper.com/Blogs/BlogView.html/50/events-in-angularjs-emit-broadcast-and-on#sthash.5kmBo9kw.dpuf


[Back to TOC](#toc)

---------------------------------------------------------------------

## Digression / Ideas

Wouldn't it be nice to have a db querying capability from client side?
eg) `Posts.findOne({attr1: "content1"})` do http request, upon receiving the request from server it runs the exact command.    edit: turns out this is how sql injector attack happen.

당신이 사람이 아니라는것을 증명해보세요.
