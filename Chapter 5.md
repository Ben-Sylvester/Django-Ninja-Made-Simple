## Connecting Django Ninja with a Database (Models + ORM)
Teaching Your API to Remember Things

## The Goal of This Chapter

So far, your APIs can accept and send data — but every time you restart the server, all that  
data disappears.  

It’s time to give your API a memory by connecting it to a database using Django’s ORM  
(Object Relational Mapper).  

By the end of this chapter, you’ll be able to:

- Connect Django Ninja to a database  
- Create and use models  
- Save, retrieve, and list data using Django ORM  

## What is an ORM?

ORM stands for **Object Relational Mapper**.  

It allows you to interact with your database using Python objects instead of raw SQL.

For example:

```sql
-- Instead of SQL like this
INSERT INTO users (username, email) VALUES ('John', 'john@example.com');
````

```python
# You can do this
user = User(username="John", email="john@example.com")
user.save()
```

ORMs make your code cleaner, safer, and database-independent.

## Django Ninja Made Simple

---

## Step 1: Setting Up Your Database

Django comes preconfigured with SQLite — perfect for beginners.

It stores data in a single file (`db.sqlite3`) and requires no setup.

In `settings.py`, you’ll see:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / "db.sqlite3",
    }
}
```

You can keep this as is for now.

---

## Step 2: Creating a Model

A **model** defines a table in your database.

Let’s create one for users.

Create `users/models.py`:

```python
from django.db import models

class User(models.Model):
    username = models.CharField(max_length=50)
    email = models.EmailField(unique=True)
    age = models.IntegerField()

    def __str__(self):
        return self.username
```

Each field represents a column in the database table.

---

## Step 3: Run Migrations

Once the model is ready, create the actual table in the database:

```bash
python manage.py makemigrations
python manage.py migrate
```

Done! You now have a `users_user` table in your database.

---

## Step 4: Connecting the Model to Ninja API

Now, create endpoints using the `User` model.

In `api.py`:

```python
from ninja import NinjaAPI, Schema
from .models import User

api = NinjaAPI()

class UserIn(Schema):
    username: str
    email: str
    age: int

class UserOut(Schema):
    id: int
    username: str
    email: str
    age: int

@api.post("/users", response=UserOut)
def create_user(request, data: UserIn):
    user = User.objects.create(**data.dict())
    return user

@api.get("/users", response=list[UserOut])
def list_users(request):
    return list(User.objects.all())
```

### What’s Happening Here

* `User.objects.create(**data.dict())` → creates and saves a new record
* `User.objects.all()` → fetches all users from the database
* Schemas validate and structure input/output data automatically

---

## Step 5: Viewing Your Data

After creating users in Swagger, check your database using Django shell:

```bash
python manage.py shell
```

```python
from users.models import User
User.objects.all()
```

You’ll see your persisted data.

### Example Output

**POST /users**

```json
{
    "username": "Alice",
    "email": "alice@example.com",
    "age": 28
}
```

**Response**

```json
{
    "id": 1,
    "username": "Alice",
    "email": "alice@example.com",
    "age": 28
}
```

**GET /users**

```json
[
    {
        "id": 1,
        "username": "Alice",
        "email": "alice@example.com",
        "age": 28
    }
]
```

---

## Updating and Deleting Data (Preview)

We’ll cover full CRUD later, but here’s a sneak peek:

```python
@api.put("/users/{user_id}", response=UserOut)
def update_user(request, user_id: int, data: UserIn):
    user = User.objects.get(id=user_id)
    for attr, value in data.dict().items():
        setattr(user, attr, value)
    user.save()
    return user

@api.delete("/users/{user_id}")
def delete_user(request, user_id: int):
    User.objects.filter(id=user_id).delete()
    return {"message": "User deleted successfully!"}
```

---

## Try It Yourself
* Create a model for `Product` with fields: `name`, `price`, and `stock`
* Create schemas: `ProductIn` and `ProductOut`
* Build `/products` (POST) and `/products` (GET) endpoints
* Test in Swagger UI and check the database
