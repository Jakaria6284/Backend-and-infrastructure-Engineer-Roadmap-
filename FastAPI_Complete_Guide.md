# 🚀 FastAPI — Complete Guide: Basic to Advanced

> **Who is this for?**  
> Anyone who wants to learn FastAPI — whether you've never built an API before, or you're a developer looking to go deep. We start from zero and build up to production-ready patterns, including databases with SQLAlchemy and PostgreSQL.

---

## 📚 Table of Contents

1. [What is FastAPI?](#1-what-is-fastapi)
2. [Installation & First App](#2-installation--first-app)
3. [Path Parameters & Query Parameters](#3-path-parameters--query-parameters)
4. [Request Body & Pydantic Models](#4-request-body--pydantic-models)
5. [Response Models & Status Codes](#5-response-models--status-codes)
6. [CRUD — Building a Full REST API](#6-crud--building-a-full-rest-api)
7. [Routers — Organizing Your Code](#7-routers--organizing-your-code)
8. [Dependency Injection](#8-dependency-injection)
9. [Middleware](#9-middleware)
10. [Error Handling](#10-error-handling)
11. [Authentication & JWT Tokens](#11-authentication--jwt-tokens)
12. [File Uploads](#12-file-uploads)
13. [Background Tasks](#13-background-tasks)
14. [WebSockets](#14-websockets)
15. [SQLAlchemy + PostgreSQL](#15-sqlalchemy--postgresql)
16. [Alembic — Database Migrations](#16-alembic--database-migrations)
17. [Testing FastAPI](#17-testing-fastapi)
18. [Project Structure (Production)](#18-project-structure-production)
19. [Deployment](#19-deployment)

---

## 1. What is FastAPI?

**FastAPI** is a modern Python web framework for building APIs. It was created by Sebastián Ramírez and is built on top of two powerful libraries:

- **Starlette** — for the web/HTTP parts
- **Pydantic** — for data validation

### Why FastAPI?

| Feature | What it means for you |
|---|---|
| ⚡ Fast | One of the fastest Python frameworks (on par with Node.js) |
| 🧠 Smart | Automatic data validation — no manual `if` checks needed |
| 📄 Auto Docs | API documentation is generated for you automatically |
| 🐍 Type hints | Uses Python's built-in type hints — code is clean and readable |
| 🔒 Secure | Easy to add authentication and security |

### The Big Picture

When you build a FastAPI app, you are building an **API** (Application Programming Interface) — a system that receives requests (from browsers, mobile apps, or other services) and returns responses (usually JSON data).

```
Client (browser / app)
        │
        │  HTTP Request: GET /users/1
        ▼
  ┌─────────────┐
  │   FastAPI   │  ← Your code lives here
  └─────────────┘
        │
        │  Response: {"id": 1, "name": "Alice"}
        ▼
Client receives the data
```

---

## 2. Installation & First App

### Install

```bash
pip install fastapi uvicorn
```

- `fastapi` — the framework itself
- `uvicorn` — the server that runs your app (like a waiter that takes requests and passes them to your kitchen)

### Your First App

Create a file called `main.py`:

```python
from fastapi import FastAPI

# Create the app
app = FastAPI()

# Create your first endpoint (route)
@app.get("/")
def read_root():
    return {"message": "Hello, World!"}
```

### Run the App

```bash
uvicorn main:app --reload
```

- `main` → the file name (`main.py`)
- `app` → the variable name (`app = FastAPI()`)
- `--reload` → auto-restart when you change code (great for development)

### Visit the App

Open your browser and go to:

| URL | What you see |
|---|---|
| `http://127.0.0.1:8000/` | Your API response |
| `http://127.0.0.1:8000/docs` | 🎉 Interactive API docs (Swagger UI) |
| `http://127.0.0.1:8000/redoc` | Alternative API docs |

> **Tip:** The `/docs` page is one of FastAPI's superpowers — it's generated automatically and lets you test your API right in the browser.

---

## 3. Path Parameters & Query Parameters

These are the two main ways a client can send data to your API through the URL.

### Path Parameters

Path parameters are part of the URL itself.

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

- `{user_id}` is a placeholder in the URL
- `user_id: int` tells FastAPI to expect an integer — if someone passes `"abc"`, FastAPI automatically returns an error

**Example request:** `GET /users/42` → returns `{"user_id": 42}`

### Multiple Path Parameters

```python
@app.get("/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    return {"user_id": user_id, "post_id": post_id}
```

### Query Parameters

Query parameters come after a `?` in the URL.

```python
@app.get("/items/")
def get_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

**Example request:** `GET /items/?skip=20&limit=5` → returns `{"skip": 20, "limit": 5}`

- Parameters with default values (`= 0`) are **optional**
- Parameters without defaults are **required**

### Optional Parameters

```python
from typing import Optional

@app.get("/search/")
def search(q: Optional[str] = None):
    if q:
        return {"query": q, "results": ["item1", "item2"]}
    return {"results": "all items"}
```

---

## 4. Request Body & Pydantic Models

When a client sends data (like creating a new user), it sends a **request body** — usually JSON. FastAPI uses **Pydantic models** to define and validate this data.

### What is Pydantic?

Pydantic is a data validation library. You define what your data should look like, and Pydantic checks it for you automatically.

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str
```

This is called a **schema** — a blueprint for your data.

### Using a Model in an Endpoint

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    in_stock: bool = True  # optional field with default

@app.post("/items/")
def create_item(item: Item):
    return {"item_name": item.name, "item_price": item.price}
```

**What FastAPI does automatically:**
1. Reads the JSON body from the request
2. Validates the data (is `price` a number? is `name` a string?)
3. Converts it to an `Item` object
4. Returns a helpful error if validation fails

### Pydantic Field Validation

```python
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    name: str = Field(min_length=2, max_length=50, description="User's full name")
    age: int = Field(ge=0, le=120, description="Age in years")
    email: EmailStr  # validates email format automatically
    password: str = Field(min_length=8)
```

> **Install EmailStr support:** `pip install pydantic[email]`

### Nested Models

```python
class Address(BaseModel):
    street: str
    city: str
    country: str

class UserWithAddress(BaseModel):
    name: str
    address: Address  # nested model
```

---

## 5. Response Models & Status Codes

### Response Models

You can control exactly what data your API returns using a `response_model`. This is important for security — you can return a database user object without exposing the password.

```python
class UserCreate(BaseModel):
    name: str
    email: str
    password: str  # we accept this...

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    # ...but we never return the password

@app.post("/users/", response_model=UserResponse)
def create_user(user: UserCreate):
    # In real code, you'd save to database here
    # FastAPI will automatically exclude 'password' from the response
    return {"id": 1, "name": user.name, "email": user.email}
```

### HTTP Status Codes

HTTP status codes tell the client what happened. Common ones:

| Code | Meaning |
|---|---|
| 200 | OK — success |
| 201 | Created — a new resource was created |
| 400 | Bad Request — the client sent invalid data |
| 401 | Unauthorized — not logged in |
| 403 | Forbidden — logged in but no permission |
| 404 | Not Found |
| 500 | Internal Server Error |

```python
from fastapi import status

@app.post("/users/", status_code=status.HTTP_201_CREATED)
def create_user(user: UserCreate):
    return user
```

---

## 6. CRUD — Building a Full REST API

CRUD stands for **Create, Read, Update, Delete** — the four basic operations of any API.

Let's build a complete CRUD API for blog posts.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

# --- Schema ---
class PostCreate(BaseModel):
    title: str
    content: str

class PostUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None

class Post(BaseModel):
    id: int
    title: str
    content: str

# --- Fake "database" (just a Python list for now) ---
posts_db = []
next_id = 1

# --- CREATE ---
@app.post("/posts/", response_model=Post, status_code=201)
def create_post(post: PostCreate):
    global next_id
    new_post = Post(id=next_id, title=post.title, content=post.content)
    posts_db.append(new_post)
    next_id += 1
    return new_post

# --- READ ALL ---
@app.get("/posts/", response_model=List[Post])
def get_posts():
    return posts_db

# --- READ ONE ---
@app.get("/posts/{post_id}", response_model=Post)
def get_post(post_id: int):
    for post in posts_db:
        if post.id == post_id:
            return post
    raise HTTPException(status_code=404, detail="Post not found")

# --- UPDATE ---
@app.put("/posts/{post_id}", response_model=Post)
def update_post(post_id: int, updated: PostUpdate):
    for post in posts_db:
        if post.id == post_id:
            if updated.title is not None:
                post.title = updated.title
            if updated.content is not None:
                post.content = updated.content
            return post
    raise HTTPException(status_code=404, detail="Post not found")

# --- DELETE ---
@app.delete("/posts/{post_id}", status_code=204)
def delete_post(post_id: int):
    for i, post in enumerate(posts_db):
        if post.id == post_id:
            posts_db.pop(i)
            return
    raise HTTPException(status_code=404, detail="Post not found")
```

### HTTPException

`HTTPException` is how you return error responses with custom messages.

```python
raise HTTPException(
    status_code=404,
    detail="The item you're looking for does not exist"
)
```

---

## 7. Routers — Organizing Your Code

As your app grows, putting everything in one file becomes messy. FastAPI **routers** let you split your code into multiple files.

### Folder Structure

```
my_app/
├── main.py
└── routers/
    ├── users.py
    └── posts.py
```

### `routers/users.py`

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",      # all routes start with /users
    tags=["Users"],       # groups routes in the docs
)

@router.get("/")
def get_users():
    return [{"id": 1, "name": "Alice"}]

@router.get("/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "Alice"}
```

### `main.py`

```python
from fastapi import FastAPI
from routers import users, posts

app = FastAPI()

app.include_router(users.router)
app.include_router(posts.router)

@app.get("/")
def root():
    return {"message": "Welcome to my API"}
```

Now `GET /users/` and `GET /users/1` both work without touching `main.py`.

---

## 8. Dependency Injection

**Dependency Injection** (DI) is a powerful pattern that lets you share logic across multiple routes without repeating code. Think of it as saying: "Before running my route, first run this function and give me the result."

### Simple Example — Pagination

Instead of repeating `skip` and `limit` parameters in every route:

```python
from fastapi import Depends

def pagination_params(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

@app.get("/items/")
def get_items(params: dict = Depends(pagination_params)):
    return params

@app.get("/users/")
def get_users(params: dict = Depends(pagination_params)):
    return params
```

Both routes now automatically have `?skip=` and `?limit=` parameters.

### Class-Based Dependencies

```python
class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 10):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/products/")
def get_products(commons: CommonQueryParams = Depends(CommonQueryParams)):
    return {"search": commons.q, "skip": commons.skip}
```

### Database Session Dependency (Preview)

Dependencies are used to inject database sessions — we'll see this in the SQLAlchemy section:

```python
def get_db():
    db = SessionLocal()
    try:
        yield db  # give the db to the route
    finally:
        db.close()  # always close when done

@app.get("/users/")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

---

## 9. Middleware

**Middleware** is code that runs before and after every request. It sits in the middle between the client and your routes — like a security checkpoint.

### Built-in CORS Middleware

CORS (Cross-Origin Resource Sharing) controls which websites can access your API. This is essential when building a frontend + backend setup.

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://mywebsite.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Custom Middleware

```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    
    response = await call_next(request)  # run the actual route
    
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

This middleware logs how long each request takes and adds it to the response headers.

---

## 10. Error Handling

### Custom Exception Handlers

Instead of raising `HTTPException` everywhere, you can define global handlers:

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class ItemNotFoundException(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

@app.exception_handler(ItemNotFoundException)
async def item_not_found_handler(request: Request, exc: ItemNotFoundException):
    return JSONResponse(
        status_code=404,
        content={"message": f"Item with id {exc.item_id} was not found"}
    )

@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id > 100:
        raise ItemNotFoundException(item_id=item_id)
    return {"item": item_id}
```

### Validation Error Handler

FastAPI automatically handles Pydantic validation errors, but you can customize the response:

```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={"detail": exc.errors(), "message": "Invalid data provided"}
    )
```

---

## 11. Authentication & JWT Tokens

**JWT (JSON Web Token)** is the most common way to secure APIs. Here's how it works:

1. User logs in with username + password
2. Server verifies credentials and gives the user a **token** (a signed string)
3. User sends this token with every future request
4. Server verifies the token and knows who the user is

### Install

```bash
pip install python-jose[cryptography] passlib[bcrypt] python-multipart
```

### Full Auth Implementation

```python
from datetime import datetime, timedelta
from typing import Optional

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# --- Config ---
SECRET_KEY = "your-secret-key-keep-it-safe"  # change this in production!
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

app = FastAPI()

# --- Password hashing ---
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

# --- JWT ---
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode["exp"] = expire
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# --- Fake DB ---
fake_users = {
    "alice": {
        "username": "alice",
        "hashed_password": hash_password("secret123")
    }
}

# --- Schemas ---
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

# --- Get current user from token ---
def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = fake_users.get(username)
    if user is None:
        raise credentials_exception
    return user

# --- Login route ---
@app.post("/token", response_model=Token)
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = fake_users.get(form_data.username)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    token = create_access_token(
        data={"sub": user["username"]},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": token, "token_type": "bearer"}

# --- Protected route ---
@app.get("/me")
def read_me(current_user: dict = Depends(get_current_user)):
    return {"username": current_user["username"]}
```

---

## 12. File Uploads

```python
from fastapi import UploadFile, File
import shutil

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    # Save the file to disk
    with open(f"uploads/{file.filename}", "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)

    return {
        "filename": file.filename,
        "content_type": file.content_type,
    }

# Upload multiple files
@app.post("/upload-multiple/")
async def upload_files(files: List[UploadFile] = File(...)):
    return [{"filename": f.filename} for f in files]
```

---

## 13. Background Tasks

Sometimes you want to do work **after** sending a response — like sending a welcome email after registration, without making the user wait.

```python
from fastapi import BackgroundTasks

def send_welcome_email(email: str):
    # This runs after the response is sent
    print(f"Sending welcome email to {email}...")
    # imagine email sending logic here

@app.post("/register/")
def register_user(email: str, background_tasks: BackgroundTasks):
    # ... save user to database ...
    background_tasks.add_task(send_welcome_email, email)
    return {"message": "User registered! Welcome email will be sent shortly."}
```

---

## 14. WebSockets

WebSockets enable **real-time, two-way communication** — perfect for chat apps, live dashboards, or notifications.

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()  # accept the connection
    
    while True:
        data = await websocket.receive_text()  # wait for message
        await websocket.send_text(f"Echo: {data}")  # send back

# Simple chat room
connected_clients = []

@app.websocket("/chat")
async def chat_endpoint(websocket: WebSocket):
    await websocket.accept()
    connected_clients.append(websocket)
    
    try:
        while True:
            message = await websocket.receive_text()
            # Broadcast to all clients
            for client in connected_clients:
                await client.send_text(message)
    except:
        connected_clients.remove(websocket)
```

---

## 15. SQLAlchemy + PostgreSQL

This is where things get real. Instead of storing data in memory (a Python list), we'll store it in a **PostgreSQL database** using **SQLAlchemy** as our ORM.

### What is an ORM?

An ORM (Object-Relational Mapper) lets you talk to your database using Python objects instead of writing raw SQL. SQLAlchemy is the most popular ORM for Python.

```
Without ORM: "SELECT * FROM users WHERE id = 1"
With ORM:    db.query(User).filter(User.id == 1).first()
```

### Install

```bash
pip install sqlalchemy psycopg2-binary
```

- `sqlalchemy` — the ORM
- `psycopg2-binary` — the PostgreSQL driver (connects Python to PostgreSQL)

---

### Project Structure

```
my_app/
├── main.py
├── database.py       # DB connection setup
├── models.py         # SQLAlchemy models (tables)
├── schemas.py        # Pydantic models (validation)
└── routers/
    └── users.py
```

---

### `database.py` — Database Connection

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

# Your PostgreSQL connection string
# Format: postgresql://username:password@host:port/database_name
DATABASE_URL = "postgresql://postgres:password@localhost:5432/mydb"

# Create the engine (the connection to the database)
engine = create_engine(DATABASE_URL)

# Session factory — used to talk to the database
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base class for all our models
class Base(DeclarativeBase):
    pass

# Dependency — creates a DB session per request, then closes it
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

### `models.py` — Database Tables

SQLAlchemy **models** define your database tables using Python classes.

```python
from sqlalchemy import Column, Integer, String, Boolean, ForeignKey, DateTime, func
from sqlalchemy.orm import relationship
from database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

    # Relationship — a user can have many posts
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(String, nullable=False)
    published = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

    # Foreign key — links post to a user
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    # Relationship — back reference to the user
    author = relationship("User", back_populates="posts")
```

---

### `schemas.py` — Pydantic Validation Schemas

We have **two types of schemas** for each resource:

- **Input schemas** (what we receive from the client)
- **Output schemas** (what we send back)

```python
from pydantic import BaseModel, EmailStr
from datetime import datetime
from typing import Optional, List

# --- User Schemas ---
class UserCreate(BaseModel):
    name: str
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    is_active: bool
    created_at: datetime

    class Config:
        from_attributes = True  # allows reading from SQLAlchemy objects

class UserWithPosts(UserResponse):
    posts: List["PostResponse"] = []

# --- Post Schemas ---
class PostCreate(BaseModel):
    title: str
    content: str
    published: bool = True

class PostUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None
    published: Optional[bool] = None

class PostResponse(BaseModel):
    id: int
    title: str
    content: str
    published: bool
    created_at: datetime
    author_id: int

    class Config:
        from_attributes = True

# Update forward reference
UserWithPosts.model_rebuild()
```

> **Why `from_attributes = True`?**  
> SQLAlchemy objects are not dicts. This tells Pydantic how to read data from them.

---

### `routers/users.py` — CRUD with Real Database

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from passlib.context import CryptContext

import models, schemas
from database import get_db

router = APIRouter(prefix="/users", tags=["Users"])
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# --- CREATE USER ---
@router.post("/", response_model=schemas.UserResponse, status_code=201)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    # Check if email already exists
    existing = db.query(models.User).filter(models.User.email == user.email).first()
    if existing:
        raise HTTPException(status_code=400, detail="Email already registered")

    hashed_pw = pwd_context.hash(user.password)
    db_user = models.User(
        name=user.name,
        email=user.email,
        hashed_password=hashed_pw
    )
    db.add(db_user)
    db.commit()          # save to database
    db.refresh(db_user)  # reload with generated id, timestamps, etc.
    return db_user

# --- GET ALL USERS ---
@router.get("/", response_model=List[schemas.UserResponse])
def get_users(skip: int = 0, limit: int = 10, db: Session = Depends(get_db)):
    return db.query(models.User).offset(skip).limit(limit).all()

# --- GET ONE USER ---
@router.get("/{user_id}", response_model=schemas.UserWithPosts)
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(models.User).filter(models.User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# --- UPDATE USER ---
@router.put("/{user_id}", response_model=schemas.UserResponse)
def update_user(user_id: int, updated: schemas.UserCreate, db: Session = Depends(get_db)):
    user = db.query(models.User).filter(models.User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")

    user.name = updated.name
    user.email = updated.email
    db.commit()
    db.refresh(user)
    return user

# --- DELETE USER ---
@router.delete("/{user_id}", status_code=204)
def delete_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(models.User).filter(models.User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")

    db.delete(user)
    db.commit()
```

---

### `routers/posts.py`

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List

import models, schemas
from database import get_db

router = APIRouter(prefix="/posts", tags=["Posts"])

@router.post("/", response_model=schemas.PostResponse, status_code=201)
def create_post(post: schemas.PostCreate, author_id: int, db: Session = Depends(get_db)):
    db_post = models.Post(**post.model_dump(), author_id=author_id)
    db.add(db_post)
    db.commit()
    db.refresh(db_post)
    return db_post

@router.get("/", response_model=List[schemas.PostResponse])
def get_posts(published: bool = True, db: Session = Depends(get_db)):
    return db.query(models.Post).filter(models.Post.published == published).all()

@router.get("/{post_id}", response_model=schemas.PostResponse)
def get_post(post_id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == post_id).first()
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    return post

@router.put("/{post_id}", response_model=schemas.PostResponse)
def update_post(post_id: int, updated: schemas.PostUpdate, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == post_id)
    if not post.first():
        raise HTTPException(status_code=404, detail="Post not found")
    
    post.update(updated.model_dump(exclude_unset=True))
    db.commit()
    return post.first()

@router.delete("/{post_id}", status_code=204)
def delete_post(post_id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == post_id).first()
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    db.delete(post)
    db.commit()
```

---

### `main.py` — Putting it all together

```python
from fastapi import FastAPI
from database import Base, engine
from routers import users, posts

# Create all tables in the database
Base.metadata.create_all(bind=engine)

app = FastAPI(title="My Blog API", version="1.0.0")

app.include_router(users.router)
app.include_router(posts.router)

@app.get("/")
def root():
    return {"message": "Blog API is running"}
```

---

### SQLAlchemy Query Cheat Sheet

```python
# Get all records
db.query(User).all()

# Get one by ID
db.query(User).filter(User.id == 1).first()

# Filter with multiple conditions
db.query(User).filter(User.is_active == True, User.age > 18).all()

# Ordering
db.query(Post).order_by(Post.created_at.desc()).all()

# Pagination
db.query(Post).offset(0).limit(10).all()

# Count
db.query(User).count()

# Searching (LIKE)
db.query(User).filter(User.name.ilike("%alice%")).all()

# Join
db.query(Post).join(User).filter(User.name == "Alice").all()
```

---

## 16. Alembic — Database Migrations

When you change your database models (add a column, rename a table), you need **migrations** — controlled changes to your database schema. Alembic handles this.

### Install & Init

```bash
pip install alembic
alembic init alembic
```

This creates an `alembic/` folder and `alembic.ini`.

### Configure `alembic.ini`

```ini
sqlalchemy.url = postgresql://postgres:password@localhost:5432/mydb
```

### Configure `alembic/env.py`

```python
from database import Base
import models  # important: import models so Alembic can detect them

target_metadata = Base.metadata
```

### Create a Migration

```bash
# Auto-generate migration from model changes
alembic revision --autogenerate -m "add users table"

# Run the migration (apply to database)
alembic upgrade head

# See migration history
alembic history

# Roll back one step
alembic downgrade -1
```

---

## 17. Testing FastAPI

FastAPI provides a built-in test client powered by `httpx`.

### Install

```bash
pip install pytest httpx
```

### `test_main.py`

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

# Test root endpoint
def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Blog API is running"}

# Test creating a user
def test_create_user():
    response = client.post("/users/", json={
        "name": "Alice",
        "email": "alice@example.com",
        "password": "secret123"
    })
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "alice@example.com"
    assert "password" not in data  # password must not be in response

# Test user not found
def test_get_nonexistent_user():
    response = client.get("/users/9999")
    assert response.status_code == 404
```

### Run Tests

```bash
pytest test_main.py -v
```

### Testing with a Database

Use a separate test database and override the `get_db` dependency:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from database import Base, get_db
from main import app
from fastapi.testclient import TestClient

TEST_DATABASE_URL = "sqlite:///./test.db"  # use SQLite for tests
engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def override_get_db():
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db
Base.metadata.create_all(bind=engine)

client = TestClient(app)
```

---

## 18. Project Structure (Production)

A well-organized FastAPI project looks like this:

```
my_project/
│
├── app/
│   ├── __init__.py
│   ├── main.py               # FastAPI app entry point
│   ├── database.py           # DB connection
│   ├── models.py             # SQLAlchemy models
│   ├── schemas.py            # Pydantic schemas
│   ├── config.py             # App settings (env vars)
│   │
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── users.py
│   │   └── posts.py
│   │
│   ├── services/             # Business logic (separate from routes)
│   │   ├── user_service.py
│   │   └── post_service.py
│   │
│   └── dependencies.py       # Shared dependencies (auth, db, etc.)
│
├── alembic/                  # DB migrations
├── tests/                    # Test files
├── .env                      # Environment variables (never commit this!)
├── requirements.txt
└── docker-compose.yml
```

### `config.py` — Environment Variables

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

### `.env` File

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/mydb
SECRET_KEY=your-very-secret-key-here
```

> **Never commit `.env` to version control.** Add it to `.gitignore`.

---

## 19. Deployment

### Option 1: Run with Uvicorn (Simple)

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Option 2: Docker

**`Dockerfile`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**`docker-compose.yml`**

```yaml
version: "3.8"

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

**Run everything:**

```bash
docker-compose up --build
```

### Option 3: Gunicorn + Uvicorn Workers (Production)

```bash
pip install gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

---

## 🗺️ Learning Path Summary

Here's the order to master FastAPI:

```
Week 1 — Foundations
  ├── Install FastAPI + run first app
  ├── Path & query parameters
  ├── Pydantic models
  └── CRUD with in-memory data

Week 2 — Intermediate
  ├── Routers & project structure
  ├── Dependency injection
  ├── Middleware & CORS
  └── Error handling

Week 3 — Database
  ├── PostgreSQL setup
  ├── SQLAlchemy models
  ├── Full CRUD with real database
  └── Alembic migrations

Week 4 — Advanced
  ├── JWT Authentication
  ├── File uploads
  ├── Background tasks
  ├── Writing tests
  └── Docker deployment
```

---

## 📎 Quick Reference

### Common Imports

```python
from fastapi import FastAPI, HTTPException, Depends, status, UploadFile, File, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel, Field, EmailStr
from sqlalchemy.orm import Session
from typing import Optional, List
```

### HTTP Methods at a Glance

| Method | Decorator | Use Case |
|---|---|---|
| GET | `@app.get()` | Retrieve data |
| POST | `@app.post()` | Create new data |
| PUT | `@app.put()` | Replace existing data |
| PATCH | `@app.patch()` | Partially update data |
| DELETE | `@app.delete()` | Delete data |

### SQLAlchemy Column Types

| Python | SQLAlchemy | PostgreSQL |
|---|---|---|
| `str` | `String(n)` | `VARCHAR(n)` |
| `int` | `Integer` | `INTEGER` |
| `float` | `Float` | `FLOAT` |
| `bool` | `Boolean` | `BOOLEAN` |
| `datetime` | `DateTime` | `TIMESTAMP` |
| `text` | `Text` | `TEXT` |

---

## 📚 Further Resources

- **Official Docs:** https://fastapi.tiangolo.com
- **SQLAlchemy Docs:** https://docs.sqlalchemy.org
- **Pydantic Docs:** https://docs.pydantic.dev
- **Alembic Docs:** https://alembic.sqlalchemy.org

---

*Built with ❤️ to help learners go from zero to production-ready FastAPI apps.*
