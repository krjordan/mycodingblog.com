---
author: Ryan Jordan
pubDatetime: 2015-12-29T13:01:00Z
title: Node Email Templates
slug: node-email-templates
draft: false
tags:
  - node
  - email
description: I have been looking for a straight forward way to use email templates in a contact form. I have searched and searched with no real success. So, I thought I would write a tutorial on how to build a (very!) simple contact page that uses an email template. This uses [Express](http://expressjs.com/), [Nodemailer](http://nodemailer.com/) and [node-email-templates](https://github.com/niftylettuce/node-email-templates).
---

I have been looking for a straight forward way to use email templates in a contact form. I have searched and searched with no real success. So, I thought I would write a tutorial on how to build a (very!) simple contact page that uses an email template. This uses [Express](http://expressjs.com/), [Nodemailer](http://nodemailer.com/) and [node-email-templates](https://github.com/niftylettuce/node-email-templates).

<!--more-->

## Basic File Structure

```yaml
├── app
├── css
└── main.css
├── templates
└── contact-request
├── html.hbs
├── style.css
└── text.hbs
└── index.html
├── package.json
└── server.js
```

We will start out by adding [Handlebars](http://handlebarsjs.com/), [express](http://expressjs.com/) and [email-templates](https://github.com/niftylettuce/node-email-templates):

```none
npm init
```

Feel free to fill in the information or press enter until it is complete. I have my <code class="language-none">entry-point</code> set as <code class="language-none">server.js</code>.

```none
npm install -S express email-templates handlebars
```

This will install those packages and add them to our dependencies in the <code class="language-none">package.json</code> file.

## Basic Styling and Markup

```html
<!-- app/index.html -->
<body>
  <div class="alert"></div>
  <input type="text" id="name" name="name" placeholder="Name" />
  <input type="email" name="email" id="email" placeholder="Email" />
  <textarea
    name="message"
    id="message"
    cols="30"
    rows="10"
    placeholder="Enter your message."
  ></textarea>
  <button id="send-contact" type="submit">Send</button>
</body>
```

```css
/* app/css/main.css */
* {
  box-sizing: border-box;
}
body {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  padding-top: 50px;
  background-color: #ccc;
}
input {
  margin-bottom: 20px;
  width: 300px;
  height: 25px;
  font-size: 16px;
  color: #3e3e3e;
}
textarea {
  font-size: 16px;
  width: 300px;
  margin-bottom: 20px;
  color: #3e3e3e;
}
button {
  padding: 15px 25px;
  font-size: 16px;
  cursor: pointer;
  color: #3e3e3e;
}
button:hover {
  background-color: #3e3e3e;
  color: #fefefe;
}
.alert {
  display: flex;
  align-items: center;
  justify-content: center;
  text-shadow: 0 -1px 0 rgba(0, 0, 0, 0.25);
  color: #fff;
  margin: 20px;
  padding: 15px;
  border-radius: 5px;
}
.error {
  background-color: #c43c35;
  background-repeat: repeat-x;
  background-image: linear-gradient(top, #ee5f5b, #c43c35);
  border-color: #c43c35 #c43c35 #882a25;
}
.success {
  background-color: #57a957;
  background-repeat: repeat-x;
  background-image: linear-gradient(top, #62c462, #57a957);
  border-color: #57a957 #57a957 #3d773d;
}
```

## Setting up Express

Since we are using express, it's pretty simple to get our server going. You can follow the [getting started](http://expressjs.com/en/starter/installing.html) and [hello world](http://expressjs.com/en/starter/hello-world.html) quick tutorials. Below is what our base <code class="language-none">server.js</code> file will currently look like.

```javascript
// server.js
var express = require("express");
var app = express();

app.use(express.static("app"));

var server = app.listen(8080, function () {
  var host = "localhost";
  var port = server.address().port;

  console.log("Example app listening at http://%s:%s", host, port);
});
```

Now you should be able to view the <code class="language-none">index.html</code> contact page in your browser (<code class="language-none">http://localhost:8080</code>) by typing <code class="language-none">node server.js</code> in your terminal.

## Setting up Nodemailer

So we are using [Nodemailer](http://nodemailer.com/) to help us send our emails. In order to use the package, we will need to add it to our dependencies and install it:

```none
npm install -S nodemailer@0.7.1
```

To get a better understanding of [Nodemailer](http://nodemailer.com/), I would encourage you to read through the documentation as I will just highlight the basics to get our simple contact page up and running. We will use the basic example given by [Nodemailer](http://nodemailer.com/) in our project.

```javascript
// server.js
var express = require("express");
var app = express();
var nodemailer = require("nodemailer");

app.use(express.static("app"));

// nodemailer
var smtpTransport = nodemailer.createTransport("SMTP", {
  service: "Gmail",
  auth: {
    user: "user@gmail.com",
    pass: "password",
  },
});

var mailOptions = {
  from: "Fred Foo <foo@blurdybloop.com>", // Sender Address
  to: "user@gmail.com",
  subject: "Contact Request", // Subject Line
  text: "Hello World!", // Plaintext Body
  html: "<b>Hello Word! </b>", // HTML Body
};

smtpTransport.sendMail(mailOptions, function (error, response) {
  if (error) {
    console.log(error);
  } else {
    console.log("Message sent: " + response.message);
  }
});

var server = app.listen(8080, function () {
  var host = "localhost";
  var port = server.address().port;

  console.log("Example app listening at http://%s:%s", host, port);
});
```

Now when you startup your server, <code class="language-none">node server.js</code>, you should receive an email (assuming you added your email address and password to the <code class="language-none">user:</code> and <code class="language-none">pass:</code> respectively.

Now that we know it works, we need to wrap <code class="language-none">mailOptions</code> and <code class="language-javascript">smtpTransport.sendMail</code> with a <code class="language-javascript">get()</code> method so we don't get an email every time we start up our app.

```javascript
// server.js
var express = require("express");
var app = express();
var nodemailer = require("nodemailer");

app.use(express.static("app"));

// nodemailer
var smtpTransport = nodemailer.createTransport("SMTP", {
  service: "Gmail",
  auth: {
    user: "user@gmail.com",
    pass: "password",
  },
});

app.get("/sendContact", function (req, res) {
  var mailOptions = {
    from: "Fred Foo <foo@blurdybloop.com>", // Sender Address
    to: "user@gmail.com",
    subject: "Contact Request", // Subject Line
    text: "Hello World!", // Plaintext Body
    html: "<b>Hello Word! </b>", // HTML Body
  };

  smtpTransport.sendMail(mailOptions, function (error, response) {
    if (error) {
      console.log(error);
    } else {
      console.log("Message sent: " + response.message);
    }
  });
});

var server = app.listen(8080, function () {
  var host = "localhost";
  var port = server.address().port;

  console.log("Example app listening at http://%s:%s", host, port);
});
```

## Using the Form Data

Next we will want to use our form to capture the data the user will input. For simplicity, I'll add jQuery to our project and add another file called <code class="language-none">main.js</code> to <code class="language-none">app/scripts/main.js</code>. I'll add jQuery by inserting it at the bottom of the <code class="language-html">\<body\></code> inside the <code class="language-none">index.html</code> with <code class="language-html">\<script src="https://code.jquery.com/jquery-2.1.4.min.js"\></code><code class="language-html">\</script\></code> along with <code class="language-none">\<script src="scripts/main.js"\></code><code class="language-html">\</script\></code> just below it. Next we'll create the <code class="language-none">main.js</code> file.

```javascript
// app/scripts/main.js

$("#send-contact").click(function (evt) {
  evt.preventDefault();
  var name = $("#name").val();
  var email = $("#email").val();
  var message = $("#message").val();

  if (name !== "" && email !== "" && message != "") {
    $.get(
      "http://localhost:8080/sendContact",
      {
        name: name,
        email: email,
        message: message,
      },
      function (data) {
        if (data == "sent") {
          $(".alert")
            .html("<strong>Success! </strong> Your message has been sent!")
            .addClass("success");
        } else {
          $(".alert")
            .html(
              "<strong>Uh oh! </strong> It looks like that didn't go through, let's try that again."
            )
            .addClass("error");
        }
      }
    );
  }
});
```

## Add the Email Templates to Server.js

We will now add node-email-templates to our server.js file. For completeness, I have added the entire server.js file:

```javascript
var express = require("express");
var app = express();
var nodemailer = require("nodemailer");
var path = require("path");
var EmailTemplate = require("email-templates").EmailTemplate;

var templatesDir = path.resolve(__dirname, "app/templates");

app.use(express.static("app"));

// nodemailer
var smtpTransport = nodemailer.createTransport("SMTP", {
  service: "Gmail",
  auth: {
    user: "user@gmail.com",
    pass: "password",
  },
});

app.get("/sendContact", function (req, res) {
  var template = new EmailTemplate(path.join(templatesDir, "contact-request"));
  var locals = {
    email: req.query.email,
    name: req.query.name,
    message: req.query.message,
  };

  template.render(locals, function (err, results) {
    if (err) {
      return console.error(err);
    }

    smtpTransport.sendMail(
      {
        from: locals.email,
        to: "user@gmail.com",
        subject: "Contact Request",
        html: results.html,
        text: results.text,
      },
      function (err, responseStatus) {
        if (err) {
          console.error(err);
          res.end("error");
        } else {
          console.log(responseStatus.message);
          res.end("sent");
        }
      }
    );
  });
});

var server = app.listen(8080, function () {
  var host = "localhost";
  var port = server.address().port;

  console.log("Example app listening at http://%s:%s", host, port);
});
```

## Email Templates

Our emails should now be sending, but in order to test them, we will need to add our templates. Unfortunately, with node-email-templates, there isn't a way for us to easily style the templates without sending an email to ourselves. But to help shorten this tutorial, I'll add some styling to our templates so we can test our email and wrap this tutorial up.

We will need to add an <code class="language-none">html.hbs</code> and <code class="language-none">text.hbs</code> to our <code class="language-none">templates/contact-request</code> directory. We have the <code class="language-none">text.hbs</code> for better support.

```html
<!-- app/templates/contact-request/html.hbs -->

<div class="container">
  <h3 class="center">Contact Request</h3>
  <p class="center">
    {{ name }} has filled out the contact form on your website with the
    following message:
  </p>

  <blockquote>
    <p>{{ message }}</p>
  </blockquote>

  <p class="center small">
    To respond to {{ name }}, you may reply directly to this email or send an
    email to {{ email }}.
  </p>
</div>
```

```html
<!-- app/templates/contact-request/text.hbs -->

Contact Request {{ name }} has filled out the contact form on your website with
the following message: {{ message }} To respond to {{ name }}, you may reply
directly to this email or send an email to {{ email }}.
```

```css
* {
  box-sizing: border-box;
}
.container {
  border: 1px solid #e5e3d8;
  padding: 20px;
  border-radius: 5px;
}
h3,
p {
  color: #808284;
  font-family: sans-serif;
  line-height: 1.5em;
}
h3 {
  font-size: 16px;
}
p {
  font-size: 15px;
}
blockquote {
  background: #f9f9f9;
  border-left: 10px solid #ccc;
  margin: 1.5em 10px;
  padding: 0.5em 10px;
}
blockquote p {
  display: inline;
}
.center {
  text-align: center;
}
.small {
  font-size: 10;
}
```

This should give us a nice template to use as a starting point. There is still plenty of work that needs to be done (form validation for starters), but this concludes this tutorial. You should have a working contact form with a nicely styled template. There are other options out there like [Swig](https://github.com/andrewrk/swig-email-templates) that allow us to view the changes to the template in a browser. I hope to have a tutorial of that up soon.

As always, feedback is welcome and if you run into any problems or have any questions, let me know in the comments below and I will try my best to help you out. I have added the code used in this tutorial to [github](https://github.com/krjordan/node-email) in case anyone wanted to play around with it or use it as a starting point.
