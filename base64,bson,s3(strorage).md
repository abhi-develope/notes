There are actually three different concepts here that often get mixed together:

Base64 → binary data represented as text
Binary/BSON → actual bytes stored inside MongoDB
Cloudinary/S3 → separate systems designed to store files/objects
1. First: what is a signature really?

When the patient draws:

       John
   ───────────

your browser's canvas doesn't fundamentally contain a PNG.

It contains things like pixels or drawing paths in memory.

Eventually you need to turn that into some representation that can be saved.

For example:

Canvas
   ↓
PNG
   ↓
Binary bytes

A PNG file is ultimately just a sequence of bytes:

89 50 4E 47 0D 0A 1A 0A ...

Computers don't fundamentally store "images", "PDFs", "signatures", etc.

They store bytes.

That's the key concept behind everything below.

2. What is Base64?

Base64 is basically a way of saying:

"I have binary bytes, but I want to represent those bytes using normal text characters."

Suppose your file contains bytes:

72 101 108 108 111

Those bytes represent:

Hello

Base64 can represent the same bytes as:

SGVsbG8=

So:

Binary
   ↓
Base64 encoding
   ↓
Text

And you can reverse it:

Base64 text
   ↓
Base64 decoding
   ↓
Original binary
Why was Base64 created?

Because historically/technically, many systems were designed to safely transport text, not arbitrary binary data.

So Base64 gives you a text representation of binary.

3. Signature example with Base64

Your frontend might do:

const dataUrl = canvas.toDataURL("image/png");

You may get something like:

data:image/png;base64,
iVBORw0KGgoAAAANSUhEUgAA...

That whole thing is essentially:

PNG binary
     ↓
Base64 encoding
     ↓
text string

So you could send:

{
  "signature": "data:image/png;base64,iVBORw0KGgoAAA..."
}

to your backend.

Then MongoDB might store:

{
  signature: "data:image/png;base64,iVBORw0KGgoAAA..."
}
The problem?

You're storing binary data as text.

And Base64 makes the data approximately 33% larger.

For example:

Original PNG       30 KB
Base64 version     ~40 KB

So Base64 isn't a storage format itself.

It's an encoding.

That's an important distinction.

4. What is BSON?

Now let's talk about MongoDB.

MongoDB doesn't internally store documents as JSON.

You write:

{
  name: "John",
  age: 35
}

but MongoDB stores documents using a format called:

BSON = Binary JSON

It's a binary representation of JSON-like documents.

Conceptually:

Your JavaScript object
        ↓
MongoDB driver
        ↓
      BSON
        ↓
 MongoDB storage

BSON supports different data types:

String
Number
Boolean
Date
ObjectId
Array
Object
Binary
...

And importantly:

Binary

is a native BSON type.

5. MongoDB Binary storage

Now we're getting to the part you're interested in.

Instead of:

{
  signature: "iVBORw0KGgoAAA..."
}

you can give MongoDB the actual bytes.

For example in Node.js:

const buffer = Buffer.from(...);

and:

{
  signature: buffer
}

Mongoose can define:

signature: {
  type: Buffer
}

Behind the scenes, MongoDB stores that as BSON Binary.

Conceptually:

PNG file
   │
   │ actual bytes
   ↓
Node.js Buffer
   │
   ↓
MongoDB driver
   │
   ↓
BSON Binary
   │
   ↓
MongoDB

There is no Base64 conversion necessary.

6. So what's actually stored?

Let's simplify dramatically.

Suppose your signature file contains:

8 bytes

Something like:

10101010
01101001
11001010
...

With Base64:

Binary bytes
     ↓
encode
     ↓
"qmnK..."

MongoDB stores:

String
"qmnK..."

With BSON Binary:

Binary bytes
     ↓
MongoDB BSON
     ↓
[actual bytes]

MongoDB stores the binary data itself.

That's why Binary is more efficient than Base64 for this purpose.

7. Then what is "MongoDB binary storage"?

This phrase can sound more complicated than it is.

It simply means:

MongoDB's BSON document contains a field whose type is Binary rather than String.

For example:

{
  patientId: "...",

  signature: <BINARY DATA>,

  mimeType: "image/png"
}

You aren't storing:

"some image"

You're storing the actual bytes.

8. How does Node.js handle this?

This is where Buffer comes in.

Node.js has a native binary data structure:

Buffer

For example:

const buffer = Buffer.from("Hello");

You can inspect it:

console.log(buffer);

Conceptually:

Buffer
┌────┬────┬────┬────┬────┐
│ 48 │ 65 │ 6C │ 6C │ 6F │
└────┴────┴────┴────┴────┘

Those are hexadecimal representations of bytes.

So:

Buffer = bytes in Node.js memory

And MongoDB's BSON Binary is:

BSON Binary = bytes stored in MongoDB

The MongoDB driver handles the conversion between your Node.js representation and BSON.

9. What happens when you READ it?

Suppose MongoDB contains:

{
  signature: <binary>
}

Your Node.js application queries:

const signature = await PatientSignature.findOne({
  patientId
});

The driver/Mongoose gives you the binary as a Node.js Buffer.

Then you can send it to the browser:

res.set("Content-Type", "image/png");

res.send(signature.data);

The browser receives:

PNG bytes

and renders the image.

So:

MongoDB
   ↓
BSON Binary
   ↓
MongoDB Driver
   ↓
Node.js Buffer
   ↓
HTTP Response
   ↓
Browser
   ↓
