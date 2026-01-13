## Authentication Basics
Keeping Your API Safe — Only the Right People Get In

## The Goal of This Chapter
Up to now, your API accepts and returns data freely — which is great for testing, but terrible for security.  

We need to make sure only authorized users can access certain endpoints. This is where **authentication** comes in.  

By the end of this chapter, you’ll understand:

- What authentication is  
- How to protect endpoints in Django Ninja  
- How to use Django’s built-in authentication system  

---

## What Is Authentication?

Authentication means **confirming who someone is**.  

When you log in to a website or mobile app, you enter your credentials — usually an email/username and password.  

If correct, you’re allowed to access protected resources.

**Example:**

- Anyone can view a public product list (no login).  
- Only admins can add or delete products (login required).  

---

## Step 1: Using Django’s Built-in Auth System

Django already comes with a full authentication system — including:

- Users  
- Passwords (securely hashed)  
- Sessions (for logged-in users)  
- Permissions & Groups  

We’ll build on top of that using Django Ninja.

---

## Step 2: Add Django’s Auth App

Open your `settings.py` and ensure this line is present:

```python
INSTALLED_APPS = [
    ...,
    'django.contrib.auth',
    'django.contrib.contenttypes',
]
````

Run migrations if you haven’t already:

```bash
python manage.py migrate
```

This creates the user tables in your database.

---

## Step 3: Create a User Registration API

Let’s allow new users to sign up via the API.

```python
from django.contrib.auth.models import User
from ninja import NinjaAPI, Schema

api = NinjaAPI()

class RegisterIn(Schema):
    username: str
    email: str
    password: str

class RegisterOut(Schema):
    id: int
    username: str
    email: str

@api.post("/register", response=RegisterOut)
def register(request, data: RegisterIn):
    user = User.objects.create_user(
        username=data.username,
        email=data.email,
        password=data.password
    )
    return user
```

This securely saves the user with a **hashed password**.

---

## Step 4: Basic Authentication (Login Required)

Django Ninja supports **Basic Authentication** — a simple username + password check for endpoints.

```python
from ninja.security import HttpBasicAuth

class BasicAuth(HttpBasicAuth):
    def authenticate(self, request, username, password):
        from django.contrib.auth import authenticate
        user = authenticate(username=username, password=password)
        if user:
            return user
```

Now use it to protect an endpoint:

```python
auth = BasicAuth()

@api.get("/profile", auth=auth)
def get_profile(request):
    user = request.auth  # authenticated user object
    return {"message": f"Welcome {user.username}!"}
```

**How It Works:**

* When calling `/profile`, the client must include credentials (username & password).
* Django verifies them using its built-in user system.
* If valid → access granted.
* If invalid → 401 Unauthorized.

Swagger UI even provides a small **"Authorize"** button so you can test authentication easily.

---

## Step 5: Restricting Access to Admins Only

You can add simple permission logic:

```python
@api.get("/admin-only", auth=auth)
def admin_area(request):
    user = request.auth
    if not user.is_staff:
        return api.create_response(request, {"error": "Access denied"}, status=403)
    return {"message": f"Welcome, admin {user.username}!"}
```

This ensures only **staff members** (`is_staff=True`) can access this route.

---

## Step 6: Testing the Authentication Flow

1. Use `/register` to create a new user.
2. Go to `/profile` in Swagger.
3. Click **Authorize** → enter your username and password.
4. Test again — you should see your welcome message.

Success! Your first protected API endpoint works.

---

## Bonus: Using Django’s Built-in Login System

You can also use Django’s built-in login/logout logic if you want to integrate with **web sessions**:

```python
from django.contrib.auth import login, logout

# Login endpoint
@api.post("/login")
def login_user(request, data: RegisterIn):
    from django.contrib.auth import authenticate
    user = authenticate(username=data.username, password=data.password)
    if user:
        login(request, user)
        return {"message": "Login successful"}
    return {"error": "Invalid credentials"}

# Logout endpoint
@api.post("/logout")
def logout_user(request):
    logout(request)
    return {"message": "Logged out successfully"}
```

This works with **session-based authentication** — great for web apps.

---

## Try It Yourself

* Create `/register` and `/profile` endpoints.
* Use `BasicAuth` to protect `/profile`.
* Test logging in with valid and invalid credentials.
* Add a new `/admin-only` route for staff users.

