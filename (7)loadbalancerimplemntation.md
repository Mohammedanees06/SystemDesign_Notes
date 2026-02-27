# Load Balancer Implementation

*(Load Balancer, Application Servers, Cache, Database Replication & Shared Storage)*

---

#  Goal

We want this:
```
Users
   ↓
Load Balancer
   ↓
Multiple MERN Backend Servers
   ↓
MongoDB
```

So traffic is distributed automatically.

---

# STEP 0 — Build Normal MERN App (Nothing Special)

You first build your app normally.

Example backend:

```js
// server.js
import express from "express";

const app = express();

app.get("/", (req, res) => {
  res.send("Server running");
});

app.listen(process.env.PORT, () =>
  console.log("Server started")
);
```

At this stage:

```
User → One Server
```

No scaling yet.

---

#  STEP 1 — Make Backend Stateless (VERY IMPORTANT)

Load balancer may send requests to **any server**.

So you must NOT store data inside server memory.

❌ Wrong:

```js
let users = [];
```

Because Server A ≠ Server B.

---

Correct:

Store shared data in:

* MongoDB
* Redis
* HTTP-only cookies (JWT)

Now any server can handle requests.

---

# STEP 2 — Run Multiple Backend Instances

Run SAME app multiple times.

Example:

```bash
PORT=3000 node server.js
PORT=3001 node server.js
PORT=3002 node server.js
```

Now you have:

```
Server 1 → :3000
Server 2 → :3001
Server 3 → :3002
```
Still no load balancing yet.
---

# STEP 3 — Install Load Balancer (Nginx)

Nginx (Load Balancer)

Install nginx:

```bash
sudo apt update
sudo apt install nginx
```

Nginx will become your traffic manager.

We run multiple servers to handle more requests, and we use Nginx to distribute incoming traffic among those servers automatically.

---

# STEP 4 — Configure Load Balancer

Open config:

```bash
sudo nano /etc/nginx/nginx.conf
```

Add:

```nginx
http {
    upstream backend_servers {
        server localhost:3000;
        server localhost:3001;
        server localhost:3002;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_servers;
        }
    }
}
```
---

# STEP 5 — Restart Nginx

```bash
sudo systemctl restart nginx
```
---

# 🎉 DONE — Load Balancer Working

Now flow becomes:

```
User Request
      ↓
Nginx (Load Balancer)
      ↓
Server 1 OR Server 2 OR Server 3
```

Requests are automatically distributed.

---

# What Happens Internally

User refreshes page multiple times:

```
Req1 → Server 1
Req2 → Server 2
Req3 → Server 3
Req4 → Server 1
```

(Default Round Robin)

---

#  STEP 6 — (Optional) Use PM2 Instead of Manual Servers

Instead of running manually:

```bash
pm2 start server.js -i max
```

PM2 creates multiple instances automatically.

---

# What YOU Actually Implemented

You:

✔ Built normal MERN backend
✔ Made it stateless
✔ Ran multiple instances
✔ Configured Nginx

You DID NOT change API code.

---

#  Real Production Architecture

```
Internet
   ↓
Nginx Load Balancer
   ↓
Node App (Instance 1)
Node App (Instance 2)
Node App (Instance 3)
   ↓
MongoDB
```

---

# Interview Answer

> To implement load balancing in a MERN application, multiple backend instances are deployed, the application is made stateless, and a load balancer such as Nginx is configured to distribute incoming requests across the instances.

---

