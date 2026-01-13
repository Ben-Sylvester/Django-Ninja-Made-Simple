## Chapter 1
## Introduction to Django Ninja

Building APIs the Easy Way

## What Is Django Ninja?
Django Ninja is a tool that helps you build APIs (Application Programming Interfaces) using
Python and Django — but faster, cleaner, and with less code.
It’s called “Ninja” because it’s:
 Fast: (built on top of FastAPI ideas)
 Simple: (you write fewer lines of code)
 Smart: (it checks your data automatically)
So  instead  of  writing  a  lot  of  code  to  make  a  simple  API,  Django  Ninja  does  most  of  the
heavy lifting for you.
## Why Should You Learn Django Ninja?
Because  it  makes  API  development  easy — especially  if  you  are  a  beginner  in  backend
development or Django.
Here’s what makes it special:
Feature Django Ninja Django REST Framework (DRF)
Speed Very fast (built on ASGI) Slower (WSGI)
Code size Less code More boilerplate
Validation Uses Pydantic automatically Needs manual setup
Docs Auto-generates Swagger & ReDoc Optional setup
Learning curve Easy for beginners Steeper learning

What Django Ninja Does for You
When you create an API, Django Ninja:
 Takes your incoming data (from a web or mobile app)
 Checks it automatically (Is it valid? Is it complete?)
 Passes it to your function to process

## Django Ninja Made Simple

## 2

 Returns a clean JSON response
 And also creates interactive API documentation for you to test right away
 All that — from just a few lines of code!

Example: Your First Django Ninja API
Here’s how easy it is to make your first API with Django Ninja:
# api.py
from ninja import NinjaAPI

api = NinjaAPI()

## @api.get("/hello")
def say_hello(request):
return {"message": "Hello, Django Ninja!"}

## Let’s Understand What’s Happening:
NinjaAPI() → This is your main API app (like a container for your endpoints).
@api.get("/hello") → This tells Django Ninja to make an API endpoint at /hello.
say_hello() → The function that runs when someone visits /hello.
It returns a dictionary, and Django Ninja automatically converts it to JSON.
Now, run your Django project and open:
http://127.0.0.1:8000/api/docs
You’ll see a beautiful Swagger documentation page where you can test your /hello endpoint
— no setup needed.

The “API Docs” Magic

## Django Ninja Made Simple

## 3

The /api/docs page is a live testing playground that Django Ninja builds automatically for
you.
It shows:
 Every endpoint in your project
 The required inputs
 Expected outputs
 And a “Try it out” button to test directly from your browser
 This saves hours of manual testing.
## Beginner Tip

Django    Ninja    is    not    a    replacement    for    Django,    it    works    with    Django.
You still use Django models, ORM, and settings, but now your APIs are easier to manage.

Key Components (You’ll Learn These Later)
## Component Purpose
NinjaAPI The main API app
Routers Group related endpoints together
Schemas (Pydantic Models) Define input/output data
Endpoints Functions that respond to API calls
Docs Built-in testing & documentation (Swagger / ReDoc)

Real-World Uses of Django Ninja
You can use Django Ninja to build:
 Backend for mobile apps (e.g., Flutter, React Native)
 APIs for web dashboards (e.g., Vue, React)
 Internal data services in organizations
 AI model endpoints (for ML predictions)
 Banking or E-commerce systems
 It’s flexible and production-ready.



## Django Ninja Made Simple


## Quick Analogy
Think of Django Ninja like a friendly translator between your website/app and your
backend logic.
User → (Request) → Django Ninja → (Response) → User
You just tell Django Ninja what to send and where, and it takes care of the rest.
## Common Terms You’ll Hear Often
## Term Meaning
API A way for software to communicate
Endpoint A specific path (like /hello) in your API
Request Data coming into your API
Response Data going out from your API
JSON The data format used to send and receive info
Swagger A web tool that shows your API docs automatically

## Exercises
- What is Django Ninja in your own words?
- What’s the difference between Django Ninja and Django REST
## Framework?
- Create a simple API endpoint /greet that returns your name.
- Open your /api/docs page and test it.
