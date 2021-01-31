---
layout: post
title: "Passing Data Between HTML & Flask"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,AJAX,JavaScript,jQuery]
image: Passing-Data-Between-HTML-Flask.png
comments: true
---

*"jQuery is a small JavaScript library commonly used to simplify working with the DOM and JavaScript in general. It is the perfect tool to make web applications more dynamic by exchanging JSON between server and client."* - [Armin Ronacher on The Pallets Projects](https://flask.palletsprojects.com/en/1.1.x/patterns/jquery/

## Summary

Running [Flask](https://flask.palletsprojects.com/) for web apps is great - a few lines of code, that are ready to [copy & paste](https://palletsprojects.com/p/flask/) from the Pallets Project site and you're good to go. Add in a choice [bootstrap](https://getbootstrap.com/) CSS and within minutes the terible looking HTML web pages you made as a kid (let alone Myspace!) are a distant memory. However, more often than not you're going to need to work with data that is retrieved from that web page, often by way of a form or bulk upload - what's the best way to do it with Flask? For simple things we can use [query strings](https://en.wikipedia.org/wiki/Query_string) in the *?foo=bar* format, but for more intensive stuff we're going to want to pass [JSON](https://www.w3schools.com/whatis/whatis_json.asp) - think, a .CSV file uploaded to a web browser with a lot of data within could need to be converted to JSON to be worked with in Python. In this post I'm going to write a few methods of passing data and sending responses, like redirects, between Flask and the web browser. 

## My Environment

Coffee: [Bobolink, Brazil from Union Hand-Roasted Coffee](https://unionroasted.com/products/bobolink-brazil)
<br>
Music: [Tigercub](https://open.spotify.com/artist/6ekYAO2D1JkI58CF4uRRqw?si=JfKaaMIeR1CWf1JkXyrPrA)
<br>
OS: Windows 10 Pro v20H2 x64. 
<br>
IDE: [Visual Studio Code v1.52.1](https://code.visualstudio.com/)
<br>
Browser: [Google Chrome v88](https://www.google.com/intl/en_uk/chrome/)

## Tip o' the Hat

The [Flask documentation](https://flask.palletsprojects.com/en/1.1.x/) by Pallets
<br>
Madan Sapkota's [response](https://stackoverflow.com/questions/18118627/redirecting-after-ajax-post) on Stack Overflow
<br>
The AJAX with jQuery [documentation](https://flask.palletsprojects.com/en/1.1.x/patterns/jquery/) on The Pallets Projects.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

To pass data, I'm going to tackle it in three ways - GET with a query string using JavaScript (piccolo), POST with JSON using jQuery (medio) and POST with JSON using AJAX (grande). All of which will log data to the [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools) console (F12 or CTRL+SHFT+C) as well as in the Python console for troubleshooting purposes. The two files that I'll be using are app.py & html.html, the basis of all three modes are below and we'll develop them throughout the post to achieve the goals - 
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="stylesheet" href="https://bootswatch.com/4/superhero/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <title>App</title>
  </head>
  <body>
    <div>
      <center>
        <br>
        <input style="width: 500px;" class="form-control" id="inputText" placeholder="Enter some text here">
        <button style="width: 500px;" onclick="myFunction(inputText.value)" type="submit" class="btn btn-primary">Submit</button>
      </center>
    </div>
  </body>
  <script>
    function myFunction(text) {
    }
  </script>
</html>
```
A few things to note here - 
- the 4th line is the [Superhero Bootstrap](https://bootswatch.com/superhero/) (just to make it look pretty);
- the 5th line is the jQuery & AJAX script, this isn't used in the first example but is in the later two;
- the input is alocated the id *inputText*;
- when clicked, the button triggers a function called *myFunction*, passing it the data (value) of the input box;
- myFunction has the variable *text*, which is the data passed to it from the button (which in turn see the points above);
- the html.html file is located inside a folder called *templates* as per the Flask requirements for all HTML files.

The Python data we have to start with is - 
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():

    return render_template("html.html")

if __name__ == "__main__":
    app.run(
        host=("127.0.0.1"), port=int(500), use_reloader=True, debug=True)
```
A few things to note here - 
- when running, Flask is running on http://127.0.0.1:500; 
- the only HTML file it is rendering is the above html.html file on the route / (AKA, nothing else to add on the end of the URI above). 

# GET With a Query String

To get this set up, there is very minimal code required - happy days. Let's start with Flask. We're going to add two routes, one to receive the data and the next to redirect you to the next page including the text from the one before. We're also going to create a variable and pass it into our function so that it can be writen to.
```python
from flask import Flask, render_template, redirect, url_for, request

text = ''

app = Flask(__name__)

@app.route('/')
def home():
    return render_template("html.html")

@app.route('/receive')
def receive():
    global data

    text = request.args.get('data', 0, type=str)
    print(text)
    return redirect(url_for('next'))

@app.route('/next')
def next():
    return f"You typed this: {text}"

if __name__ == "__main__":
    app.run(
        host=("127.0.0.1"), port=int(500), use_reloader=True, debug=True)
```
So, what has changed here? We've added - 
- three more Flask binaries: redirect handles redirects, passing HTTP 302 redirects; url_for retrieves the app.route URL for a given function; request allows you to parse the query strings that Flask receives. 
- a string variable called *text* with no value.
- an app.route for */receive* with the function *receive*. The function is given the *text* variable via global so that it can write to it. It also populates then prints *text* variable with the *data* query string. Lastly, it uses redirect & url_for to tell the browser to go to the *next* page. 
- another app.route for */next* with the function *next*. This function is very simple HTML that simply displays the *text* variable. 

But what is the data query string and the text variable? Well, *text* is JavaScript varaible but the query string must be constructed using the *?foo=bar* format and sent to our */receive* app.route. This is done by the JavaScript code below. It's only one line of code that needs to be placed inside the original JavaScript curley braces, but we're going to pass two lines - one of which is just so we can see the variable in the Chrome DevTools console.
```html
<script>
  function myFunction(text) {
    console.log(text)
    window.location.href = `/receive?data=${text}`;
  }
</script>
```
No time like the present - run it! You'll see the variable breifly if you have the DevTools open, you'll also see it printed in the Python terminal as well as the */next* HTML page. You can see that the results are basically the same as the blog post picture. In the Python window you'll note the 200 OK code for the initial GET request to the */* page, the 302 redirect code with the query string and the 200 OK code for the */next* page. Working as expected - data to Python from HTML via JavaScript using a query string.

**&lt;NOTE>**: If you wanted to display the text in HTML in a .html file, you'll need to specifiy render_remplate like with the initial */* page but use [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/) syntax to display it - which is basically the variable name inside two curley braces either side, i.e. {{ var }} **&lt;/NOTE>**

# POST With jQuery

Although we can use GET requests like the above, we should realy use POST method to send data to a server - [By design, the POST request method requests that a web server accepts the data enclosed in the body of the request message, most likely for storing it](https://en.wikipedia.org/wiki/POST_(HTTP)). The Python part has a few more changes, but to send a POST request with JavaScript, we can use the single line post jQuery function below. The great thing about this function is that we can pass JSON data as seen below with the *data* and *text* key value pair. What happens is the server receives a POST request with the JSON data in the body - which is very simple to manipulate/store.
```html
<script>
  function myFunction(text) {
    console.log(text)
    $.post("/receive", {"data": text});
  }
</script>
```
For the Python part - 
```python
@app.route('/receive', methods=['POST'])
def receive():
    global text

    text = request.form['data']
    print(text)
    return '', 200
```
Here we have - 
- changed the app.route for */receive* to only invoke when it receives a POST request;
- it then takes the JSON key value pair and stores the variable assigned to *data* as the Python *text* variable;
- the server is then responded a 200 code, as a success.

**&lt;NOTE>**: A big thing to note here is jQuery can't really deal with 302 redirects. A workaround may be to use Jinja2 to then redirect you to the */next* page with -
```python
$.post("/receive", {"data": text}, (window.location.href = "{{ url_for('next') }}"));
```
 **&lt;/NOTE>**
 
As with the above GET request, you'll see the variable breifly if you have the DevTools open, you'll also see it printed in the Python terminal as well as the */next* HTML page. You can see that the results are basically the same as the blog post picture. In the Python window you'll note the 200 OK code for the initial GET request to the */* page, but unlike the 302 redirect code with the query string from above, you'll see a POST 200 OK code for the */receive* page. 

# POST With AJAX

Here it is. Although there are more ways to send data between browser/client & server, the method below (IMHO) is the best way to pass data between HTML and Flask. This method uses AJAX, a set of functions that work very well in asynchronous scenarios -[AJAX allows web pages to be updated asynchronously by exchanging data with a web server behind the scenes](https://www.w3schools.com/whatis/whatis_ajax.asp). We will use it to send data to and from Flask, without 'bothering' the browser. Unlike the name suggests (**A**synchronous **J**avaScript **A**nd **X**ML), we will use it to transfer JSON data - which it is very good at. The Python code is very similar, all bar the last two lines which are - 
- creating a variable containing JSON data called *redirect_json*;
- using the Flask binary jsonify to transfer JSON data.
```python
from flask import Flask, render_template, redirect, url_for, request, jsonify
#
#rest of the 
#script from above
#
@app.route('/receive', methods=['POST'])
def receive():
    global text

    text = request.form['data']
    print(text)
    redirect_json = {"redirect_url": url_for('next')}
    return jsonify(redirect_json)
```
Inside the HTML, we need to write the following code. There's a lot more here than our one/two lines in the last examples but because AJAX works somewhat independantly (same issue again as above with redirects) we need to tell it to do a bit more - 
```html
<script>
  function myFunction(text) {
      console.log(text)
      var jsonVar = {"data": text};
      $.ajax({
          type: "POST",
          url: "/receive",
          data: jsonVar,
          dataType: "json",
          success: function (redirect_json) {
            if (redirect_json.redirect_url) {
                console.log(redirect_json.redirect_url)
                window.location.href = redirect_json.redirect_url;
              }
            }
      });
  }
</script>
```
From the top, we have - 
- *var jsonVar = {"data": text};* this line creates JSON using the JavaScript variable *text* formed from the data in input box;
- *$.ajax({* invokes AJAX. All of the text to follow relates to AJAX; 
- *type: "POST"* sets the request method to be POST;
- *url: "/receive"* is the app.route location;
- *data: jsonVar* is the newly created JavaScript variable;
- *success:*[..] to the end is the success function. It's what happens when AJAX receives a 200 code in response. Here we are telling AJAX to look out for the *redirect_json* JSON file and specifically if it has a key value pair for *redirect_url* - if it does, it will load the */next* URL.

As with the above GET & POST requests, you'll see the variable breifly if you have the DevTools open, you'll also see it printed in the Python terminal as well as the */next* HTML page. You can see that the results are basically the same as the blog post picture. In the Python window you'll note the 200 OK code for the initial GET request to the */* page, a POST 200 OK code for the */receive* page and a final 200 OK code for the */next* page. In the DevTools you'll also note the *redirect_url* JSON varialbe - again, useful for troubleshooting.

For completeness, here's the full Python code -
```python
from flask import Flask, render_template, redirect, url_for, request, jsonify

text = ''

app = Flask(__name__)

@app.route('/')
def home():
    return render_template("html.html")

@app.route('/receive', methods=['POST'])
def receive():
    global text

    text = request.form['data']
    print(text)
    redirect_json = {"redirect_url": url_for('next')}
    return jsonify(redirect_json)

@app.route('/next')
def next():
    return f"You typed this: {text}"

if __name__ == "__main__":
    app.run(
        host=("127.0.0.1"), port=int(500), use_reloader=True, debug=True)
```
... and HTML code -
```html
<!DOCTYPE html>
<html lang="en">
  <head>
  <link rel="stylesheet" href="https://bootswatch.com/4/superhero/bootstrap.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
  <title>App</title>
  </head>
  <body>
  <div>
    <center>
    <br>
    <input style="width: 500px;" class="form-control" id="inputText" placeholder="Enter some text here">
    <button style="width: 500px;" onclick="myFunction(inputText.value)" type="submit" class="btn btn-primary">Submit</button>
    </center>
  </div>
  </body>
  <script>
    function myFunction(text) {
        console.log(text)
        var jsonVar = {"data": text};
        $.ajax({
            type: "POST",
            url: "/receive",
            data: jsonVar,
            dataType: "json",
            success: function (redirect_json) {
              if (redirect_json.redirect_url) {
                  console.log(redirect_json.redirect_url)
                  window.location.href = redirect_json.redirect_url;
                }
              }
        });
    }
  </script>
</html>
```
So there we have it. Three ways to send data to and from Flask & HTML. 

Happy scripting!
