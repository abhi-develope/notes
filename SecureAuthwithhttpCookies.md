
# 🛡️ Secure Authentication with HttpOnly Cookies (React + Express + Axios)

## 📌 Table of Contents

1. [Introduction](#1-introduction)
2. [Why Use HttpOnly Cookies](#2-why-use-httponly-cookies)
3. [Auth Flow Overview](#3-auth-flow-overview)
4. [Frontend (React + Axios) Setup](#4-frontend-react--axios-setup)
5. [Backend (Express.js) Setup](#5-backend-expressjs-setup)
6. [Cookie Handling with `withCredentials`](#6-cookie-handling-with-withcredentials)
7. [Logout Flow](#7-logout-flow)
8. [Best Practices Summary](#8-best-practices-summary)
9. [FAQs](#9-faqs)

---

## 1. 📘 Introduction

Token-based authentication is common in web apps. While storing JWTs in `localStorage` is easy, it's **not secure** because JavaScript can access it (making it vulnerable to XSS).

**Recommended approach:** Use **HttpOnly cookies** to store tokens securely. These cookies:
- Cannot be accessed by JavaScript
- Are automatically sent with each HTTP request

---

## 2. 🧠 Why Use HttpOnly Cookies

| Method              | Secure? | Accessible in JS? | Auto-Sent with Requests |
|---------------------|---------|--------------------|--------------------------|
| `localStorage`      | ❌ No   | ✅ Yes             | ❌ No                    |
| `HttpOnly Cookie`   | ✅ Yes  | ❌ No              | ✅ Yes                   |

**Conclusion**: HttpOnly cookies are best for security and developer convenience.

---

## 3. 🔁 Auth Flow Overview

### 🧩 Steps:

1. User logs in or signs up.
2. Backend verifies credentials and sets a **JWT token in an HttpOnly cookie**.
3. Frontend stores nothing manually — browser stores the cookie.
4. On future requests, browser **automatically sends cookie**.
5. Backend checks the cookie to allow or deny access.

---

## 4. 💻 Frontend (React + Axios) Setup

### ➤ Install Axios

```bash
npm install axios
```

### ➤ Axios Configuration (`api.js`)

```js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://your-api.com/api',
  withCredentials: true, // 🔐 Important for sending/receiving cookies
});

export default api;
```

### ➤ Login Example

```js
// login.js
import api from './api';

api.post('/login', {
  email: 'user@example.com',
  password: '123456'
})
.then(res => {
  console.log('Login success:', res.data.message);
})
.catch(err => {
  console.error('Login failed:', err.response.data.message);
});
```

### ➤ Accessing a Protected Route

```js
// profile.js
api.get('/profile')
  .then(res => {
    console.log('User profile:', res.data);
  })
  .catch(err => {
    console.error('Unauthorized:', err.response.status);
  });
```

✅ **No need to manually attach tokens** — browser includes the cookie.

---

## 5. 🔧 Backend (Express.js) Setup

### ➤ Install Dependencies

```bash
npm install express cors cookie-parser jsonwebtoken
```

### ➤ Express Server Setup

```js
const express = require('express');
const cors = require('cors');
const cookieParser = require('cookie-parser');
const jwt = require('jsonwebtoken');

const app = express();
app.use(express.json());
app.use(cookieParser());

app.use(cors({
  origin: 'https://your-frontend.com',
  credentials: true // 🟢 Must be true to allow cookies
}));
```

### ➤ Login Route

```js
app.post('/api/login', (req, res) => {
  const { email, password } = req.body;

  // Dummy check (replace with DB check)
  if (email === 'user@example.com' && password === '123456') {
    const token = jwt.sign({ id: 1, email }, 'SECRET_KEY', { expiresIn: '1h' });

    res.cookie('token', token, {
      httpOnly: true,
      secure: true,      // Only over HTTPS
      sameSite: 'Strict',
      maxAge: 3600000    // 1 hour
    });

    res.json({ message: 'Login successful' });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
});
```

### ➤ Middleware to Protect Routes

```js
function authenticateToken(req, res, next) {
  const token = req.cookies.token;
  if (!token) return res.status(401).json({ message: 'Access denied' });

  try {
    const decoded = jwt.verify(token, 'SECRET_KEY');
    req.user = decoded;
    next();
  } catch (err) {
    res.status(403).json({ message: 'Invalid or expired token' });
  }
}
```

### ➤ Protected Route

```js
app.get('/api/profile', authenticateToken, (req, res) => {
  res.json({ id: req.user.id, email: req.user.email });
});
```

---

## 6. 🍪 Cookie Handling with `withCredentials`

| Use Case             | Required? |
|----------------------|-----------|
| Login / Signup       | ✅ Yes    |
| Protected Requests   | ✅ Yes    |
| Logout               | ✅ Yes    |

### Example:

```js
axios.post('/api/login', data, { withCredentials: true });
axios.get('/api/profile', { withCredentials: true });
```

---

## 7. 🚪 Logout Flow

### ➤ Backend Logout Route

```js
app.post('/api/logout', (req, res) => {
  res.clearCookie('token');
  res.json({ message: 'Logged out successfully' });
});
```

### ➤ Frontend Logout Example

```js
api.post('/logout')
  .then(() => {
    window.location.href = '/login';
  });
```

---

## 8. ✅ Best Practices Summary

| Item                | Best Practice                                   |
|---------------------|--------------------------------------------------|
| Token Storage        | Use `HttpOnly`, `Secure` cookies                |
| Token Format         | JWT with short expiration                       |
| Transmission         | Always use `withCredentials: true` in Axios     |
| CSRF Protection      | Use `sameSite=Strict` or CSRF token strategy    |
| Logout               | Clear cookie on backend                         |
| Token Access in JS   | Never! Use HttpOnly cookies to prevent XSS      |

---

## 9. ❓ FAQs

### Q1. Does `withCredentials: true` store the token?
✅ Yes, **if the backend sets a cookie**, the browser will store it automatically.

### Q2. Is the token sent with every request?
✅ Yes, **if `withCredentials: true` is used**, the browser will send the cookie with each request.

### Q3. Do I need to manually add `Authorization: Bearer <token>` headers?
❌ No, not when using cookies. The browser handles it via the `Cookie:` header.

### Q4. Can JavaScript access HttpOnly cookies?
❌ No. That’s the point — they are protected from XSS attacks.

---

## 🔚 Conclusion

Using **HttpOnly cookies** for token storage and authentication is the most **secure and scalable** way to protect your users and backend. It reduces the risk of token theft via XSS and simplifies token handling.
