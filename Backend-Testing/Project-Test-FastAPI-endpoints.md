Perfect! Let’s design a **project for testing FastAPI endpoints** using **Pytest**, including database mocking. I’ll break it down step by step: setup, endpoints, tests, and best practices.

---

## **1. Project Overview**

**Goal:** Test a FastAPI application with **User Registration and Login endpoints**.

* **Tech Stack:** FastAPI, Pydantic, SQLAlchemy (ORM), Pytest.
* **Focus:**

  * Unit test endpoints without hitting a real database (mocking DB).
  * Test HTTP responses, status codes, and data.

---

## **2. Setup**

Install packages:

```bash
pip install fastapi uvicorn sqlalchemy pytest pytest-asyncio httpx
```

**Optional for mocking:**

```bash
pip install pytest-mock
```

---

## **3. Example FastAPI App**

### **a. Models**

```python
# models.py
from sqlalchemy import Column, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    email = Column(String, primary_key=True)
    name = Column(String)
    password = Column(String)
```

---

### **b. Schemas**

```python
# schemas.py
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    name: str
    password: str

class UserOut(BaseModel):
    email: str
    name: str
```

---

### **c. FastAPI App**

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from models import User
from schemas import UserCreate, UserOut

app = FastAPI()

# Dependency to get DB session
def get_db():
    # Placeholder: In real app, returns DB session
    pass

# In-memory "DB" for simplicity in example
fake_db = []

@app.post("/register", response_model=UserOut)
def register(user: UserCreate):
    # Check if user exists
    for u in fake_db:
        if u.email == user.email:
            raise HTTPException(status_code=400, detail="User exists")
    fake_db.append(user)
    return user

@app.post("/login")
def login(user: UserCreate):
    for u in fake_db:
        if u.email == user.email and u.password == user.password:
            return {"message": "Login successful"}
    raise HTTPException(status_code=401, detail="Invalid credentials")
```

---

## **4. Testing FastAPI Endpoints**

We use **Pytest + TestClient** from FastAPI.

```python
# test_main.py
import pytest
from fastapi.testclient import TestClient
from main import app, fake_db

client = TestClient(app)

@pytest.fixture(autouse=True)
def clear_db():
    # Clear in-memory DB before each test
    fake_db.clear()

def test_register_success():
    response = client.post("/register", json={
        "email": "alice@example.com",
        "name": "Alice",
        "password": "123456"
    })
    assert response.status_code == 200
    data = response.json()
    assert data["email"] == "alice@example.com"
    assert data["name"] == "Alice"

def test_register_duplicate():
    # Pre-add user
    fake_db.append({"email": "bob@example.com", "name": "Bob", "password": "123"})
    response = client.post("/register", json={
        "email": "bob@example.com",
        "name": "Bob",
        "password": "123"
    })
    assert response.status_code == 400
    assert response.json()["detail"] == "User exists"

def test_login_success():
    fake_db.append({"email": "charlie@example.com", "name": "Charlie", "password": "abc"})
    response = client.post("/login", json={
        "email": "charlie@example.com",
        "password": "abc"
    })
    assert response.status_code == 200
    assert response.json()["message"] == "Login successful"

def test_login_failure():
    response = client.post("/login", json={
        "email": "nonexist@example.com",
        "password": "wrong"
    })
    assert response.status_code == 401
    assert response.json()["detail"] == "Invalid credentials"
```

---

## **5. Database Mocking with SQLAlchemy**

Instead of using the real database, you can **mock SQLAlchemy session** using `pytest-mock` or `unittest.mock`.

```python
# test_db_mock.py
from unittest.mock import MagicMock
from main import register

def test_register_db_mock():
    mock_db = MagicMock()
    # Simulate query/filter
    mock_db.query.return_value.filter_by.return_value.first.return_value = None

    user = MagicMock(email="mock@example.com", name="Mock", password="123")
    # Call your service function with mocked DB
    # register(user, db=mock_db)  # If your register function accepted DB session
    # Then assert mock_db.insert called
```

* Use **mock_db** to simulate database behavior without hitting a real DB.
* Ideal for **unit tests** of service functions.

---

## **6. Test Scenarios**

| Endpoint  | Scenario            | Expected Outcome     |
| --------- | ------------------- | -------------------- |
| /register | Valid user          | 200, user returned   |
| /register | Duplicate user      | 400, error detail    |
| /login    | Correct credentials | 200, success message |
| /login    | Wrong credentials   | 401, error detail    |

---

## **7. Best Practices**

1. **Use TestClient** for HTTP-level tests.
2. **Isolate tests**: Use fixtures to reset DB state before each test.
3. **Mock database** for unit tests; real DB for integration tests.
4. **Check both status codes and response body**.
5. **Use async tests** if your endpoints are async:

```python
import pytest
import asyncio

@pytest.mark.asyncio
async def test_async_endpoint():
    response = await client.post("/register", json={...})
```

6. **Separate unit tests and integration tests**. Unit tests mock DB, integration tests can use SQLite in-memory DB.

---

## **8. Summary**

* **FastAPI + Pytest** allows testing endpoints at HTTP level.
* Use **TestClient** for requests, **fixtures** to isolate state.
* **Database mocking** helps unit test services without real DB.
* Test **status codes, response data, and error handling**.
* Combine **unit + integration tests** for full coverage.

---

If you want, I can **create a complete ready-to-run FastAPI project** with:

* `/register` & `/login` endpoints
* Pytest tests with **mocked database**
* `pytest` commands to run all tests

It’ll be fully copy-paste runnable.

Do you want me to do that?
