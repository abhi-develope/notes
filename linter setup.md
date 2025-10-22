# 🧹 ESLint + Prettier Setup Guide for MERN Projects

This guide explains how to set up **ESLint** and **Prettier** in both the **frontend (React)** and **backend (Node.js)** of a MERN stack project for clean, consistent, and bug-free code.

---

## 🧱 Step 1: Install ESLint and Prettier

Run this command in **both** `frontend/` and `backend/` folders:


npm install --save-dev eslint prettier
🧩 Step 2: Initialize ESLint


Run the ESLint initialization command:

npx eslint --init
Recommended Answers
For React (frontend):

pgsql
✔ How would you like to use ESLint? · To check syntax, find problems, and enforce code style
✔ What type of modules does your project use? · JavaScript modules (import/export)
✔ Which framework does your project use? · React
✔ Does your project use TypeScript? · No
✔ Where does your code run? · Browser
✔ What format do you want your config file in? · JSON

For Node.js (backend):
✔ What type of modules does your project use? · CommonJS (require/exports)
✔ Does your project use TypeScript? · No
✔ Where does your code run? · Node
⚙️ Step 3: Install Additional Plugins
Frontend:
npm install --save-dev eslint-plugin-react eslint-plugin-react-hooks eslint-config-prettier eslint-plugin-prettier
Backend:
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
🧠 Step 4: Configure .eslintrc.json
Frontend (frontend/.eslintrc.json)
json
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
Backend (backend/.eslintrc.json)

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
🎨 Step 5: Add Prettier Configuration
Create a .prettierrc file in both folders with this configuration:


{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
💻 Step 6: Add Lint Commands in package.json
Add the following scripts to your frontend and backend package.json files:

"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix"
}

Now you can run:

npm run lint
npm run lint:fix
🧩 Step 7: VS Code Setup
Install these VS Code extensions:

ESLint

Prettier - Code Formatter

Then, in VS Code settings (Ctrl + ,):

Enable ✅ Format On Save

Set Default Formatter → Prettier - Code Formatter

🧠 Step 8: Test It
Try this unformatted code:
const a=5
console.log(a)
After saving, it will automatically format to:

const a = 5;
console.log(a);
If there’s a logic or style issue, ESLint will underline it and show the problem in the “Problems” tab.

🎯 Benefits
✅ Auto-formatting on save (Prettier)
✅ Error detection and rule enforcement (ESLint)
✅ Consistent coding style across teams
✅ Catches bugs early
✅ Works with CI/CD pipelines for code quality checks

📁 Folder Example
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
Author: Abhishek Prajapati
Purpose: Standard ESLint + Prettier setup for consistent, clean, and production-quality MERN codebases.

---

Would you like me to also include a **GitHub Actions CI script** that automatically runs ESLint checks on every pull request (so code quality is enforced automatically)?






