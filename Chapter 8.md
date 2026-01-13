## JWT Authentication
Modern, Token-Based Security for Your Django Ninja API

---

## The Goal of This Chapter

Basic Authentication is simple, but it’s not ideal for production apps — credentials are sent with every request.  

Modern systems use **JWT (JSON Web Tokens)** — short-lived, secure tokens that clients can store and send automatically.  

By the end of this chapter, you’ll know:

- What JWT is and why it’s used  
- How to set up JWT authentication in Django Ninja  
- How to protect routes using tokens  

---

## What Is JWT?

A **JWT (JSON Web Token)** is a secure digital token that identifies a user.  

Instead of sending a username/password every time, a user logs in once and gets a token. That token is then sent with every request.  

**Typical flow:**

1. User logs in → API gives a JWT token.  
2. User includes token in headers → API checks it → Grants access.  

JWTs are **stateless** — they don’t require Django sessions or cookies. They’re ideal for mobile and modern web apps.

---

## Step 1: Install the JWT Library

Django Ninja doesn’t include JWT by default, but it works perfectly with **PyJWT**.  

```bash
pip install PyJWT
````

---

## Step 2: Create a JWT Utility File

In your app folder, create `auth_utils.py`:

```python
import jwt
from datetime import datetime, timedelta
from django.conf import settings

# Secret key from settings.py
JWT_SECRET = settings.SECRET_KEY
JWT_ALGORITHM = "HS256"

def create_jwt_token(user):
    payload = {
        "user_id": user.id,
        "username": user.username,
        "exp": datetime.utcnow() + timedelta(hours=1)  # expires in 1 hour
    }
    return jwt.encode(payload, JWT_SECRET, algorithm=JWT_ALGORITHM)

def decode_jwt_token(token):
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None
```

This file **creates and validates JWTs**.

---

## Step 3: Create Login API That Returns JWT

```python
from django.contrib.auth import authenticate
from ninja import NinjaAPI, Schema
from .auth_utils import create_jwt_token

api = NinjaAPI()

class LoginIn(Schema):
    username: str
    password: str

class LoginOut(Schema):
    access_token: str
    token_type: str = "Bearer"

@api.post("/login", response=LoginOut)
def login(request, data: LoginIn):
    user = authenticate(username=data.username, password=data.password)
    if not user:
        return api.create_response(request, {"error": "Invalid credentials"}, status=401)

    token = create_jwt_token(user)
    return {"access_token": token}
```

When a user logs in, they receive a token like:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Step 4: Create JWT Authentication Class

In `api.py`:

```python
from ninja.security import HttpBearer
from .auth_utils import decode_jwt_token
from django.contrib.auth.models import User

class JWTAuth(HttpBearer):
    def authenticate(self, request, token):
        payload = decode_jwt_token(token)
        if payload:
            try:
                user = User.objects.get(id=payload["user_id"])
                return user
            except User.DoesNotExist:
                return None
```

This `JWTAuth` class can now be attached to any endpoint you want to secure.

---

## Step 5: Protect an Endpoint

```python
auth = JWTAuth()

@api.get("/profile", auth=auth)
def get_profile(request):
    user = request.auth
    return {"username": user.username, "email": user.email}
```

Now, to access `/profile`, the user must send a **valid JWT token**.

Swagger UI provides an **“Authorize”** button. Enter your token as:

```
Bearer <your_token_here>
```

---

## Example Flow

**Login**

```
POST /login
{
    "username": "alice",
    "password": "password123"
}
```

**Response**

```
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer"
}
```

**Access Protected Route**

```
GET /profile
Header: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response**

```
{
    "username": "alice",
    "email": "alice@example.com"
}
```

---

## Step 6: Handling Expired Tokens

If the token expires (after 1 hour in our example), the API responds:

```json
{"detail": "Unauthorized"}
```

You can refresh tokens by simply re-logging in or building a `/refresh-token` route.

---

## Optional: Token Expiry and Refresh

```python
def refresh_jwt_token(old_token):
    payload = decode_jwt_token(old_token)
    if payload:
        user_id = payload["user_id"]
        username = payload["username"]
        new_payload = {
            "user_id": user_id,
            "username": username,
            "exp": datetime.utcnow() + timedelta(hours=1)
        }
        return jwt.encode(new_payload, JWT_SECRET, algorithm=JWT_ALGORITHM)
    return None
```

---

## Try It Yourself

* Register a user.
* Log in to get a JWT token.
* Copy the token and paste it in Swagger’s **Authorize** section.
* Try accessing `/profile` (it will work only if the token is valid).

