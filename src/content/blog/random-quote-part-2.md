---
author: Ryan Jordan
pubDatetime: 2015-11-30T12:56:00Z
title: Random Quote Part II
slug: random-quote-part-2
draft: false
description: In part II of this tutorial, we will get this random quote generator working with some jQuery. If you haven't read [part I](/post/random-quote-part-I/), I would suggest doing so now as I won't be covering any styling in this part of the tutorial.
---

In part II of this tutorial, we will get this random quote generator working with some jQuery. If you haven't read [part I](/post/random-quote-part-I/), I would suggest doing so now as I won't be covering any styling in this part of the tutorial.

<!--more-->

So the html looks like this:

```html
<div class="container">
  <h2>Random Quote Generator</h2>
  <a href="#" id="btn-quote">Generate New Quote</a>
  <blockquote>
    <p id="quoteText"></p>
    <cite
      >&mdash; <span id="quoteAuthor"></span>
      <span id="btn-tweet">
        <i class="fa fa-twitter"></i>
      </span>
    </cite>
  </blockquote>
  <div class="load">
    <i class="fa fa-spinner fa-spin"></i>
  </div>
</div>
<footer>
  Powered by&nbsp; <a href="http://forismatic.com/en/">Forismatic</a>
</footer>
<script src="http://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<script src="main.js"></script>
```

Notice how we have added the <code class="language-none">script</code> tags at the end of the <code class="language-none">body</code>. Also notice that in order to use jQuery, we have to reference it in our HTML. So, we add the following to our HTML:

```html
<script src="http://cdnjs.cloudflare.com/ajax/libs/jq<uery/2.1.3/jquery.min.js"></script>
```

## jQuery

It's time to add some jQuery. I've created a file called <code class="language-none">main.js</code> in the root directory, to make things easier, and added:

```html
<script src="main.js"></script>
```

This tells the browser that we have an exterior JavaScript file it will need to look for.

Currently, if you followed along with the first tutorial, we have the following in our <code class="language-none">main.js</code> file.

```javascript
$(document).ready(function () {
  // Hide the loading icon
  $(".load").hide();
});
```

This hides the loading icon once the page is loaded. The <code class="language-javascript">$('.load')</code> finds the elements with the class of <code class="language-none">load</code> and hides them with jQuery's <code class="language-javascript">.hide()</code> method. For the time being, lets remove the <code class="language-javascript">$('.load').hide();</code> so that our file now looks like:

```javascript
$(document).ready(function () {});
```

Now we want to hide the <code class="language-none">blockquote</code> element and show the loading icon as the quote is loading. So we will add the following code.

```javascript
$(document).ready(function () {
  $("blockquote").hide();
  $(".load").show();
});
```

Once again, when the page loads, the browser locates the <code class="language-none">blockquote</code> element and hides it with the <code class="language-javascript">.hide()</code> jQuery method. Then it finds the elements with the <code class="language-none">load</code> class and makes it visible with the <code class="language-javascript">.show()</code> method. Now the <code class="language-none">blockquote</code> should be hidden and the loading icon should be displayed.

## Get the quote

Let's write the function that gets the quote from [forismatic](http://forismatic.com/en/).

```javascript
$(document).ready(function () {
  // Hide the empty blockquote and show the loading icon.
  $("blockquote").hide();
  $(".load").show();
});

// Get the quote
var getQuote = function () {};
```

Inside the <code class="language-javascript">getQuote()</code> function we want to do the following:

- Hide the loading icon
- Show the blockquote element
- Add the quote to the blockquote
- If there is an author, display the author

We will use jQuery's <code class="language-javascript">$.ajax()</code> method to accomplish this.

So let's get started (<code class="language-none">main.js</code>):

```javascript
.
.
.
var getQuote = function() {
  $.ajax({
    url: '',
    dataType: '',
    // If the request is successful
    success: function (data){
      // Hide the loading icon
      // Show the blockquote element
      // Add the quote to the blockquote
      // If there is an author, display the author
    },
    // Set up a way to gracefully handle errors
    error: function (xhr, status, error){
      console.log('xhr: ' + xhr + '\nStatus: ' + status + '\nError: ' + error);
    }
  });
};
```

