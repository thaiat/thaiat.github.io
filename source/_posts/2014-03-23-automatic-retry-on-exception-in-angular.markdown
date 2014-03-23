---
layout: post
title: "Automatic retry on exception in angular"
date: 2014-03-23 15:01:11 +0200
comments: true
categories: [angularjs, karma, unit test, javascript]
---
Every so often, you run across some action, which just fails, where the best response it to just try it again. This is particularly true when dealing with an external source, like a database or web service, which can have network or other temporary problems, which would have cleared up when you repeat the call seconds later. 

Often, these actions fail by throwing an exception which makes the process of trying the call again rather cumbersome. Having to deal with this a number of times in one application, I decided to wrap the full procedure up in an angular service, which I share here with you:

##Sleep function
First we need to implement a `sleep` function, that will just wait for a specified interval of milliseconds while returning a promise.

You would think that the best fit is to use `$timeout`.    
The problem is, it has a bad impact on your unit tests....
The reason is, if you use `$timeout`, you will have at the end of your test to call `$timeout.flush()` as many times as `$timeout` was called in your implementation.

Instead, if we use the standard javascript `setTimeout` function, we only have to call once `$rootScope.$apply()` at the end of our test and we're done.

Let's start by defining our service with an empty implementation:
In addition to `sleep`, we will expose 2 other functions `toAsync` and `retry`, that we'll explain later.

```javascript promiseService.js
var app = angular.module("myApp", []);

var dependencies = ['$q', '$rootScope'];
var service = function($q, $rootScope) {

	var sleep = function() {

	};

	var toAsync = function(){
	
	};

	var retry = function() {

	};

	return {
		sleep : sleep,
		toAsync : toAsync,
		retry : retry
	}	

};

app.factory("promiseService", dependencies.concat(service));
```

For the implementation of `sleep` we are going to generate our own promise, call `setTimeout`, and not forget to wrap the callback in a `$rootScope.$apply()` call.

```javascript sleep function
var sleep = function(interval) {
	// check parameter
	if(!(interval === parseFloat(interval)) || interval < 0)
		throw new Error("interval must be a positive float");
    
    var deferred = $q.defer();

    // sleep
    setTimeout(function () {
     	$rootScope.$apply(function () {
        	deferred.resolve(interval);
        });
    }, interval);
               
    return deferred.promise;
};
```

Before we continue, let's write some unit tests to verify the implementation.
I'm using here **jasmine 2.0** which comes with a new syntax for async testing.   
It's pretty simple : when testing an async function (aka a promise), you add a `done` parameter to your `it`, and you call `done()` when the promise returns some result.
In addition, if your implementation uses setTimeout, you add a call to `$rootScope.$apply()`

```javascript promiseService.tests.js
describe("Service : promiseService" , function() {
	var service;
	var $rootScope;
	var $httpbackend;
	var $http;

	beforeEach(module("myApp"));

	beforeEach(inject(function ($injector) {
        service = $injector.get("promiseService");
        $timeout = $injector.get("$timeout");
        $rootScope = $injector.get("$rootScope");
        $httpBackend = $injector.get("$httpBackend");
        $http = $injector.get("$http");
    }));

	it("should_be_defined", function(){
        expect(service).toBeDefined();
	});

	it("sleep_with_not_integer_interval_should_throw_exception", function () {
        expect(function () {
            service.sleep("invalid");
        }).toThrow(new Error("interval must be a positive float"));
    });

    it("sleep_with_not_positive_integer_interval_should_throw_exception", function () {
        expect(function () {
            service.sleep(-2);
        }).toThrow(new Error("interval must be a positive float"));
    });

    it("sleep_with_positive_integer_interval_should_succeed", function (done) {
        var interval = 100;
        service.sleep(interval).then(function (result) {
            expect(result).toEqual(interval);
            done();
        });
        $rootScope.$apply();
    });

});

```

We have a fully tested `sleep` function.

##toAsync function
The next function that we need to write is a function that will transform a passed function into a promise.
That way we can retry on fail, either standard functions, or promises.
The trick here, if you want your unit test to pass, is to encapsulate the call to the passed function in a try catch block.
It is similar to `$q.when`, but instead of passing a value or a promise, we pass a function or a promise.
```javascript toAsync
 var toAsync = function (action) {
    if (typeof action !== "function") {
        throw new Error("action must be a function");
    }
    var deferred = $q.defer();
    try {
        var retval = action();
        deferred.resolve(retval);

    }
    catch (ex) {
        deferred.reject(ex);
    }
    return deferred.promise;
};
```

