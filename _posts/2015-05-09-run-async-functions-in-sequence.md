---
layout: post
title: Run async functions in sequence
comments: true
---

A few times in the past I have had to dynamically create a list of asynchronous operations and run them sequentially.  In this post, I will show how to accomplish this using promises.

The [Q][Q] library shows how to do this.  In summary, when we have an array of functions that return promises, we can do this:

{% highlight javascript linenos %}
var funcs = [f1, f2, f3, f4];

var promise = Q(initialVal);
funcs.forEach(function (f) {
    promise = promise.then(f);
});
return promise;
{% endhighlight %}

What is this doing?  Start by assigning a promise to the promise variable, which resolves immediately.  This is just to avoid null reference on line 5 for the first element in the array.  On line 5 we are saying - when the promise resolves, execute the function f, and the promise of that is assigned back to promise.  This happens for all functions in the funcs array, and the promise of the last function is returned on line 7.

If we did not want to create the first empty promise, we could do this instead:

{% highlight javascript %}
var promise = null;
funcs.forEach(function(f) {
    if (promise == null) {
        promise = f();
    }
    else {
        promise = promise.then(f);
    }
});
return promise;
{% endhighlight %}

Now what if the functions in the array had to be parameterized?  We could wrap them in another function like this:

{% highlight js %}
funcs.push(function() {
    return f1(param1);
});

funcs.push(function() {
    return f1(param2);
});
{% endhighlight %}

Now if we want to generate the function calls in a loop, we need to wrap it in a closure, so that each call preserves its parameter.

{% highlight js %}
var parameters = [1, 2, 3];
for (var i = 0; i < parameters.length; i++) {    
    addToQueue(parameters[i]);
}

function addToQueue(parameter) {
    funcs.push(function() {
        return f1(parameter);
    })
}
{% endhighlight %}

[Q]:      https://github.com/kriskowal/q