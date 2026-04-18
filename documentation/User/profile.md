Here’s a **production-ready `user_profile.md`** you can directly use for implementation and reference.

---

# 📄 `profile.md` — User Profile & Password Management

## 1. Overview

This module handles:

- Viewing user profile details
- Updating profile information
- Secure password change flow
- Authenticated access using JWT (Bearer Token)

---

## 2. Features

### 👤 User Profile

- View personal details
- Update editable fields

### 🔐 Change Password

- Verify current password
- Set new password
- Enforce password rules

---

## 3. Authentication Requirement

All profile-related APIs must require:

```http
Authorization: Bearer <access_token>
```

- Token is validated
- User is extracted from JWT
- Unauthorized users → `401`

---

## 4. User Schema (Reference)

```python
id: UUID
email: str (required, unique)
name: str (required)
password: hashed (required)
phone: str | None
date_of_birth: date (required)
created_at: datetime
updated_at: datetime
```

---

## 5. API Endpoints

---

### 🔹 1. Get User Profile

```http
GET /user/profile
```

**Response:**

```json
{
  "id": "uuid",
  "name": "Dhyana",
  "email": "user@gmail.com",
  "phone": "9876543210",
  "date_of_birth": "2002-01-01",
  "created_at": "2026-04-18T10:00:00"
}
```

---

### 🔹 2. Update Profile

```http
PUT /user/profile
```

**Request Body:**

```json
{
  "name": "New Name",
  "phone": "9999999999"
}
```

**Rules:**

- Email should NOT be updated (avoid identity issues)
- Only allow safe fields (name, phone)

---

### 🔹 3. Change Password

```http
POST /user/change-password
```

**Request Body:**

```json
{
  "current_password": "oldpassword",
  "new_password": "newStrongPassword123"
}
```

---

## 6. Change Password Flow (Important)

### Step-by-Step:

1. User sends:
   - current_password
   - new_password

2. Backend:
   - Fetch user from token
   - Verify current password (bcrypt check)

3. If incorrect:

```json
{
  "error": "Current password is incorrect"
}
```

4. If correct:
   - Hash new password
   - Update DB
   - Invalidate old refresh tokens (optional but recommended)

5. Response:

```json
{
  "message": "Password updated successfully"
}
```

---

## 7. Password Validation Rules

- Minimum length: 8
- Must include:
  - Uppercase
  - Lowercase
  - Number
  - Special character (recommended)

---

## 8. FastAPI Implementation (Core Logic)

### Password Verification

```python
def change_password(user, current_password, new_password):
    if not verify_password(current_password, user.password):
        raise Exception("Invalid current password")

    user.password = hash_password(new_password)
    db.commit()
```

---

### Route Example

```python
@router.post("/user/change-password")
def change_password_api(data: ChangePasswordSchema, user=Depends(get_current_user)):
    return change_password(user, data.current_password, data.new_password)
```

---

## 9. Database Update

```sql
UPDATE users
SET password = 'hashed_password',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 'user_id';
```

---

## 10. Security Best Practices

- Always hash passwords (bcrypt)
- Never return password in API response
- Use HTTPS
- Rate limit password attempts
- Log suspicious activity

---

## 11. Optional Enhancements (Recommended)

- 🔁 Logout from all devices after password change
- 📩 Email notification: “Your password was changed”
- 🔐 Add 2FA (future scope)

---

## 12. Testing

### ✅ Get Profile

- Valid token → success
- Invalid token → 401

---

### ✅ Update Profile

```json
{
  "name": "Test User Updated",
  "phone": "8888888888"
}
```

---

### ✅ Change Password (Success)

```json
{
  "current_password": "123456",
  "new_password": "NewPass@123"
}
```

---

### ❌ Change Password (Fail)

```json
{
  "current_password": "wrongpassword",
  "new_password": "NewPass@123"
}
```

---

## 13. Folder Structure

```bash
app/
│── routes/
│   └── user.py
│── schemas/
│   └── user_schema.py
│── services/
│   └── user_service.py
```

---

## 14. UI Behavior (Frontend Reference)

### Profile Page:

- Show:
  - Name
  - Email (readonly)
  - Phone
  - DOB

### Change Password Section:

- Input:
  - Current Password
  - New Password
  - Confirm Password

### UX Flow:

- Validate before API call
- Show success/error messages

---

## 15. Final Notes

- Keep profile APIs **secure and minimal**
- Avoid over-exposing user data
- Always validate inputs strictly

---
