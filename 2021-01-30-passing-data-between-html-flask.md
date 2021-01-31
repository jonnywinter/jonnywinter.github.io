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

Running [Flask](https://flask.palletsprojects.com/) for web apps is great - a few lines of code, that are ready to [copy & paste](https://palletsprojects.com/p/flask/) from the Pallets Project site and you're good to go. Add in a choice [bootstrap](https://getbootstrap.com/) CSS and within minutes the terible looking HTML web pages you made as a kid (let alone Myspace!) are a distant memory. However, more often than not you're going to need to work with data that is retrieved from that web page, often by way of a form or bulk upload - what's the best way to do it with Flask? For simple things we can use query strings, but for more intensive stuff we're going to want to pass JSON - think, a .CSV file uploaded to a web browser with a lot of data within could need to be converted to JSON to be worked with in Python. In this post I'm going to write a few methods of passing data and sending responses, like redirects, between Flask and the web browser. 

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

## Let's begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

To pass data, I'm going to tackle it in three ways - GET with a query string using JavaScript (piccolo), POST with JSON using jQuery (medio) and POST with JSON using AJAX (grande). All of which will log data to the [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools) console (F12 or CTRL+SHFT+C) as well as in the Python console for troubleshooting purposes. The two files that I'll be using are app.py & html.html, the basis of all three modes are below and we'll develop them throughout the post to achieve the goals - 
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="stylesheet" href="https://bootswatch.com/4/superhero/bootstrap.min.css">
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


Happy scripting!