Image

Again, no Base64 is required.

10. Now compare that with Cloudinary

This is fundamentally different.

With MongoDB Binary:

                Your Server
                    │
             ┌──────┴──────┐
             │             │
          MongoDB        App DB
          Binary

Your database contains the actual file bytes.

With Cloudinary:

Frontend
   ↓
Backend
   ↓
Cloudinary
   ↓
Image/File Storage
   ↓
URL

MongoDB only stores:

{
  signatureUrl: "https://..."
}

The actual image lives somewhere else.

11. S3 works similarly

AWS S3 is object storage.

You upload:

signature.png

to S3.

S3 stores the actual bytes.

Your MongoDB stores something like:

{
  patientId: "...",

  signature: {
    storageKey: "patients/123/signature.png",
    mimeType: "image/png"
  }
}

Then your application can retrieve the file from S3.

So:

MongoDB
   │
   └── "patients/123/signature.png"
                  │
                  ↓
                 S3
                  │
                  └── actual bytes
12. The biggest conceptual difference

Think about MongoDB vs S3 like this:

MongoDB

MongoDB is primarily:

a database that can contain binary fields

S3

S3 is:

a storage system specifically designed for objects/files

That's why people usually don't treat them as interchangeable.

13. Imagine you have 100 GB of files

Suppose your HMS eventually has:

10,000 patients

Each patient:
 ├── Signature
 ├── Profile image
 ├── Lab report PDFs
 ├── X-rays
 ├── Insurance documents
 └── Discharge documents

Now you might have:

100 GB
500 GB
1 TB

of files.

Putting all of that into MongoDB is possible in certain architectures, but it's generally not what you'd choose when object storage is available.

Instead:

                  HMS
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
    MongoDB                  S3
    metadata                 files
        │                     │
        │                     ├── PDFs
        │                     ├── images
        │                     ├── scans
        │                     └── signatures
        │
        └── references

That's a very common architecture.

14. Why S3/Cloudinary are good at files

Object storage systems are designed around things like:

Upload
Download
Delete
Version
Access control
Encryption
CDN
Large files
High throughput
Durability

And they can handle enormous amounts of data.

MongoDB is optimized around:

Query documents
Update records
Indexes
Relationships/references
Transactions
Aggregation
Application data

So you generally don't want to turn your database into a giant file server.

15. But your signature is a special case

This is where your original question becomes important.

Suppose your HMS has:

Patient signature = 15 KB

and:

Patient data = 10 KB

Putting the 15 KB signature in MongoDB is not crazy at all.

You could have:

PatientSignature {
    patientId,
    data: Buffer,
    mimeType: "image/png"
}

That's perfectly reasonable.

But storing:

500 MB X-ray

inside MongoDB is a very different discussion.

I'd rather have:

MongoDB
    ↓
xrayKey: "patients/123/xray/abc.jpg"

S3
    ↓
actual 500 MB file
16. One more important thing: GridFS

You may eventually hear another MongoDB term:

GridFS

MongoDB has a mechanism specifically for storing files larger than the normal BSON document limit.

Conceptually:

Large File
   ↓
GridFS
   ↓
MongoDB
   ├── file metadata
   └── file chunks

GridFS breaks a large file into chunks and stores those chunks in MongoDB.

But you don't need GridFS for a small patient signature.

A normal BSON Binary field is enough.

17. So your three options look like this
Option 1 — Base64
Canvas
 ↓
PNG bytes
 ↓
Base64
 ↓
String
 ↓
MongoDB String

Example:

{
  signature: "iVBORw0KGgoAAA..."
}

Easy, but inefficient.

Option 2 — MongoDB Binary
Canvas
 ↓
PNG bytes
 ↓
Node Buffer
 ↓
BSON Binary
 ↓
MongoDB

Example:

{
  signature: Buffer(...)
}

Efficient and simple for small files.

Option 3 — S3/Cloudinary
Canvas
 ↓
PNG bytes
 ↓
S3 / Cloudinary
 ↓
actual file stored there

MongoDB
 ↓
URL / storage key

Example:

{
  signature: {
    storageKey: "patients/123/signature.png",
    mimeType: "image/png"
  }
}

Best when you're dealing with lots of files or large files.

18. And where does PNG/SVG fit into all this?

This is another separate concept.

PNG/SVG describes the file format.

Base64 describes an encoding.

BSON Binary describes how MongoDB stores bytes.

S3/Cloudinary describes where the file is stored.

These are four different layers:

             WHAT IS THE DATA?
                   │
              PNG / SVG
                   │
                   ↓
             HOW REPRESENTED
                   │
          Binary / Base64
                   │
                   ↓
             WHERE STORED?
                   │
          ┌────────┴────────┐
          ↓                 ↓
       MongoDB            S3
       Binary

That's the mental model I'd keep in your head.

Finally, for your HMS

I'd personally think about your signature pipeline as:

                 Patient signs
                       │
                       ↓
                 HTML Canvas
                       │
                       ↓
                 PNG / SVG
                       │
                ┌──────┴──────┐
                │             │
           Small file      Large file
                │             │
                ↓             ↓
        MongoDB Binary       S3
                │             │
                ↓             ↓
          Signature ID      Storage Key

And don't confuse Base64 with binary.

The biggest takeaway is:

Base64 is a way to represent bytes as text. BSON Binary is MongoDB's native way of storing bytes. S3/Cloudinary are separate file/object-storage systems where those bytes can live.