The <code class="language-javascript">.ajax()</code> method needs a few things before it can do anything for us. It will need the <code class="language-none">url</code> that it needs to request the data from. It will need the <code class="language-none">dataType</code> of the data we are requesting. It uses the <code class="language-none">success</code> method to return the data to us in the way we specify and the <code class="language-none">error</code> method to gracefully tell us something went wrong. Visiting [forismatic.com/en/api](http://forismatic.com/en/api), will give us an idea of what query parameters are needed to make a request. We will need to have <code class="language-none">http://api.forismatic.com/api/1.0/</code> for starters. We will use their getQuote method <code class="language-none">?method=getQuote</code> along with a format and language. Our <code class="language-none">url</code> will be <code class="language-none">http://api.forismatic.com/api/1.0?method=getQuote&format=jsonp&lang=en</code>. Notice the query parameters are separated by an ampersand (&).

We can test our api call by logging the data out to the console.

```javascript
var getQuote = function () {
  $.ajax({
    url: "http://api.forismatic.com/api/1.0/?method=getQuote&format=jsonp&lang=en",
    dataType: "jsonp",
    // If the request is successful
    success: function (data) {
      console.log(data);
    },
    error: function (xhr, status, error) {
      console.log("xhr: " + xhr + "\nStatus: " + status + "\nError: " + error);
    },
  });
};
```

We can test this by opening the <code class="language-none">index.html</code> in Chrome, opening the Developer tools and clicking on the <code class="language-none">Console</code>. We can call the <code class="language-javascript">getQuote();</code> function by typing it into the console. It will return a somewhat uninformative error. However, looking over the documentation again, you might notice that since we are using <code class="language-none">jsonp</code> we will also need to have the jsonp query parameter. So when we add <code class="language-none">&jsonp=?</code> at the end of our <code class="language-none">url</code>, it should work. The full url will look like <code class="language-none">http://api.forismatic.com/api/1.0/?method=getQuote&format=jsonp&lang=en&jsonp=?</code>. When we call this is the console, it should return a quote object.

## Looking at the quote object

In the console of Chrome dev tools, you should see something like:

```javascript
getQuote();
undefined

Object {
  quoteText: "Time is the most valuable thing a man can spend. ",
  quoteAuthor: "Theophrastus ",
  senderName: "",
  senderLink: "",
  quoteLink: "http://forismatic.com/en/2c9cbc5e2a/"
}
```

This is how we will access the data inside the object:

```javascript
console.log(data.quoteText);
console.log(data.quoteAuthor);
```

Now that we know how to access the data inside the object, lets put it all together.

```javascript
var getQuote = function () {
  $.ajax({
    url: "http://api.forismatic.com/api/1.0/?method=getQuote&format=jsonp&lang=en&jsonp=?",
    dataType: "jsonp",
    // If the request is successful
    success: function (data) {
      // Hide the loading icon
      $(".load").hide();
      // Show the blockquote element
      $("blockquote").show();
      // Add the quote to the #quoteText element
      $("#quoteText").html(data.quoteText);
      if (data.quoteAuthor !== "") {
        // If there is an author
        $("#quoteAuthor").html(data.quoteAuthor);
      } else {
        // When there is no author
        $("#quoteAuthor").html("Unknown");
      }
    },
    // Set up a way to gracefully handle errors
    error: function (xhr, status, error) {
      $("#quoteText").text(
        "I'm not sure what happened there. Click again and let's see if that does the trick!"
      );
      $("#quoteAuthor").text("Your Trusty Browser");
    },
  });
};
```

And there you have it. The only thing new with this code is the <code class="language-javascript">html()</code> method which just sets the html content for the element that was called. Now we will add an event listener and call it a day.

```javascript
$(document).ready(function()({
  // Hide the empty blockquote and show the loading
  // icon as we request another quote.
  $('blockquote').hide();
  $('.load').show();
  getQuote();

  // Do the same as above when the button is clicked.
  $('#btn-quote').on('click', function(){
    $('blockquote').hide();
    $('.load').show();
    $getQuote();
  });
});

// Get the quote
var get Quote = function(){
.
.
.
};
```

That concludes part II of this tutorial. jQuery can make manipulating the DOM a lot easier than regular JavaScript. As always, feel free to comment and let me know what you think and ask any questions you might have.
