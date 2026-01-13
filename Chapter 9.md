## Role-Based Permissions
Controlling What Users Can Access in Your API

---

## The Goal of This Chapter

Now that we’ve learned how to authenticate users with JWT, the next step is **authorization** — deciding what a user can do after they log in.  

By the end of this chapter, you’ll learn:

- The difference between authentication and authorization  
- How to assign user roles (e.g., admin, editor, viewer)  
- How to restrict routes based on user permissions  
- How to make your API smarter and more secure  

---

## Authentication vs. Authorization

| Concept        | Description                       | Example                          |
|----------------|----------------------------------|----------------------------------|
| Authentication | Confirms who the user is          | Logging in with username/password |
| Authorization  | Determines what the user can do   | Admin can delete users; viewer cannot |

They work together:

- **Authentication** → Validates the user (using JWT).  
- **Authorization** → Checks their permissions (using roles or groups).  

---

## Step 1: Add Roles to Users

There are two common ways to manage roles:

- Use Django’s built-in **Groups and Permissions**  
- Create a **custom field** in your user model (simpler for small apps)  

We’ll use the **custom field method** for clarity.

Update `models.py`:

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    ROLE_CHOICES = (
        ("admin", "Admin"),
        ("editor", "Editor"),
        ("viewer", "Viewer"),
    )
    role = models.CharField(max_length=10, choices=ROLE_CHOICES, default="viewer")

    def __str__(self):
        return f"{self.username} ({self.role})"
````

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Step 2: Create Sample Users with Roles

You can create users using **Django Admin** or the **Python shell**:

```bash
python manage.py createsuperuser
```

Or directly in the shell:

```python
from app.models import User

User.objects.create_user(username="alice", password="1234", role="admin")
User.objects.create_user(username="bob", password="1234", role="editor")
User.objects.create_user(username="eve", password="1234", role="viewer")
```

---

## Step 3: Create Role-Based Permission Decorator

Create a helper function to check if a user has the right role.
Create `permissions.py`:

```python
from ninja.errors import HttpError

def role_required(allowed_roles):
    def decorator(func):
        def wrapper(request, *args, **kwargs):
            user = request.auth
            if user.role not in allowed_roles:
                raise HttpError(403, f"Permission denied: {user.role} cannot access this resource.")
            return func(request, *args, **kwargs)
        return wrapper
    return decorator
```

Now you can **protect routes** by specifying allowed roles.

---

## Step 4: Use Role-Based Decorators on Routes

In `api.py`:

```python
from .permissions import role_required
from .auth_utils import JWTAuth

auth = JWTAuth()

@api.get("/admin/users", auth=auth)
@role_required(["admin"])
def list_users(request):
    from app.models import User
    users = User.objects.all().values("id", "username", "email", "role")
    return list(users)

@api.post("/edit/article", auth=auth)
@role_required(["admin", "editor"])
def edit_article(request):
    return {"message": f"{request.auth.username} edited the article!"}

@api.get("/read/article", auth=auth)
@role_required(["admin", "editor", "viewer"])
def read_article(request):
    return {"message": "You can view this article"}
```

**Example Flow:**

* **alice (admin)** → can list users, edit, and read
* **bob (editor)** → can edit and read
* **eve (viewer)** → can only read

---

## Step 5: Test It in Swagger

1. Log in as **bob (editor)**.
2. Copy your JWT token.
3. Click **Authorize** in Swagger → Paste `Bearer <token>`.
4. Call `/admin/users` → You’ll get **403 Forbidden**
5. Call `/edit/article` → Works
6. Call `/read/article` → Works

This proves your **permission system works correctly**.

---

## Optional: Use Django’s Built-in Groups

For larger systems, you can use Django Groups:

```python
from django.contrib.auth.models import Group
```

Each group (like “Admin”, “Editor”, etc.) can be linked to permissions dynamically.

But for learning and small apps, the **custom field method** is simpler and easier to understand.

---

## Exercise

* Add a new role called `"moderator"`.
* Give moderators the ability to **read and edit**, but **not delete**.
* Update the decorator to allow this new role in `/edit/article`.
* Test it by creating a moderator user and checking your API routes.

