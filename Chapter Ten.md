## API Versioning and Documentation Mastery
Keeping Your APIs Organized, Compatible, and Well-Explained

---

## The Goal of This Chapter

As your API grows, new versions may be needed — for bug fixes, feature upgrades, or compatibility reasons.  
You’ll also want **clear, automatic documentation** for users and developers.  

By the end of this chapter, you’ll learn:

- How to organize and version your APIs (v1, v2, etc.)  
- How to customize Django Ninja’s Swagger / ReDoc docs  
- How to provide rich documentation with examples and tags  

---

## Step 1: What Is API Versioning?

Imagine your mobile app calls your API. You update it and change some routes — suddenly, old apps stop working.  

**Versioning solves this.**  
It lets you publish new versions while keeping old ones running.

**Example:**

```

/api/v1/users
/api/v2/users

````

Both can exist at the same time, and clients choose which version to use.

---

## Step 2: Implementing Versioning in Django Ninja

In Django Ninja, versioning is simple — just create multiple NinjaAPI instances.

```python
from ninja import NinjaAPI

api_v1 = NinjaAPI(version="1.0.0", title="Banking API - v1")
api_v2 = NinjaAPI(version="2.0.0", title="Banking API - v2")
````

Register both in `urls.py`:

```python
from django.urls import path
from .api_v1 import api as api_v1
from .api_v2 import api as api_v2

urlpatterns = [
    path("api/v1/", api_v1.urls),
    path("api/v2/", api_v2.urls),
]
```

Now, you can **update routes independently** without breaking older clients.

---

## Step 3: Organizing Endpoints by Version

Example: v1 returns basic user info, v2 adds account balance.

**api_v1.py**

```python
from ninja import NinjaAPI

api = NinjaAPI(title="Banking API v1")

@api.get("/users")
def list_users(request):
    return [{"id": 1, "name": "Alice"}]
```

**api_v2.py**

```python
from ninja import NinjaAPI

api = NinjaAPI(title="Banking API v2")

@api.get("/users")
def list_users(request):
    return [{"id": 1, "name": "Alice", "balance": 2000}]
```

Visiting:

* `/api/v1/docs` → Version 1 docs
* `/api/v2/docs` → Version 2 docs

Both versions coexist safely.

---

## Step 4: Auto Documentation (Swagger & ReDoc)

Django Ninja automatically generates:

* **Swagger UI** (interactive testing)
* **ReDoc UI** (clean reference-style documentation)

Visit:

```
http://127.0.0.1:8000/api/v1/docs
http://127.0.0.1:8000/api/v1/redoc
```

You’ll see all **routes, models, and example requests**. No extra setup required.

---

## Step 5: Customizing Documentation

You can enhance the docs using parameters inside route decorators.

```python
from ninja import Router, Schema

router = Router(tags=["User Management"])

class UserOut(Schema):
    id: int
    name: str
    email: str

@router.get(
    "/users", 
    response=list[UserOut], 
    summary="Get all users",
    description="Returns a list of all users in the system."
)
def get_users(request):
    return [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"},
    ]
```

The docs will show:

* Tag: **“User Management”**
* Summary: Short title
* Description: Full explanation

This makes your docs **professional and easy to navigate**.

---

## Step 6: Adding Examples and Metadata

You can show **request/response examples** in Swagger for better clarity.

```python
class CreateUserIn(Schema):
    name: str
    email: str

class CreateUserOut(Schema):
    id: int
    name: str
    email: str

@router.post(
    "/users",
    response=CreateUserOut,
    summary="Create new user",
    description="Registers a new user and returns their details",
    examples={
        "input": {"name": "John", "email": "john@example.com"},
        "output": {"id": 1, "name": "John", "email": "john@example.com"}
    }
)
def create_user(request, data: CreateUserIn):
    return {"id": 1, **data.dict()}
```

Swagger will show **“Try it out” examples** — perfect for beginners testing your API.

---

## Step 7: Customizing Global API Info

When creating your NinjaAPI instance, you can personalize the documentation details:

```python
api = NinjaAPI(
    title="Cyber Oracle Banking API",
    version="2.0.0",
    description="A secure and high-performance banking backend powered by Django Ninja.",
    docs_url="/docs",
    openapi_url="/openapi.json"
)
```

Your docs will now show your **title, version, and description beautifully**.

---

## Exercise

* Create a **v1** and **v2** version of your “Banking API.”
* In **v2**, add a new field called `balance` to the User model output.
* Add **Swagger tags and a summary** to each route.
* Visit both `/v1/docs` and `/v2/docs` — compare the results.

