## Error Handling & Validation
**Making Your API Smart, Reliable, and Foolproof**

### The Goal of This Chapter
Your API should never crash or confuse users when bad data or errors occur. This chapter teaches you how to gracefully handle such issues.

By the end, you’ll learn:
- How Django Ninja automatically validates input
- How to define custom error messages
- How to handle server and client errors
- How to send clean, structured responses for all situations

---

## Step 1: Built-in Validation with Pydantic
Django Ninja uses **Pydantic** under the hood — a library that validates data automatically based on Python type hints.

```python
from ninja import Schema, NinjaAPI

api = NinjaAPI()

class UserIn(Schema):
    name: str
    age: int

@api.post("/users")
def create_user(request, data: UserIn):
    return {"message": f"User {data.name} ({data.age}) created successfully"}
````

Example invalid input:

```json
{"name": "Alice", "age": "twenty"}
```

Response from Ninja:

```json
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": ["body", "age"],
      "msg": "Input should be a valid integer"
    }
  ]
}
```

No manual validation is needed — Ninja handles it for you automatically.

---

## Step 2: Adding Default Values and Constraints

You can add validation rules and defaults inside your schemas using Pydantic's `Field`.

```python
from pydantic import Field

class RegisterUser(Schema):
    username: str = Field(..., min_length=3, max_length=20)
    password: str = Field(..., min_length=6)
    email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$")
```

Ninja will:

* Reject usernames shorter than 3 characters
* Reject invalid emails
* Require all fields (`...` means required)

---

## Step 3: Handling Common Errors Gracefully

If something goes wrong in your route, raise a `HttpError`.

```python
from ninja.errors import HttpError

@api.get("/divide")
def divide(request, a: int, b: int):
    if b == 0:
        raise HttpError(400, "Division by zero is not allowed.")
    return {"result": a / b}
```

Example:

```
GET /divide?a=10&b=0
```

Response:

```json
{
  "detail": "Division by zero is not allowed."
}
```

---

## Step 4: Custom Error Responses

Define your own error format using exception handlers.

```python
from ninja.errors import HttpError
from django.http import JsonResponse

@api.exception_handler(HttpError)
def custom_http_error(request, exc):
    return JsonResponse({
        "success": False,
        "error": str(exc),
        "path": request.path
    }, status=exc.status_code)
```

Response format becomes:

```json
{
  "success": false,
  "error": "Division by zero is not allowed.",
  "path": "/divide"
}
```

---

## Step 5: Global Exception Handling

Catch unexpected server errors to prevent exposing ugly traces:

```python
@api.exception_handler(Exception)
def global_exception_handler(request, exc):
    return JsonResponse({
        "success": False,
        "error": "Unexpected server error. Please try again later."
    }, status=500)
```

---

## Step 6: Validation for Query & Path Parameters

Validate query parameters using type hints:

```python
@api.get("/search")
def search_users(request, keyword: str, limit: int = 10):
    if limit > 100:
        raise HttpError(400, "Limit cannot exceed 100")
    return {"message": f"Searching for {keyword}, limit {limit}"}
```

* `keyword` is required
* `limit` defaults to 10
* Raises an error if `limit > 100`

---

## Step 7: Custom Validation with Pydantic Validators

Use `@validator` for complex rules:

```python
from pydantic import validator

class ProductIn(Schema):
    name: str
    price: float

    @validator("price")
    def price_must_be_positive(cls, v):
        if v <= 0:
            raise ValueError("Price must be greater than zero")
        return v
```

Invalid input:

```json
{"name": "Laptop", "price": -100}
```

Response:

```json
{
  "detail": [
    {
      "loc": ["body", "price"],
      "msg": "Price must be greater than zero",
      "type": "value_error"
    }
  ]
}
```

---

## Exercise

* Add validation for user registration (name, email, password)
* Add a `/calculate` route that validates positive numbers only
* Add a global handler for all `ValueError` exceptions
* Test it using Swagger’s **Try It Out** feature

