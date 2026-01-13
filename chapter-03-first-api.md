## Chapter 3
Building Your First Real API Step by Step

# The Goal of This Chapter
Now that you have Django and Django Ninja installed and running, it’s time to build your
first real API.
You’ll learn:
 How API routes (endpoints) work
 How to organize your code
 How to accept user input
 How to return data as JSON
By the end of this chapter, you’ll have a working mini API project.

What Is an Endpoint?
An endpoint is simply a URL path your API listens to.
For example:
## Endpoint Description
/hello Returns a greeting
/add Adds two numbers
/users Returns a list of users

When someone visits your endpoint (via a web or mobile app), Django Ninja runs the
Python function linked to that endpoint.
The Basic Structure of a Ninja API
You already created a simple api.py in Chapter 2.
Here’s how a typical api.py looks:
# api.py
from ninja import NinjaAPI

## Django Ninja Made Simple

api = NinjaAPI()

@api.get("/hello")
def hello(request):
return {"message": "Hello, Django Ninja!"}
When you open http://127.0.0.1:8000/api/docs, Django Ninja automatically shows this
/hello endpoint in the documentation — ready to test.

## Creating Multiple Endpoints
You can create as many endpoints as you want, just by adding new decorators like
@api.get() or @api.post().
Let’s add a few examples:
# api.py
from ninja import NinjaAPI
api = NinjaAPI()

@api.get("/hello")
def hello(request):
return {"message": "Hello, Django Ninja!"}

@api.get("/greet")
def greet_user(request, name: str):
return {"greeting": f"Hello, {name}!"}

@api.get("/add")
def add_numbers(request, a: int, b: int):

## Django Ninja Made Simple


return {"result": a + b}

## What’s Happening Here?
 /greet?name=John → returns “Hello, John!”
 /add?a=10&b=20 → returns { "result": 30 }
Django Ninja automatically converts query parameters like a=10 into Python integers (int).
Different Request Types (GET, POST, PUT, DELETE)
APIs use HTTP methods to define what action to perform:

## Method Action Example
GET Retrieve data /users
POST Create new data /users
PUT Update existing data /users/1
DELETE Remove data /users/1
You’ll use these to create CRUD operations (Create, Read, Update, Delete) later.

Example of a POST endpoint:
@api.post("/welcome")
def welcome_user(request, name: str):
return {"message": f"Welcome, {name}!"}
When you test this from Swagger, it will send a POST request with name in the form data or

## JSON.
Handling URL Path Parameters
Sometimes you want to include a value inside the URL, not as a query.
## Example:
@api.get("/user/{user_id}")

## Django Ninja Made Simple

def get_user(request, user_id: int):
return {"user_id": user_id, "status": "Found"}
If you visit /api/user/10, Django Ninja passes 10 to the function automatically.

## Beginner Tip

Query parameters ( ?a=1&b=2) are best for filtering or optional data.
Path parameters (/user/10) are for identifying specific items.
Organizing Your Endpoints (Routers)
As your project grows, your api.py file can get big.
That’s where Routers come in — they help organize related endpoints.
## Example:
# users.py
from ninja import Router

router = Router()

@router.get("/list")
def list_users(request):
return {"users": ["Alice", "Bob", "Charlie"]}
Now in api.py:
from ninja import NinjaAPI
from users import router as users_router

api = NinjaAPI()
api.add_router("/users", users_router)

## Django Ninja Made Simple
Now your docs will show:
 /users/list → returns a list of users.
You can create routers for different modules (e.g., users, products, orders) to keep code
clean.

## Testing It All
Start your server:
python manage.py runserver
## Visit:
http://127.0.0.1:8000/api/docs
You’ll see all endpoints listed — /hello, /greet, /add, /users/list, etc.
Click on any one, hit “Try it out”, and you’ll get instant results!

## Common Mistakes (and Fixes)
## Mistake Fix
Forgot to add api.urls in urls.py Add path("api/", api.urls)
Missing type hints (a: int, b: int) Always include data types
Browser shows “Not Found” Check the endpoint path carefully
Error: “import not found” Make sure the file name and import path are correct

## Exercises
- Create an endpoint /multiply that takes x and y and returns their product.
- Add a new router for products with a /list endpoint that returns a few product
names.
- Create a /welcome/{name} endpoint that returns "Welcome, <name>!".
