# 🧹 ESLint + Prettier Setup Guide for MERN Projects

This guide explains how to set up **ESLint** and **Prettier** for both **frontend (React)** and **backend (Node.js)** in a MERN stack project.  
It ensures clean, consistent, and bug-free code across your entire application.

---

## 🧱 Step 1: Install ESLint and Prettier

Run this command in **both** `frontend/` and `backend/` folders:

```bash
npm install --save-dev eslint prettier
```

---

## 🧩 Step 2: Initialize ESLint

Initialize ESLint in each folder:

```bash
npx eslint --init
```

### Recommended Answers

#### ✅ For React (Frontend)
```
✔ How would you like to use ESLint? → To check syntax, find problems, and enforce code style
✔ What type of modules does your project use? → JavaScript modules (import/export)
✔ Which framework does your project use? → React
✔ Does your project use TypeScript? → No
✔ Where does your code run? → Browser
✔ What format do you want your config file in? → JSON
```

#### ✅ For Node.js (Backend)
```
✔ What type of modules does your project use? → CommonJS (require/exports)
✔ Does your project use TypeScript? → No
✔ Where does your code run? → Node
```

This creates a `.eslintrc.json` file in each folder.

---

## ⚙️ Step 3: Install Additional ESLint Plugins

### 📦 Frontend (React)
```bash
npm install --save-dev eslint-plugin-react eslint-plugin-react-hooks eslint-config-prettier eslint-plugin-prettier
```

### 📦 Backend (Node.js)
```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

---

## 🧠 Step 4: Configure `.eslintrc.json`

### 🪄 Frontend (`frontend/.eslintrc.json`)
```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:prettier/recommended"
  ],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "rules": {
    "react/prop-types": "off",
    "no-unused-vars": "warn",
    "prettier/prettier": ["error"]
  }
}
```

### ⚙️ Backend (`backend/.eslintrc.json`)
```json
{
  "env": {
    "node": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:prettier/recommended"
  ],
  "parserOptions": {
    "ecmaVersion": "latest"
  },
  "rules": {
    "no-unused-vars": "warn",
    "prettier/prettier": ["error"]
  }
}
```

---

## 🎨 Step 5: Add Prettier Configuration

Create a `.prettierrc` file in both `frontend/` and `backend/` folders with the following content:

```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

---

## 💻 Step 6: Add Lint Commands in `package.json`

Add these scripts under `"scripts"` in both **frontend** and **backend** `package.json` files:

```json
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix"
}
```

Run these commands:

```bash
npm run lint
npm run lint:fix
```

---

## 🧩 Step 7: VS Code Setup

To enable auto-formatting and linting in VS Code:

### 🧰 Required Extensions
- **ESLint**
- **Prettier – Code Formatter**

### ⚙️ Settings
Open VS Code settings (`Ctrl + ,`) and update:
- ✅ Enable **Format On Save**
- ⚙️ Set **Default Formatter** → `Prettier - Code Formatter`

Now every time you save, your code will auto-format and fix linting errors.

---

## 🧠 Step 8: Test the Setup

Try writing unformatted code:

```js
const a=5
console.log(a)
```

After saving, it automatically becomes:

```js
const a = 5;
console.log(a);
```

If there’s a rule violation, ESLint will highlight it in the “Problems” tab.

---

## 🎯 Benefits of ESLint + Prettier

| Benefit | Description |
|----------|--------------|
| ✅ **Consistency** | Ensures a uniform code style across the team |
| ⚡ **Speed** | Auto-formats on save — no manual cleanup needed |
| 🧠 **Error Prevention** | Detects potential bugs and unused code |
| 🤝 **Team Collaboration** | Every contributor writes code in the same style |
| 🚀 **CI/CD Ready** | Can be added to GitHub Actions for automated code checks |

---

## 📁 Example Project Structure

```
mern-app/
│
├── backend/
│   ├── .eslintrc.json
│   ├── .prettierrc
│   └── package.json
│
├── frontend/
│   ├── .eslintrc.json
│   ├── .prettierrc
│   └── package.json
│
└── README.md or docs/linter-setup.md
```

---

## 🧑‍💻 Author
**Abhishek Prajapati**  
MERN Developer | Node.js | React | MongoDB | Express  

---

**✅ Final Tip:**  
You can also integrate this setup with **GitHub Actions** to automatically lint your code on each pull request — ensuring consistent quality across your repo.
