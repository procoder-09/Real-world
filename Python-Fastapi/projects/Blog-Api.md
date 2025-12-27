Perfect! Let’s design a **full Blog API project** with **role-based access control (Admin/User)** using **Python + FastAPI + SQLAlchemy + JWT**. I’ll give a **step-by-step structure, endpoints, database design, code snippets, and best practices**.

---

# Project: Blog API with Roles (Admin/User)

---

## 🧠 Requirements

1. Users can **register** and **login**
2. Users can **create, read, update, delete their own posts**
3. Admins can **manage all posts** and users
4. JWT-based authentication
5. Role-based authorization
6. Basic pagination & filtering for posts

---

## 🏗️ Tech Stack

* **Backend:** Python, FastAPI
* **Database:** SQLite / PostgreSQL
* **ORM:** SQLAlchemy + Alembic (for migrations)
* **Authentication:** JWT tokens
* **Password hashing:** `passlib`
* **Optional:** `Pydantic` for request validation

---

## 🔹 Database Design

### 1️⃣ Users Table

| Field      | Type     | Notes            |
| ---------- | -------- | ---------------- |
| id         | int      | PK               |
| username   | str      | Unique           |
| email      | str      | Unique           |
| password   | str      | Hashed           |
| role       | str      | "admin" / "user" |
| created_at | datetime | Default now      |

### 2️⃣ Posts Table

| Field      | Type     | Notes         |
| ---------- | -------- | ------------- |
| id         | int      | PK            |
| title      | str      |               |
| content    | text     |               |
| author_id  | int      | FK → Users.id |
| created_at | datetime | Default now   |
| updated_at | datetime | Nullable      |

---

## 🔁 Project Structure

```
blog-api/
├─ app/
│  ├─ main.py
│  ├─ models.py
│  ├─ schemas.py
│  ├─ database.py
│  ├─ auth.py
│  ├─ dependencies.py
│  ├─ routers/
│  │   ├─ users.py
│  │   └─ posts.py
├─ requirements.txt
```

---

## 🔹 JWT Authentication

### utils/auth.py

```python
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext

SECRET_KEY = "YOUR_SECRET_KEY"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```

---

## 🔹 Role-Based Dependency

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/login")
SECRET_KEY = "YOUR_SECRET_KEY"
ALGORITHM = "HS256"

def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = payload.get("user_id")
        role = payload.get("role")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return {"id": user_id, "role": role}
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

def require_role(roles: list):
    def role_checker(current_user: dict = Depends(get_current_user)):
        if current_user["role"] not in roles:
            raise HTTPException(status_code=403, detail="Forbidden")
        return current_user
    return role_checker
```

---

## 🔹 Users Router (Registration/Login)

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.models import User
from app.auth import hash_password, verify_password, create_access_token

router = APIRouter()

@router.post("/register")
def register(username: str, email: str, password: str, db: Session = Depends(get_db)):
    hashed = hash_password(password)
    user = User(username=username, email=email, password=hashed, role="user")
    db.add(user)
    db.commit()
    db.refresh(user)
    return {"id": user.id, "username": user.username}

@router.post("/login")
def login(email: str, password: str, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == email).first()
    if not user or not verify_password(password, user.password):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    token = create_access_token({"user_id": user.id, "role": user.role})
    return {"access_token": token, "token_type": "bearer"}
```

---

## 🔹 Posts Router (CRUD + Role-Based)

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.models import Post
from app.dependencies import get_current_user, require_role

router = APIRouter(prefix="/posts")

# Create post (any user)
@router.post("/")
def create_post(title: str, content: str, db: Session = Depends(get_db),
                current_user: dict = Depends(get_current_user)):
    post = Post(title=title, content=content, author_id=current_user["id"])
    db.add(post)
    db.commit()
    db.refresh(post)
    return post

# Get all posts (any user)
@router.get("/")
def get_posts(db: Session = Depends(get_db)):
    return db.query(Post).all()

# Update post (owner or admin)
@router.put("/{post_id}")
def update_post(post_id: int, title: str, content: str, db: Session = Depends(get_db),
                current_user: dict = Depends(get_current_user)):
    post = db.query(Post).filter(Post.id == post_id).first()
    if not post:
        raise HTTPException(404, "Post not found")
    if post.author_id != current_user["id"] and current_user["role"] != "admin":
        raise HTTPException(403, "Forbidden")
    post.title = title
    post.content = content
    db.commit()
    db.refresh(post)
    return post

# Delete post (owner or admin)
@router.delete("/{post_id}")
def delete_post(post_id: int, db: Session = Depends(get_db),
                current_user: dict = Depends(get_current_user)):
    post = db.query(Post).filter(Post.id == post_id).first()
    if not post:
        raise HTTPException(404, "Post not found")
    if post.author_id != current_user["id"] and current_user["role"] != "admin":
        raise HTTPException(403, "Forbidden")
    db.delete(post)
    db.commit()
    return {"detail": "Deleted"}
```

---

## 🔹 Pagination & Filtering for Posts

```python
@router.get("/paginated")
def get_posts_paginated(page: int = 1, limit: int = 10, db: Session = Depends(get_db)):
    offset = (page - 1) * limit
    posts = db.query(Post).offset(offset).limit(limit).all()
    total = db.query(Post).count()
    return {"page": page, "limit": limit, "total": total, "posts": posts}
```

---

## ✅ Best Practices

1. **Use JWT for stateless authentication**
2. **Centralize role-based access checks**
3. **Validate user inputs** (Pydantic schemas)
4. **Hash passwords** (never store plain passwords)
5. **Use pagination and filtering** for posts
6. **Log sensitive actions** (create/update/delete)
7. **Secure sensitive endpoints with HTTPS**

---

## 🌍 Real-World Use Cases

* Blogging platforms (Medium, Dev.to)
* SaaS admin dashboards
* Multi-user content platforms
* Learning management systems

---

## 📝 Summary

* Users can register/login → JWT token issued
* Role-based access → Admin vs User permissions
* CRUD operations on posts → Owners can edit/delete, Admin can manage all
* Pagination/filtering → efficient retrieval of posts
* Secure authentication & authorization → core backend principle

---

I can also make a **visual flow diagram** of this **Blog API with role-based access**, showing **auth → JWT → role check → CRUD endpoints**, which is perfect for **interviews and project documentation**.

Do you want me to create that next?
