# 📒 Notes Repository

This repository contains notes on various programming concepts and tools. Below is the latest information added about **DataURI** and **Cloudinary**.

---

## 🌐 Understanding DataURI

A **DataURI** represents data as a Base64-encoded string that browsers and applications can easily process. This is particularly useful for embedding binary data (like images) directly into text-based formats, such as JSON or HTML.

### 📄 Code Example

```javascript
import DataUriParser from "datauri/parser.js";
import path from "path";

// Converts a file into DataURI format
const getDataUri = (file) => {
    const parser = new DataUriParser();
    const extName = path.extname(file.originalname).toString(); // Get file extension
    return parser.format(extName, file.buffer); // Convert to DataURI
};

export default getDataUri;

