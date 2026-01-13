 ## Request & Response Models (Schemas)  
Teaching Your API How to Speak in Organized Data

## The Goal of This Chapter
You’ve built simple endpoints using strings and numbers.  
Now it’s time to teach your API to handle real-world data like user info, orders, or posts  
— in a structured, reliable way.

We’ll do that using Schemas, which are like data blueprints.

By the end of this chapter, you’ll know:

- What schemas are
- How to send and receive structured data
- How Django Ninja automatically validates and documents your data

## What Is a Schema?
A Schema is a Python class that describes the shape of your data.

For example, if you’re building a user registration API, your data might look like this:

```json
{
  "username": "JohnDoe",
  "email": "john@example.com",
  "age": 25
}
````

To represent that data in Django Ninja, you create a Schema using Pydantic (a powerful
data validation library built into Django Ninja).

Creating a Schema

## Here’s how to define one:

```python
from ninja import Schema

class UserSchema(Schema):
    username: str
    email: str
    age: int
```

That’s it!

This class tells Django Ninja what to expect — and it will automatically:

* Validate incoming data
* Convert data to JSON
* Generate API documentation fields

Using Schema for Input (Request Body)

Now, let’s accept user data in an endpoint:

```python
from ninja import NinjaAPI, Schema

api = NinjaAPI()

class UserSchema(Schema):
    username: str
    email: str
    age: int

@api.post("/register")
def register_user(request, data: UserSchema):
    return {"message": f"User {data.username} registered successfully!"}
```

## How It Works:

* When a user sends JSON data (username, email, age)
* Django Ninja automatically converts it into a UserSchema object (data)
* You can then access data.username, data.email, etc.

Try it in Swagger!
It automatically shows the form fields and checks the data types for you.

Sending Structured Data Back (Response Model)

Now let’s return a schema as the response too.

```python
class UserResponse(Schema):
    id: int
    username: str
    email: str

@api.post("/create", response=UserResponse)
def create_user(request, data: UserSchema):
    return {"id": 1, "username": data.username, "email": data.email}
```

Now, Django Ninja automatically formats and documents the response structure.

Behind the Scenes

When you add `response=UserResponse`, Ninja:

* Validates your output
* Converts Python objects to JSON
* Documents the result in Swagger

This makes your API predictable and consistent.

Example: Full Create and Get User Flow

```python
from ninja import NinjaAPI, Schema

api = NinjaAPI()

class UserIn(Schema):
    username: str
    email: str

class UserOut(Schema):
    id: int
    username: str
    email: str

users = []  # pretend database

@api.post("/users", response=UserOut)
def create_user(request, data: UserIn):
    user_id = len(users) + 1
    new_user = {"id": user_id, **data.dict()}
    users.append(new_user)
    return new_user

@api.get("/users", response=list[UserOut])
def list_users(request):
    return users
```

## What’s Happening:

* `/users` (POST) → Creates a user
* `/users` (GET) → Lists all created users

Try it in Swagger — you’ll see both routes, full form fields, and automatic documentation.

## Data Validation Magic

Django Ninja automatically checks input data for you.

For example, if your schema says:

```python
class Product(Schema):
    name: str
    price: float
```

Then sending this:

```json
{ "name": "Laptop", "price": "abc" }
```

will return:

```json
{
  "detail": [
    {
      "loc": ["body", "price"],
      "msg": "value is not a valid float"
    }
  ]
}
```

No manual validation needed!

Default Values and Optional Fields

You can set defaults and make some fields optional.

```python
from typing import Optional

class Book(Schema):
    title: str
    author: Optional[str] = "Unknown"
    year: int = 2024
```

If a user omits author, it’ll automatically become `"Unknown"`.

## Quick Tips

* Use Input Schema for data coming in (like UserIn)
* Use Output Schema for data going out (like UserOut)
* Always use type hints (str, int, float, bool, etc.)
* Swagger auto-generates docs for all fields

## Try It Yourself

* Create a schema called `ProductIn` with fields:
  name (str), price (float), stock (int)
* Create `ProductOut` with an extra field `id` (int)

## 3. Build:

* `/products` (POST) → adds a product
* `/products` (GET) → lists all products
