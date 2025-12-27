Perfect! Let’s build a **full Auth System project using FastAPI** with **JWT access + refresh tokens**. I’ll explain it **step-by-step**, with **code structure, Pydantic models, security, and best practices**.

---

# 🔐 FastAPI Auth System Project — JWT + Refresh Token

---

## 1️⃣ Project Structure

```
fastapi_auth/
├─ app/
│  ├─ main.py
│  ├─ models.py           # Pydantic models
│  ├─ schemas.py          # Request/response schemas
│  ├─ database.py         # Database setup (SQLAlchemy or SQLite)
│  ├─ auth.py             # JWT utils
│  ├─ dependencies.py     # Dependencies (get_current_user)
│  └─ routers/
│       └─ auth.py        # Auth routes (login, register, refresh)
└─ requirements.txt
```

---

## 2️⃣ Install Dependencies

```bash
pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] sqlalchemy pydantic email-validator python-multipart
```

* **fastapi** → API framework
* **uvicorn** → server
* **python-jose** → JWT encoding/decoding
* **passlib** → password hashing
* **sqlalchemy** → ORM
* **pydantic** → validation

---

## 3️⃣ Database Setup (SQLite Example)

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

---

## 4️⃣ Models (SQLAlchemy + Pydantic)

### SQLAlchemy User Model

```python
# models.py
from sqlalchemy import Column, Integer, String
from .database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
```

### Pydantic Schemas

```python
# schemas.py
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    email: EmailStr
    password: str

class UserOut(BaseModel):
    id: int
    email: EmailStr

    class Config:
        orm_mode = True

class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
```

---

## 5️⃣ JWT Utilities

```python
# auth.py
from datetime import datetime, timedelta
from jose import jwt

SECRET_KEY = "your_secret_key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 15
REFRESH_TOKEN_EXPIRE_DAYS = 7

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def create_refresh_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str):
    from jose import JWTError
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None
```

---

## 6️⃣ Password Hashing

```python
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)
```

---

## 7️⃣ Auth Routes

```python
# routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from app.database import SessionLocal
from app import models, schemas, auth

router = APIRouter(prefix="/auth")

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Register
@router.post("/register", response_model=schemas.UserOut)
def register(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = db.query(models.User).filter(models.User.email == user.email).first()
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    hashed_password = auth.hash_password(user.password)
    new_user = models.User(email=user.email, hashed_password=hashed_password)
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user

# Login
@router.post("/login", response_model=schemas.Token)
def login(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = db.query(models.User).filter(models.User.email == user.email).first()
    if not db_user or not auth.verify_password(user.password, db_user.hashed_password):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    access_token = auth.create_access_token({"sub": db_user.email})
    refresh_token = auth.create_refresh_token({"sub": db_user.email})
    return {"access_token": access_token, "refresh_token": refresh_token}

# Refresh token
@router.post("/refresh", response_model=schemas.Token)
def refresh(token: str):
    payload = auth.verify_token(token)
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid refresh token")
    email = payload.get("sub")
    access_token = auth.create_access_token({"sub": email})
    refresh_token = auth.create_refresh_token({"sub": email})
    return {"access_token": access_token, "refresh_token": refresh_token}
```

---

## 8️⃣ Protect Routes (JWT Dependency)

```python
# dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from app.auth import verify_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/login")

def get_current_user(token: str = Depends(oauth2_scheme)):
    payload = verify_token(token)
    if not payload:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    return payload.get("sub")
```

```python
# main.py
from fastapi import FastAPI, Depends
from app.routers import auth
from app.dependencies import get_current_user

app = FastAPI()
app.include_router(auth.router)

@app.get("/protected")
def protected_route(user_email: str = Depends(get_current_user)):
    return {"message": f"Hello {user_email}, you are authenticated!"}
```

---

## 9️⃣ Workflow Summary

1. **Register** → save hashed password
2. **Login** → generate access + refresh tokens
3. **Access protected route** → verify access token
4. **Refresh token** → generate new access + refresh tokens
5. **Logout** → optionally blacklist token

---

## 🔟 Best Practices ✅

* Store refresh tokens securely (DB or Redis)
* Short-lived access tokens + longer refresh tokens
* Use HTTPS in production
* Use **scopes / roles** for authorization
* Blacklist refresh tokens on logout

---

## 1️⃣1️⃣ Common Mistakes ❌

❌ Storing plain passwords
❌ Long-lived access tokens
❌ Not validating tokens in every protected route
❌ Not handling token expiration properly
❌ Returning sensitive info in JWT

---

## 1️⃣2️⃣ Final Summary 🧠

* **FastAPI + Pydantic → validate requests**
* **JWT access token** → short-lived auth
* **JWT refresh token** → renew access tokens
* **Middleware / Dependencies → protect routes**
* **Security best practices** → password hashing, HTTPS, token revocation

---

If you want, I can create a **full working mini-project** with **SQLite DB, JWT auth, refresh token workflow, and protected CRUD endpoints** that you can **run directly with Uvicorn**.

Do you want me to do that next?
