# Create a jQuery Plugin

##Example
[jsfiddle](https://jsfiddle.net/fiffty/k57vht4u/)

##Key Learning points
The goal of this module is to review the following topics: 
* Self-executing anonymous functions 
* Object oriented javascript

##Basics of jQuery methods

Let's first understand how jQuery functions work. Whenever you use `$` to select DOM elements, it returns a jQuery obejct. That object will have access to all jQuery methods. Let's look at the code below.
```javascript
$('body').css('background','#ddd');
```

`$('body')` returns a jQuery object which contains the method `.css()`. We have been using these built-in jQuery methods all along, such as `.addClass()`, `.click()`, etc. To write our own jQuery methods, we need to know how a jQuery object gets these methods. The object `$.fn` is what provides the link between a jQuery object and its methods.
```javascript
$.fn.greyBackground = function() {
	$('body').css('background', '#ddd');
};

$().greyBackground();
```


##Self-executing anonymous functions

Your jQuery plugin will most likely be longer than a few lines of code. So we will encapsulate the plugin within a self-executing anonymous function. The reason behind this is to make the code more modular. By wrapping our code this way, we:
* Create a scope just for the plugin, so that the variables used within will not be in conflict with any code outside of it.
* This is especially important for jQuery, as jQuery's `$` notation is commonly used among other javascript libraries.
```javascript
(function($) {
	$.fn.greyBackground = function() {
		$('body').css('background', '#ddd');
	};
})(jQuery);

$().greyBackground();
```

This can be a little confusing at first glance, so let's break it up. We understand that we can write anonymous functions the following way:
```javascript
function() { // do something }
```

This on its own is quite meaningless, as it doesn't get executed when it's read, and neither do we have access to it in any other parts of our code. So we often store anonymous functions in variables, such as:
```javascript
var myFunction = function() { // do something };

myFunction();
```

However, there is a way to execute an anonymous function without storing it into a variable. First, you wrap the entire function in parentheses, `(function() { // do something })`, then execute it using bracket notations at the end `(function() { // do something })()`.
Now let's revisit our code from above. In
```javascript
function($) {
	$.fn.alert = function(str) {
		alert(str);
	};
}
```
the `$` is a parameter being passed into the anonymous function. On its own, it is NOT the jQuery notation. However, once you wrap the entire anonymous function with a parentheses, then execute the function with the object `jQuery` as the corresponding parameter, you are making sure that all `$` notations within that function points to jQuery. The code below is pretty much the same from our code from above:
```javascript
(function(foo) {
	foo.fn.greyBackground = function() {
		foo('body').css('background', '#ddd');
	}
})(jQuery);
```

##Adding features and options

Let's now write some code that is a little bit more interesting.
```javascript
(function($) {

	$.fn.noisyBtn = function() {
		var self = $(this);
		self.hover(function() {
			self.append('ha');
		});
		self.click(function() {
			alert(self.html());
		});
	}

})(jQuery);
```

We are creating a jQuery method called <code>.noisyBtn</code> that makes the DOM element respond to hover and click events in a certain way. The line `var self = $(this)` captures the jQuery object for the DOM element so that we can use methods such as `.hover` and `.append` on it. We can test whether our plugin works by creating a HTML element with a class named 'noisyBtn' and executing the method on it like: `$('.noisyBtn').noisyBtn();`
Now let's try adding some options for our plugin.
```javascript
(function($) {

	$.fn.noisyBtn = function(customOptions) {
		var options = {'text-transform': 'none'}
		$.extend(options, customOptions);

		var self = $(this);
		self.css(options);

		self.hover(function() {
			self.append('ha');
		});
		self.click(function() {
			alert(self.html());
		})
	}

})(jQuery);
```

First, we've added `customOptions` as a parameter for our jQuery method. Then we defined the default options with `var options = {'text-transform': 'none'}` and let it be overwritten if the `customOptions` parameter exists with `$.extend`. We then apply the options with `self.css(options)`. Now we can create a noisyBtn in all caps by doing `$('.noisyBtn').noisyBtn({'text-align': 'uppercase'})`

##Object oriented javascript

Rewriting our code in an OOP fashion provides us with many benefits. Namely, it lets us easily attach additional functions to our custom jQuery method. First, we will create a constructor function within our anonymous function.
```javascript
var NoisyBtn = function() {
	this.init = function(elem, options) {
		return this;
	}
}	
```

This way, our jQuery method will simply create a new instance of the object, and most of the features will be coded within the `.init` function inside the constructor.
```javascript
$.fn.noisyBtn = function(options) {
	return (new NoisyBtn).init(this, options);
}		
```

Notice how `$.fn.noisyBtn()` creates a new instance with `(new NoisyBtn)`, then executes the function `.init` on it right away, with the parameters `this` and `options`. This way the the constructor function has access to the DOM element that our jQuery method is being applied on and the options we are putting in. With this, we will beef up our `.init` function to do what it's supposed to.
```javascript
var NoisyBtn = function() {
	this.options = {
		'text-transform': 'capitalize'
	};

	self.init = function(elem, options) {
		$.extend(this.options, options);
		elem.css(this.options);
		elem.hover(function(){
			elem.append('ha');
		});
		elem.click(function() {
			alert(elem.html());
		});
		return this;
	}
}
```

As our last step, let's add a function to our object. Perhaps our noisyBtn can be too annoying at times, and we want a way to make it shut up. Inside our constructor, we will write a method `.shutup`, so that when it's executed, all listeners will be dropped. 
First, we need a way to store the HTML element that we are running our jQuery method on. Currently we have access to it in our `.init` function passed in as a parameter called `elem`. To make this avaialbe outside the `.init` function, we will first stage the instance of the NoisyBtn object in a variable.
```javascript
var NoisyBtn = function() {
	var self = this;
	...
}
```

This way we can reference it within other scope such as `.init` function. And with it, we will store the HTML element.
```javascript
var NoisyBtn = function() {
	var self = this;
	self.options = {
		'text-transform': 'capitalize'
	};

	self.init = function(elem, options) {
		self.elem = elem;
		...
	}	
}
```

Finally, we can creat a function that takes the `self.elem` and do something with it.
```javascript
var NoisyBtn = function() {
	...
	self.shutup = function() {
		self.elem.off();
	}	
}
```

Now when we initiate our jQuery plugin, we will store it in a variable like so: `var noisyBtn = $('.noisyBtn').noisyBtn();`, so that we can use our `.shutup` function to disable the noisyBtn: `noisyBtn.shutup();`



