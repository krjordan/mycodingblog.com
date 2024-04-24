---
author: Ryan Jordan
pubDatetime: 2016-06-01T15:05:00Z
title: Email Newsletter Adventure
slug: email-newsletter-adventure
draft: false
tags:
  - email
description: I was recently tasked with building out an email newsletter. Honestly, I was dreading the task. Luckily, I did a bit of research before I started out on this adventure and found [Foundation for Emails](http://foundation.zurb.com/emails.html). It's well documented and even has a [CLI](http://foundation.zurb.com/emails/docs/sass-guide.html).
---

I was recently tasked with building out an email newsletter. Honestly, I was dreading the task. Luckily, I did a bit of research before I started out on this adventure and found [Foundation for Emails](http://foundation.zurb.com/emails.html). It's well documented and even has a [CLI](http://foundation.zurb.com/emails/docs/sass-guide.html).

Getting started is as simple as <code class="language-none">npm install -g foundation-cli</code> and then <code class="language-none">foundation new \-\-framework emails</code>. Once complete, you are ready to start coding an email using Inky, Foundation's email templating language. Inky allows you to write `<column>` and `<row>` instead of writing out your emails in a table.

<!--more-->

So basically, Inky turns this:

```html
<container>
  <row>
    <columns>This is a column.</columns>
  </row>
</container>
```

Into this:

```html
<table class="container">
  <tbody>
    <tr>
      <td>
        <table class="row">
          <tbody>
            <tr>
              <th class="small-12 large-12 columns first last">
                <table>
                  <tr>
                    <th>This is a column.</th>
                    <th class="expander"></th>
                  </tr>
                </table>
              </th>
            </tr>
          </tbody>
        </table>
      </td>
    </tr>
  </tbody>
</table>
```

As you can see, this makes things so much easier.

While in development, you run <code class="language-none">npm start</code> to run the gulp tasks that watch and automatically update the browser. Once complete, use <code class="language-none">npm run build</code> to inline all of your CSS into your HTML. Then <code class="language-none">npm run zip</code> to zip the file to get it ready to submit to your email service provider of choice.

This saved me a lot of time and effort and for that I'd like to thank the guys over at [Zurb](http://foundation.zurb.com/)! You should check it out when you get a chance!
