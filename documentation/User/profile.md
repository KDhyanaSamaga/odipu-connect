Here’s a **clean, production-oriented `login.md` reference document** you can directly use while building your system.

---

# 📄 `login.md` — Authentication & User Management Design

## 1. Overview

This module handles:

- User Registration (Signup)
- User Authentication (Login)
- Token Management (Access + Refresh Tokens)
- Password Reset Flow
- Secure API Access (Bearer Token)

Tech Stack:

- **Backend**: FastAPI
- **Language**: Python
- **Auth**: JWT (JSON Web Tokens)
- **Database**: PostgreSQL (Recommended) or MongoDB (Alternative)

---

## 2. Login & Signup Requirements

### 🔐 Login Fields

- `email` (required)
- `password` (required)

### 📝 Signup Fields

- `name` (required)
- `email` (required)
- `password` (required → hashed)
- `phone` (optional)
- `date_of_birth` (required)

---

## 3. Token Strategy

### Tokens Used:

1. **Access Token**
   - Short-lived (15–30 mins)
   - Used for API authentication

2. **Refresh Token**
   - Long-lived (7–30 days)
   - Used to generate new access tokens

3. **Bearer Token**
   - Format: `Authorization: Bearer <access_token>`

### Flow:

```
Login → Generate Access + Refresh Token
Access Token expires → Use Refresh Token → Get new Access Token
```

---

## 4. Tech Stack Decision: MongoDB vs PostgreSQL

### ✅ Recommended: PostgreSQL

**Why PostgreSQL is better for this module:**

- Strong schema enforcement (important for auth data)
- ACID compliance (data consistency)
- Better for relationships (users ↔ complaints)
- Works well with Alembic migrations

### ❗ MongoDB Use Case:

- If your entire system is already NoSQL-heavy
- Flexible schema (but less strict validation)

👉 **Final Recommendation:**

- Use **PostgreSQL for authentication**
- You can still use MongoDB for complaints if needed (hybrid architecture)

---

## 5. Database Model

### User Table (PostgreSQL)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    password TEXT NOT NULL,
    phone VARCHAR(15),
    date_of_birth DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Optional: Refresh Tokens Table

```sql
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    token TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 6. API Endpoints

### 🔑 Authentication Routes

#### 1. Signup

```
POST /signup
```

**Request Body:**

```json
{
  "name": "Dhyana",
  "email": "user@example.com",
  "password": "password123",
  "phone": "9876543210",
  "date_of_birth": "2002-01-01"
}
```

---

#### 2. Login

```
POST /login
```

**Response:**

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "bearer"
}
```

---

#### 3. Refresh Token

```
POST /token/refresh
```

---

#### 4. Forgot Password

```
POST /login/forgot-password
```

**Flow:**

- User enters email
- Send reset link/token via email

---

#### 5. Reset Password

```
POST /login/reset-password
```

---

#### 6. Get Current User

```
GET /me
```

Header:

```
Authorization: Bearer <access_token>
```

---

## 7. Implementation Details (FastAPI)

### Password Hashing

Use:

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(plain, hashed):
    return pwd_context.verify(plain, hashed)
```

---

### JWT Token Creation

```python
from jose import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your_secret"
ALGORITHM = "HS256"

def create_access_token(data: dict, expires_delta: int = 15):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=expires_delta)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```

---

### Dependency for Protected Routes

```python
from fastapi import Depends
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

def get_current_user(token: str = Depends(oauth2_scheme)):
    # decode token and fetch user
    pass
```

---

## 8. Alembic Setup (Database Migration)

### Install:

```bash
pip install alembic
```

### Initialize:

```bash
alembic init alembic
```

### Configure `alembic.ini`:

```
sqlalchemy.url = postgresql://user:password@localhost/dbname
```

---

### Create Migration:

```bash
alembic revision --autogenerate -m "create users table"
```

### Apply Migration:

```bash
alembic upgrade head
```

---

## 9. Testing

### Sample Test Cases

#### ✅ Signup Test

```json
{
  "name": "Test User",
  "email": "test@gmail.com",
  "password": "123456",
  "phone": "9999999999",
  "date_of_birth": "2000-01-01"
}
```

---

#### ✅ Login Test

```json
{
  "email": "test@gmail.com",
  "password": "123456"
}
```

---

### Edge Cases

- Invalid email format
- Weak password
- Duplicate email
- Expired token
- Invalid refresh token

---

## 10. Security Best Practices

- Always hash passwords (bcrypt)
- Use HTTPS only
- Store secrets in `.env`
- Implement rate limiting on login
- Add email verification (optional but recommended)
- Use token blacklisting (for logout)

---

## 11. Suggested Folder Structure

```
app/
│── main.py
│── models/
│   └── user.py
│── schemas/
│   └── user_schema.py
│── routes/
│   └── auth.py
│── services/
│   └── auth_service.py
│── core/
│   └── security.py
│── db/
│   └── database.py
```

---

## 12. Final Notes

- Keep auth **separate module**
- Design APIs **stateless**
- Make tokens **lightweight but secure**
- Plan future:
  - OAuth (Google login)
  - Role-based access (admin/officer/user)

---
