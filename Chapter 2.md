
## Chapter 2  
Setting Up Your Development Environment

Preparing Your System to Build Django Ninja APIs

## The Goal of This Chapter

Before you start building APIs, you need a proper setup — your tools, editor, and Python  
environment.

In this chapter, you’ll learn how to:

- Install Python and Django Ninja
- Create a virtual environment
- Set up your first Django project
- Verify that everything works correctly

## Step 1: Check If Python Is Installed

Open your terminal (Command Prompt, PowerShell, or VS Code Terminal) and type:

```bash
python --version
````

You should see something like:

```text
Python 3.10.11
```

Django Ninja works best with Python **3.9 or later**.

If you don’t have Python installed:

* Go to [https://python.org/downloads](https://python.org/downloads)
* Download the latest Python version for your OS
* During installation, check **“Add Python to PATH”**

## Step 2: Create a Project Folder

Choose a folder where you’ll keep all your code projects.

For example:

```bash
mkdir django_ninja_project
cd django_ninja_project
```

This will be your working directory.

## Step 3: Create a Virtual Environment

A virtual environment keeps your project’s Python packages separate from your system packages.

Create one with:

```bash
python -m venv venv
```

Then activate it:

### On Windows

```bash
venv\Scripts\activate
```

### On Mac/Linux

```bash
source venv/bin/activate
```

Once activated, you’ll see `(venv)` at the start of your terminal line — that means your
virtual environment is active.

### Beginner Tip

Always activate your virtual environment before running or installing anything related to Django.

## Step 4: Install Django and Django Ninja

Now install both Django and Django Ninja using pip:

```bash
pip install django django-ninja
```

Once done, verify the installation:

```bash
python -m django --version
```

You should see something like:

```text
5.1.2
```

Django Ninja is now available as part of your installed packages.

## Step 5: Create a Django Project

Now, let’s create your main Django project.

Run this:

```bash
django-admin startproject ninja_api .
```

That will create several files and folders:

```
ninja_api/
├── __init__.py
├── settings.py
├── urls.py
├── asgi.py
└── wsgi.py
```

These are the core files of any Django project.
You’ll mainly work with `settings.py`, `urls.py`, and `manage.py`.

## Step 6: Run the Development Server

Test that your Django project runs correctly:

```bash
python manage.py runserver
```

If everything is okay, you’ll see something like:

```text
Starting development server at http://127.0.0.1:8000/
```

Open that link in your browser.
You should see the Django welcome page.

## Step 7: Connect Django Ninja

Now let’s connect Django Ninja to your Django project.

Create a new file named `api.py` in the same folder as `manage.py`.

Paste this inside it:

```python
# api.py
from ninja import NinjaAPI

api = NinjaAPI()

@api.get("/hello")
def hello(request):
    return {"message": "Hello from Django Ninja!"}
```

Then open `ninja_api/urls.py` and update it:

```python
from django.contrib import admin
from django.urls import path
from api import api

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", api.urls),
]
```

Now run your server again:

```bash
python manage.py runserver
```

Visit:

```
http://127.0.0.1:8000/api/docs
```

You’ll see the Django Ninja Swagger documentation — ready to test your first endpoint.

## Beginner Tips

* Always activate your virtual environment before running commands
* Keep your `api.py` clean and organized — it’s where your endpoints live
* Visit `/api/docs` to test APIs without writing frontend code
* If the server fails to start, check that `api` is imported correctly in `urls.py`

## Troubleshooting

| Issue                                        | Fix                                              |
| -------------------------------------------- | ------------------------------------------------ |
| `'django' is not recognized`                 | Reinstall Python or check PATH                   |
| `ModuleNotFoundError: No module named ninja` | Run `pip install django-ninja` again             |
| Server not loading                           | Make sure you are in the folder with `manage.py` |

## Exercise

Try these steps:

* Create your own API endpoint `/welcome` that returns your name
* Test it on the Swagger docs page
* Create a new endpoint `/add` that takes two numbers and returns their sum

### Example:

```python
@api.get("/add")
def add(request, a: int, b: int):
    return {"result": a + b}
```

Visit `/api/docs` and try it out!
