Web application frameworks, or simply "web frameworks", are the de facto way to
build web-enabled applications. From simple blogs to complex AJAX-rich applications, every
page on the web was created by writing code. I've recently found that
many developers interested in learning a web framework like Flask or Django
don't really understand what a web framework *is*, what their purpose is, or how they
work. In this article, I'll explore the oft-overlooked topic of web framework
fundamentals. By the end of the article, you should have a solid understanding
of what a web framework is and why they exist in the first place.
This will make it *far* easier to learn a new web framework and make an informed
decision regarding which framework to use.
<!--more-->

## How The Web Works

Before we talk about frameworks, we need to understand how the web "works". To
do so, we'll delve into what happens when you type a URL into your browser and
hit `Enter`. Open a new tab in your browser and navigate to 
[http://www.jeffknupp.com](http://www.jeffknupp.com). Let's talk about 
the steps your browser took in order to display the page (minus DNS lookups).

### Web Servers and ... web ... servers...

Every web page is transmitted to your browser as `HTML`, a language used by
browsers to describe the content and structure of a web page. The application
responsible for sending `HTML` to browsers is called a *web server*.
Confusingly, the machine this application resides on is also usually called a
web server. 

The important thing to realize, however, is that at the end of the
day, all a web application really does is send `HTML` to browsers. No matter how
complicated the logic of the application, the final result is always `HTML`
being sent to a browser (I'm purposely glossing over the ability for
applications to respond with different types of data, like `JSON` or CSS files,
as the concept is the same).

How does the web application know *what* to send to the browser? **It sends
whatever the browser requests**.

### HTTP

Browsers download websites from *web servers* (or "application servers") using
the `HTTP` *protocol* (a *protocol*, in the realm of programming, is a 
universally known data format and sequence of steps enabling communication 
between two parties). The `HTTP` protocol is based on a `request-response` model. 
The client (your browser) *requests* data from a web application that resides 
on a physical machine. The web application in turn *responds* to the request with 
the data your browser requested.

An important point to remember is that communication is always initiated by the
*client* (your browser). The *server* (web server, that is) has no way of
initiating a connection to you and sending your browser unsolicited data. If you
receive data from a web server, it is because your browser explicitly asked for
it.

#### HTTP Methods

Every message in the `HTTP` protocol has an associated *method* (or *verb*). The various `HTTP` methods 
correspond to logically different types of requests the client can send, which in turn
represent different intentions on the client side. Requesting the HTML 
of a web page, for example, is logically different than submitting a form, so the 
two actions require the use of different methods.

##### HTTP GET

The `GET` method does exactly what it sounds like: gets (requests) data from the
web server. `GET` requests are the by far the most common `HTTP` request. During 
a `GET` request the web application shouldn't need to do anything more than 
respond with the requested page's HTML. Specifically, the web application should not 
alter the state of the application as a result of a `GET` request (for example,
it should not create a new user account based on a `GET` request). For 
this reason, `GET` requests are usually considered "safe" since they don't
result in changes to the application powering the website.

##### HTTP POST

Clearly, there is more to interacting with web sites than simply looking at
pages. We are also able to *send* data to the application, e.g. via a form. 
To do so, a different type of request is required: `POST`. `POST` requests
usually carry data entered by the user and result in some action being taken
within the web application. Signing up for a web site by entering your
information on a form is done by `POST`ing the data contained in the form to the
web application.

Unlike a `GET` request, `POST` requests usually result in the state of the
application changing. In our example, a new user account is created when the
form is `POST`ed. Unlike `GET` requests, `POST` requests do not always result in
a new HTML page being sent to the client. Instead, the client uses the response's
*response code* do determine if the operation on the application was successful.

#### HTTP Response Codes

In the normal case, a web server returns a *response code* of 200, meaning, "I did
what you asked me to and everything went fine". *Response codes* are always a 
three digit numerical code. The web applications must send one with each
response to indicate what happened as a result of a given request. The response code `200`
literally means "OK" and is the code most often used when responding to a `GET`
request. A `POST` request, however, may result in code `204` ("No Content")
being sent back, meaning "Everything went OK but I don't really have anything to
show you."

It's important to realize that `POST` requests are still sent
to a specific URL, which may be different from the page the data was submitted
from. Continuing our sign up example, the form may reside at
`www.foo.com/signup`. Hitting `submit`, however, may result in a `POST` request
with the form data being sent to `www.foo.com/process_signup`. The location a
`POST` request should be sent to is specified in the form's `HTML`.

## Web Applications

You can get quite far using only `HTTP` `GET` and `POST`, as they're the two most
common `HTTP` methods by a wide margin. A web application, then, is responsible
for receiving an `HTTP` request and replying with an `HTTP` response, usually
containing HTML that represents the page requested. `POST` requests cause the
web application to take some action, perhaps adding a new record in the
database. There are a number of other `HTTP` methods, but we'll focus on `GET` and
`POST` for now.

What would the simplest web application look like? We could write an application
that listened for connections on port `80` (the well-known `HTTP` port that
almost all `HTTP` traffic is sent to). Once it received a connection it would
wait for the client to send a request, then it might reply with some very simple
HTML.

Here's what that would look like:
    
    #!py
    import socket

    HOST = ''
    PORT = 80
    listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    listen_socket.bind((HOST, PORT))
    listen_socket.listen(1)
    connection, address = listen_socket.accept()
    request = connection.recv(1024)
    connection.sendall("""HTTP/1.1 200 OK
    Content-type: text/html


    <html>
        <body>
            <h1>Hello, World!</h1>
        </body>
    </html>""")
    connection.close()

(If the above doesn't work, try changing the `PORT` to something like `8080`)

This code accepts a single connection and a single request. Regardless of what
URL was requested, it responds with an `HTTP 200` response (so it's not *really* a
web server). The `Content-type: text/html` line represents a *header* field.
*Headers* are used to supply meta-information about the request or response.
In this case, we're telling the client that the data that follows
is HTML (rather than, say, JSON).

### Anatomy of a Request

If I look at the `HTTP` request I sent to test the program above, I find it looks
quite similar to the response. The first line is `<HTTP Method> <URL> <HTTP version>`
or, in this case, `GET / HTTP/1.1`. After the first line come a few headers like `Accept: */*`
(meaning we will accept any type of content in a response). That's basically it.

The reply we send has a similar first request line, in the format `<HTTP version> <HTTP
Status-Code> <Status-Code Reason-Phrase>` or `HTTP/1.1 200 OK` in our case. Next
come headers, in the same format as the request headers. Lastly, the actual
content of the response is included. Note that this can be encoded as a string
or binary object (in the case of files). The `Content-type` header lets the
client know how to interpret the response.

### Web Server Fatigue

If we were going to continue building on the example above as the basis for a
web application, there are a number of problems we'd need to solve:

1. How do we inspect the requested URL and return the appropriate page?
1. How do we deal with `POST` requests in addition to simple `GET` requests
1. How do we handle more advanced concepts like sessions and cookies?
1. How do we scale the application to handle thousands of concurrent connections?

As you can imagine, no one wants to solve these problems each time they build a
web application. For that reason, packages exist that handle the nitty-gritty
details of the `HTTP` protocol and have sensible solutions to problems
the problems above. Keep in mind, however, at their core they function in much 
the same way as our example: listening for requests and sending `HTTP` responses with 
some HTML back.

*Note that **client-side** web frameworks are a much different beast and deviate significantly from the above description.*

## Solving The Big Two: Routing and Templates

Of all the issues surrounding building a web application, two stand out.

1. How do we map a requested URL to the code that is meant to handle it?
1. How do we create the requested HTML dynamically, injecting calculated values
   or information retrieved from a database?

Every web framework solves these issues in some way, and there are many
different approaches. Examples will be helpful, so I'll discuss Django 
and Flask's solutions to both of these problems. First, though, we need 
to briefly discuss the *MVC* pattern.

### MVC in Django

Django makes use of the *MVC* pattern and requires code using the framework
to do the same. *MVC*, or "Model-View-Controller" is simply a way of logically
separating the different responsibilities of the application. Resources like
database tables are represented by *models* (in much the same way a `class` in
Python often models some real-world object). *controllers* contain the business
logic of the application and operate on models. *Views* are given all of
the information they needs to dynamically generate the HTML representation of the page.

Somewhat confusingly, in Django, *controllers* are called *views* and *views*
are called *templates*. Other than naming weirdness, Django is a pretty
straightforward implementation of an *MVC* architecture.

### Routing in Django

*Routing* is the process of mapping a requested URL to the code responsible for
generating the associated HTML. In the simplest case, *all* requests are handled
by the same code (as was the case in our earlier example). Getting a little more
complex, every URL could map 1:1 to a `view function`. For example, we could
record somewhere that if the URL `www.foo.com/bar` is requested, the function
`handle_bar()` is responsible for generating the response. We could build up
this mapping table until all of the URLs our application supports are enumerated
with their associated functions.

However, this approach falls flat when the URLs contain useful data, such as the
ID of a resource (as is the case in `www.foo.com/users/3/`). How do we map that
URL to a view function, and at the same time make use of the fact that we want
to display the user with ID `3`? 

Django's answer is to map URL *regular expressions* to view functions that can
take parameters. So, for example, I may say that URLs that match
`^/users/(?P<id>\d+)/$` calls the `display_user(id)` function where the `id`
argument is the captured group `id` in the regular expression. In that way, any
`/users/<some number>/` URL will map to the `display_user` function. These
regular expressions can be arbitrarily complex and include both keyword and
positional parameters.

### Routing in Flask

Flask takes a somewhat different approach. The canonical method for hooking up
a function to a requested URL is through the use of the `route()` decorator. The
following Flask code will function identically to the regex and function listed 
above:

    #!py
    @app.route('/users/<id:int>/')
    def display_user(id):
        # ...

As you can see, the decorator uses an almost simplified form of regular expression
to map URLs to arguments (one that implicitly uses `/` as separators). Arguments are 
captured by including a `<name:type>` directive in the URL passed to `route()`. 
Routing to static URLs like `/info/about_us.html` is handled as you would 
expect: `@app.route('/info/about_us.html')`

### HTML Generation Through Templates

Continuing the example above, once we have the appropriate piece of code mapped
to the correct URL, how do we dynamically generate HTML in a way that still
allows web designers to hand-craft it? For both Django and Flask,
the answer is through *HTML templating*.

*HTML Templating* is similar to using `str.format()`: the desired output is written 
with placeholders for dynamic values. These are later replaced by arguments to
the `str.format()` function. Imagine writing an entire web page as a single string, 
marking dynamic data with braces, and calling `str.format()` at the end. 
Both *Django templates* and [jinja2](http://jinja.pocoo.org), the template engine 
Flask uses, are designed to be used in this way.

However, not all templating engines are created equal. While Django has
rudimentary support for programming in templates, Jinja2 basically lets you execute
arbitrary code (it doesn't *really*, but close enough). Jinja2 also aggressively *caches*
the result of rendering templates, so that subsequent requests with the exact
same arguments are returned from the cache instead of expensively being
re-rendered.

### Database Interaction

Django, with its "batteries included" philosophy, includes an `ORM`
("Object Relational Mapper"). The purpose of an `ORM` is two-fold: it maps Python
classes to database tables and abstracts away the differences between various
database engines (though the former is its primary role). No one loves `ORM`s
(because the mapping between domains is never perfect), rather, they are
tolerated. Django's is reasonably full-featured. Flask, being a
"micro-framework", does not include one (though it is quite compatible with
SQLAlchemy, the Django `ORM`'s biggest/only competitor).

The inclusion of an `ORM` gives Django the ability to create a full-featured
`CRUD` application. `CRUD` (**C**reate **R**ead **U**pdate **D**elete)
applications seem to be the sweet spot for web frameworks (on the server side).
Django (and Flask-SQLAlchemy) make the various `CRUD` operations for each model
straightforward.

## Web Framework Round-Up

By now, the purpose of web frameworks should be clear: to hide the boilerplate
and infrastructural code related to handling `HTTP` requests and responses. Just
*how much* is hidden depends on the framework. Django and Flask represent two
extremes. Django includes something for every situation, almost to its
detriment. Flask bills itself as a "micro-framework" and handles the bare
minimum of web application functionality, relying on third-party packages to do
some of the less common web framework tasks.

Remember, though, that at the end of the day, Python web frameworks all work the
same way: they receive `HTTP` requests, dispatch code that generates HTML, and
creates an `HTTP` response with that content. In fact, *all* major server-side
frameworks work in this way (excluding JavaScript frameworks). Hopefully, you're
now equipped to choose between frameworks as you understand their purpose.


http://www.jeffknupp.com/blog/2014/03/03/what-is-a-web-framework/
