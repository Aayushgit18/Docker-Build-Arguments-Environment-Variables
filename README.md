# Docker-Build-Arguments-Environment-Variables

Docker supports **two types of variables**:

| Type    | When it's used    | Set via         | Accessible in    |
| ------- | ----------------- | --------------- | ---------------- |
| **ARG** | During build time | `--build-arg`   | Dockerfile       |
| **ENV** | During runtime    | `--env` or `-e` | Dockerfile & App |

---

## 1️⃣ ENV (Environment Variables)

### 🔹 Syntax (in Dockerfile)

```dockerfile
ENV <KEY> <value>
```

> Declares an environment variable with a default value.

---

### 🔹 Syntax (when running container)

```bash
docker run --env <KEY>=<value> <image_name>
# OR (shorthand)
docker run -e <KEY>=<value> <image_name>
```

You can also load environment variables from a file:

```bash
docker run --env-file .env <image_name>
```

---

### 🔹 Example

#### ✅ Dockerfile

```dockerfile
ENV PORT 80

EXPOSE $PORT

CMD ["node", "server.js"]
```

#### ✅ Node.js App (e.g. `server.js`)

```js
const http = require('http');
const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.end('Server is running');
});

server.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

#### ✅ Run with different PORT at runtime

```bash
docker build -t myapp-env .
docker run -p 8000:8000 -e PORT=8000 myapp-env
```

OR use `.env` file:

```env
# .env
PORT=8000
```

```bash
docker run -p 8000:8000 --env-file .env myapp-env
```

---

### 🔐 Environment Variables & Security

* ❌ **DON'T** hard-code secrets in Dockerfile using `ENV`.
* ✅ **DO** use `.env` file for secrets and pass it via `--env-file` at runtime.
* ⚠️ Environment variables declared in Dockerfile are **stored in the image** and can be viewed via:

```bash
docker history <image_name>
```

> Never store credentials, tokens, or private keys this way!

---

## 2️⃣ ARG (Build Arguments)

### 🔹 Syntax (in Dockerfile)

```dockerfile
ARG <KEY>
```

You can also assign default values:

```dockerfile
ARG BASE_VERSION=14
```

Use the argument in Dockerfile:

```dockerfile
FROM node:${BASE_VERSION}
```

---

### 🔹 Syntax (when building image)

```bash
docker build --build-arg <KEY>=<value> -t <image_name> .
```

---

### 🔹 Example

#### ✅ Dockerfile with ARG

```dockerfile
ARG BASE_VERSION=14
FROM node:${BASE_VERSION}

WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

#### ✅ Build image with custom base version

```bash
docker build --build-arg BASE_VERSION=20 -t myapp-arg .
```

---

## 🆚 ARG vs ENV — Quick Comparison

| Feature                  | `ARG`                          | `ENV`                               |
| ------------------------ | ------------------------------ | ----------------------------------- |
| Available in             | Build phase (Dockerfile only)  | Runtime (Dockerfile & app)          |
| Can be used in app code  | ❌ No                           | ✅ Yes                               |
| Default values allowed   | ✅ Yes                          | ✅ Yes                               |
| Passed via               | `--build-arg` during `build`   | `--env` / `--env-file` during `run` |
| Visible in image history | ❌ Only if used in later layers | ✅ Yes if declared in Dockerfile     |

---

## 🧪 Practical Testing

```bash
# Build with ARG
docker build --build-arg BASE_VERSION=20 -t myapp-arg .

# Run with ENV
docker run -e PORT=8000 -p 8000:8000 myapp-arg

# OR use env file
docker run --env-file .env -p 8000:8000 myapp-arg
```

---

## 📁 .env File Example

```env
PORT=8000
API_KEY=your_api_key
```

> 🔐 **Note:** Add `.env` to your `.gitignore` to prevent committing sensitive values.

---

## ✅ Final Tips

* Use **ENV** for values used by your app at **runtime** (like `PORT`, `NODE_ENV`, `API_KEY`).
* Use **ARG** for **build-time configs** (like base image version, labels).
* Avoid putting secrets in Dockerfile via `ENV` or `ARG`.