And the unit tests :
```javascript toAsync unit tests
it("toAsync_with_invalid_parameter_function_should_throw_exception", function () {
    expect(function () {
        service.toAsync("invalid");
    }).toThrow(new Error("action must be a function"));
});

it("toAsync_with_valid_sync_function_should_succeed", function (done) {
    spyOn(mockHelper, 'addOne').and.callThrough();
    var action = function () {
        return mockHelper.addOne(100);
    };

    service.toAsync(action).then(function (result) {
        expect(result).toBe(101);
        expect(mockHelper.addOne).toHaveBeenCalledWith(100);
        done();
    });
    $rootScope.$apply();
});

it("toAsync_with_valid_async_function_should_succeed", function (done) {
    spyOn(mockHelper, 'getUrl').and.callThrough();
    $httpBackend.when('GET', '/dummy').respond(mockHelper.dummyResponse);
    var action = function () {
        return mockHelper.getUrl();
    };
    service.toAsync(action).then(function (result) {
        expect(mockHelper.getUrl).toHaveBeenCalled();
        expect(result.data).toEqual(mockHelper.dummyResponse);
        done();
    });

    $httpBackend.flush();
    $rootScope.$apply();
});

it("toAsync_with_faulty_sync_function_should_succeed", function (done) {
    spyOn(mockHelper, 'faultyFn').and.callThrough();
    var action = function () {
        return mockHelper.faultyFn();
    };
    service.toAsync(action).then(null, function (rejection) {
        expect(mockHelper.faultyFn).toHaveBeenCalled();
        expect(rejection.message).toBe("I'm a faulty function");
        done();
    });
    $rootScope.$apply();
});

var mockHelper = {
    faultyFn : function () {
        throw new Error("I'm a faulty function");
    },

    addOne : function (value) {
        return value + 1;
    },

    getUrl: function () {
        return $http.get("/dummy");
    },

    dummyResponse : {
        "id" : 1,
        "content" : "Hello World"
    }
};
```

The `mockHelper` object holds our unit test functions, so we can reuse and spy on them across the tests.
This is usefull for checking how many times they are called and with which parameters.


##retry function
This is the last piece of the puzzle.
To be as generic as possible, our function will accept a set of parameters in order to control how many times we should retry on fail, the interval between each trial, and also an interval multiplicator if we want to add some extra delay between each trial.

The implementation is straight forward, using the building blocks we previously wrote.
In the first part we just do some argument checking, and assign default values.
In the second part, we recursivly call resolver, which, execute the passed action, and if an expection is detected, do a `sleep` and retry.

```javascript retry function
var retry = function (action, options) {

    retry.DEFAULT_OPTIONS = {
        maxRetry : 3,
        interval : 500,
        intervalMultiplicator : 1.5
    };

    if (typeof action !== "function") {
        throw new Error("action must be a function");
    }
    if (!options) {
        options = retry.DEFAULT_OPTIONS;
    }
    else {
        for (var k in retry.DEFAULT_OPTIONS) {
            if (retry.DEFAULT_OPTIONS.hasOwnProperty(k) && !(k in options)) {
                options[k] = retry.DEFAULT_OPTIONS[k];
            }
        }
    }

	var resolver = function(remainingTry, interval) {
        var result = toAsync(action);
        if (remainingTry <= 1) {
            return result;
        }
        return result.catch(function (e) {
            return sleep(interval).then(function () {
            	// recursion
                return resolver(remainingTry - 1, interval * options.intervalMultiplicator);
            });
        });
    }
    return resolver(options.maxRetry, options.interval);
};
```

Here are the unit tests:
```javascript retry function unit tests
it("retry_with_invalid_parameter_function_should_throw_exception", function () {
    expect(function () {
        service.retry("invalid");
    }).toThrow(new Error("action must be a function"));

});

it("retry_with_faulty_sync_function_should_succeed", function (done) {
    spyOn(mockHelper, 'faultyFn').and.callThrough();
    var action = function () {
        return mockHelper.faultyFn();
    };
    var promise = service.retry(action);
    promise.then(null, function (rejection) {
        expect(mockHelper.faultyFn).toHaveBeenCalled();
        expect(mockHelper.faultyFn.calls.count()).toBe(3);
        expect(rejection.message).toBe("I'm a faulty function");
        done();
    });
    $rootScope.$apply();

});

it("retry_with_faulty_sync_function_and_options_should_succeed", function (done) {
    spyOn(mockHelper, 'faultyFn').and.callThrough();
    var action = function () {
        return mockHelper.faultyFn();
    };
    var promise = service.retry(action, {maxRetry : 5});
    promise.then(null, function (rejection) {
       
        expect(mockHelper.faultyFn).toHaveBeenCalled();
        expect(mockHelper.faultyFn.calls.count()).toBe(5);
        expect(rejection.message).toBe("I'm a faulty function");
        done();
    });
    $rootScope.$apply();

});

it("retry_with_valid_sync_function_should_succeed", function (done) {
    spyOn(mockHelper, 'addOne').and.callThrough();
    var action = function () {
        return mockHelper.addOne(100);
    };
    var promise = service.retry(action);
    promise.then(function (result) {
        expect(result).toBe(101);
        expect(mockHelper.addOne).toHaveBeenCalled();
        expect(mockHelper.addOne.calls.count()).toBe(1);
        done();
    });
    $rootScope.$apply();
});

it("retry_with_valid_async_function_should_succeed", function (done) {
    spyOn(mockHelper, 'getUrl').and.callThrough();
    $httpBackend.when('GET', '/dummy').respond(mockHelper.dummyResponse);
    var action = function () {
        return mockHelper.getUrl();
    };
    var promise = service.retry(action);
    promise.then(function (result) {
        expect(mockHelper.getUrl).toHaveBeenCalled();
        expect(result.data).toEqual(mockHelper.dummyResponse);
        done();
    });
    $httpBackend.flush();
    $rootScope.$apply();
});
```

That's it. Our service is fully unit tested (100% coverage!!!), and we can reuse it anywhere in our applications.


The repository is available [here][1].   


Let me know what you think and happy coding.

Avi Haiat


[1]: https://github.com/thaiat/angular-retry