## CRUD Operations in Django Ninja
Create, Read, Update, Delete — the Four Pillars of Every API

## The Goal of This Chapter

Every useful API needs to do four basic things:

- Create data  
- Read data  
- Update data  
- Delete data  

These are called **CRUD operations**, and Django Ninja makes them super easy to build.  

By the end of this chapter, you’ll have a fully functional CRUD API that works with a database and validates everything automatically.

---

## What is CRUD?

CRUD stands for:

| Operation | HTTP Method | Description        |
|-----------|-------------|------------------|
| Create    | POST        | Add new data      |
| Read      | GET         | Retrieve data     |
| Update    | PUT/PATCH   | Modify existing data |
| Delete    | DELETE      | Remove data       |

Let’s bring this to life with an example.

---

## Our Example: Managing Products

We’ll build a simple **Product Management API** that can:

- Add new products  
- View all products  
- Update existing ones  
- Delete unwanted ones  

---

## Step 1: Define the Model

In `products/models.py`:

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.FloatField()
    stock = models.IntegerField(default=0)

    def __str__(self):
        return self.name
````

Run your migrations to create the table:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Step 2: Create Schemas

In `products/api.py`:

```python
from ninja import Schema

class ProductIn(Schema):
    name: str
    price: float
    stock: int

class ProductOut(Schema):
    id: int
    name: str
    price: float
    stock: int
```

---

## Step 3: Create the API Endpoints

```python
from ninja import NinjaAPI
from .models import Product
from .api import ProductIn, ProductOut

api = NinjaAPI()

# CREATE
@api.post("/products", response=ProductOut)
def create_product(request, data: ProductIn):
    product = Product.objects.create(**data.dict())
    return product

# READ ALL
@api.get("/products", response=list[ProductOut])
def list_products(request):
    return list(Product.objects.all())

# READ SINGLE
@api.get("/products/{product_id}", response=ProductOut)
def get_product(request, product_id: int):
    return Product.objects.get(id=product_id)

# UPDATE
@api.put("/products/{product_id}", response=ProductOut)
def update_product(request, product_id: int, data: ProductIn):
    product = Product.objects.get(id=product_id)
    for attr, value in data.dict().items():
        setattr(product, attr, value)
    product.save()
    return product

# DELETE
@api.delete("/products/{product_id}")
def delete_product(request, product_id: int):
    Product.objects.filter(id=product_id).delete()
    return {"message": "Product deleted successfully!"}
```

---

## Understanding the Flow

| Action | Endpoint       | Description           |
| ------ | -------------- | --------------------- |
| POST   | /products      | Create a new product  |
| GET    | /products      | View all products     |
| GET    | /products/{id} | View a single product |
| PUT    | /products/{id} | Update product info   |
| DELETE | /products/{id} | Delete product        |

You can test all these actions directly in Swagger UI (`/api/docs`).

---

## Example Interaction

**POST /products**

```json
{
    "name": "Laptop",
    "price": 899.99,
    "stock": 10
}
```

**Response**

```json
{
    "id": 1,
    "name": "Laptop",
    "price": 899.99,
    "stock": 10
}
```

**GET /products**

```json
[
    {
        "id": 1,
        "name": "Laptop",
        "price": 899.99,
        "stock": 10
    }
]
```

---

## Common Errors (and Fixes)

**Error:** `Product.DoesNotExist`
Happens when you try to access a product that doesn’t exist.

**Fix:**

```python
from django.shortcuts import get_object_or_404

@api.get("/products/{product_id}", response=ProductOut)
def get_product(request, product_id: int):
    product = get_object_or_404(Product, id=product_id)
    return product
```

**Error:** Missing field in POST
Django Ninja automatically checks for missing or invalid fields and returns helpful validation errors.

Example:

```json
{
    "detail": [
        {
            "loc": ["body", "price"],
            "msg": "field required"
        }
    ]
}
```

No manual validation needed — Ninja and Pydantic handle it for you.

---

## Optional Improvement: PATCH for Partial Updates

Sometimes you don’t want to update all fields — only one or two. That’s what PATCH is for.

```python
from typing import Optional
from ninja import Schema

class ProductPatch(Schema):
    name: Optional[str]
    price: Optional[float]
    stock: Optional[int]

@api.patch("/products/{product_id}", response=ProductOut)
def partial_update(request, product_id: int, data: ProductPatch):
    product = Product.objects.get(id=product_id)
    for attr, value in data.dict(exclude_unset=True).items():
        setattr(product, attr, value)
    product.save()
    return product
```

Now you can send only the fields you want to change.
