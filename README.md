# MERN Multi-Container Application (Backend + Frontend + MongoDB)

## 1️⃣ Project Structure
```
mern-multi-container/
├── backend/
│ ├── Dockerfile
│ ├── package.json
│ └── server.js
├── frontend/
│ ├── Dockerfile
│ ├── package.json
│ ├── public/
│ │ └── index.html
│ └── src/
│ ├── App.js
│ └── index.js
├── docker-compose.yml
└── README.md
```

---

## 2️⃣ Backend Code

**backend/package.json**
```json
{
  "name": "mern-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^5.1.0",
    "cors": "^2.8.5",
    "mongoose": "^7.4.0",
    "dotenv": "^16.3.1"
  }
}
```

**backend/server.js**
```js
const express = require("express");
const cors = require("cors");
const mongoose = require("mongoose");
require("dotenv").config();

const app = express();
app.use(cors());
app.use(express.json());

const MONGO_URI = process.env.MONGO_URI || "mongodb://mongo:27017/mern";
mongoose.connect(MONGO_URI)
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.error("MongoDB connection error:", err));

app.get("/", (req, res) => {
  res.send("Hello from Backend 👋");
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

## 3️⃣ Frontend Code

**frontend/package.json**
```json
{
  "name": "mern-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^19.1.1",
    "react-dom": "^19.1.1",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  }
}
```

**frontend/public/index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>MERN Multi-Container App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>

```

**frontend/src/index.js**
```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);

```

**frontend/src/App.js**
```js
import React from "react";

function App() {
  return (
    <div>
      <h1>Hello from Frontend 👋</h1>
    </div>
  );
}

export default App;
```

---

## 4️⃣ DevOps Deployment Steps

**4.1 First Version (Without Domain)**
- cd /home/ubuntu

- git clone <YOUR_REPO_URL> mern-multi-container

- cd mern-multi-container

- Creating Dockerfiles:

**backend/Dockerfile**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

**frontend/Dockerfile**
```dockerfile
# Stage 1: Build React app
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
RUN npm install ajv@^6 ajv-keywords@^3 --legacy-peer-deps
COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- Creating docker-compose.yml File:

**docker-compose.yml**
```yml
version: "3.9"

services:
  backend:
    build: ./backend
    container_name: mern-backend
    ports:
      - "5000:5000"
    environment:
      - MONGO_URI=mongodb://mongo:27017/mern
    depends_on:
      - mongo

  frontend:
    build: ./frontend
    container_name: mern-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

  mongo:
    image: mongo:6
    container_name: mern-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

- docker-compose up -d --build

- docker ps

Access:
- Backend: http://<EC2_PUBLIC_IP>:5000

- Frontend: http://<EC2_PUBLIC_IP>:3000

Check logs:
- docker-compose logs -f backend

- docker-compose logs -f frontend

**4.2 Second Version (With Domain + SSL)**
- Stop containers first: docker-compose down

- Nginx config: sudo mkdir -p /etc/nginx/conf.d
		sudo nano /etc/nginx/conf.d/mern-app.conf

- mern-app.conf: 
```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location /api/ {
        proxy_pass http://localhost:5000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Install SSL via Certbot: 
	sudo apt update
	sudo apt install certbot python3-certbot-nginx -y
	sudo certbot --nginx -d example.com -d www.example.com

- Start Docker containers: docker-compose up -d --build

Access:
- Backend: https://example.com/api
- Frontend: https://example.com

---













