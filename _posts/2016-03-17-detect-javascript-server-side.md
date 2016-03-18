---
layout: post
title: Detect Javascript on the server side
author: soutoner
---

Willing to deploy an awesome feature of your web entirely written in Javascript? 
What if and user comes to your page and has Javascript disabled on the browser?
Let's learn how to detect if our user has Javascript disabled on his browser.

## On the client side

The easiest way is to check on the client side if Javascript is enabled.
If it's not enabled, we may redirect the user to an error page.
How its made? With the `<noscript>` [HTML tag](http://www.w3schools.com/tags/tag_noscript.asp).

{% highlight js %}
<noscript>
    // This code will be executed if user's Javascript is disabled
</noscript>
{% endhighlight %}

What's wrong with this method? This test will be made on the client side,
and as you already know, the user could fool us if, for example,
they delete the code between the `<noscript>` tags.

## On the server side

If you need to ensure that the user is able to run Javascript code in
their browser, you should check it on the server side.
There is no standard way to do this, besides the fact that
it's impossible to test it on the first request to the server.

#### What are we going to do?
 
1. We will force the user to make an **AJAX request** from the client side on the main page. 
2. The endpoint listening to the request, once called, is going to **set a cookie** on the browser.
3. On the desired 'Javascript-tuned' page, we will **look for this cookie**, and if it's not set, we will redirect
the user to our custom error page for our Javascript detractors.

We will have some actors (with HTML, Javascript and PHP for example):

- `index.html`: that we will assume that is the main entry point 
for our site.
- `javascript_enabled.php`: our PHP script that will be listening
to POST requests, waiting to set a cookie called `js` for example.
- `awesome_js_page.php`: the 'Javascripted' page that will look for
the cookie.
- `no-javascript.html`: the redirection will guide the user to this
page, where we'll display some error message.

### Let's draw some code

#### AJAX request

{% highlight html %}
<!-- `index.html` -->
<head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.0/jquery.min.js"></script>
    
    <script>
        $(function() {
            // If the cookie is not already set
            if (document.cookie.indexOf('js') < 0) {
                $.ajax({
                    method: 'POST',
                    url: 'javascript-enabled.php'
                });
            }
        });
    </script>
</head>
{% endhighlight %}

> Note: Remember to include JQuery. We've used it because 'raw' AJAX call
> with javascript are kind of code obfuscation.

We will check if the cookie is set (we don't want to be all the time making
POST request if we've already set it before) and if it's not, we will make
the AJAX call to our endpoint. If Javascript is disabled, no AJAX call will
be made, so there is no cookie around.

You have to be **very careful** at this point, because if your site has more
than one entry point, you may refer to put this code in a different file
and you **should include** it there.

#### Setting the cookie

{% highlight php %}
<?php
// `javascript_enabled.php`

if (!isset($_COOKIE['js'])) {
    $hash = md5($_SERVER['HTTP_USER_AGENT'] . 'is Javascriptable');
    setcookie('js', $hash);
    $_COOKIE['js'] = $hash;
}
{% endhighlight %}

We are going to set a MD5 hash as the value of the cookie so it will be
harder for the user to fake it. Of course this practice won't be useful at all
if you always use the same word for hashing.

> Note: the cookie could be faked even by doing this, there is no perfect method
> to ensure 'the perfect identifier hash of the death' but here you can let your imagination run free.

#### Looking for cookies

Here is the point where the usage of Javascript is unavoidable, so you
should be looking for the `js` cookie right now!

{% highlight php %}
<?php
// `awesome_js_page.php`

$hash = md5($_SERVER['HTTP_USER_AGENT'] . 'is Javascriptable');
if (!isset($_COOKIE['js']) || $_COOKIE['js'] != $hash) {
    header('Location: http://myawsmpage.com/no-javascript.html');
    die();
} 

// Your awesome Javascript is waiting here
{% endhighlight %}

As easy as that, if the cookie is not set or the hash value is not
the desired one, you do a simple redirect to another page (by the way, why you should
be using 
`die()` or `exit()`: [The daily WTF](http://thedailywtf.com/Articles/WellIntentioned-Destruction.aspx)).

### Conclusion

There is no perfect (and when I say perfect, I mean non-hackable-nor-fakeable) method 
to check if the user has Javascript enabled on the browser, even on the sever side.

You should be checking things like:

- Is the user really doing a 'genuine' POST request?
- Does it comes from an AJAX request?
- Do you send things through the POST request (to combine and hash) that can be faked?

Those things are impossible to know sure-fire, so you can complicate this
method until infinity, it depends on your necessity.