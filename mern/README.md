# 🚀 Containerizing a MERN Stack Application with Docker Compose

![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white)  
![Express.js](https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white)  
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)  
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white)  
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)  
![Docker Compose](https://img.shields.io/badge/Docker%20Compose-000000?style=for-the-badge&logo=docker&logoColor=2496ED)  

This repo is my journey of containerizing a **MERN stack application** and running it using **Docker Compose**. Instead of just listing commands, I’ll explain the thought process behind each step — almost like a story of how I set it up, fixed errors, and got everything working together.

---

## 📖 Understanding the MERN Stack (and Why It’s Popular)

Before diving into Docker, let’s set the stage.

A **three-tier architecture** is a classic way to design end-to-end applications. It has three main layers:

1. **Presentation Layer** – The user interface (frontend).
2. **Business Logic Layer** – The backend or server logic.
3. **Data Layer** – The database.

The **MERN stack** is a modern way to implement this architecture:

* **M** – MongoDB → Database layer
* **E** – Express.js → Backend framework
* **R** – React.js → Frontend framework
* **N** – Node.js → Runtime for backend

It’s popular because:

* It’s **open source** (free to use).
* It’s all based on **JavaScript**, so one language ties frontend + backend together.
* It’s **simple and beginner-friendly** compared to many other stacks.

In my app, I had two main folders:

* `frontend/` – React code (UI)
* `backend/` – Express.js + Node.js server
* MongoDB wasn’t in the repo — we’d run it as a container. The backend connects to it using `connection.js`.

Here’s the connection string from my backend’s `connection.js`:

```js
const URI = "mongodb://mongodb:27017";
const client = new MongoClient(URI, {
  serverApi: { version: ServerApiVersion.v1, strict: true, deprecationErrors: true },
});
```

Notice the container name is **`mongodb`** — that’s what makes networking between containers work later.

---

## 🛠 Step 0: Cloning the Project

I started by cloning my MERN repo:

```bash
git clone https://github.com/arnab-logs/containerize-mern-app-docker-compose.git
```

Then navigated into it:

```bash
cd containerize-mern-app-docker-compose/mern
```

And opened it up in **VS Code** for easier editing.

---

## 🎨 Step 1: Containerizing the Frontend

First stop: the **frontend**. Inside the `frontend/` folder, I created a `Dockerfile`. (I won’t paste the full Dockerfile here, but it installs dependencies and runs React on port `5173`).

Then I built the image:

```bash
docker build -t mern-frontend .
```

Now, before containers can talk to each other, they need a **common network**. I created one called `mern`:

```bash
docker network create mern
```

And ran the frontend container:

```bash
docker run --name=frontend --network=mern -d -p 5173:5173 mern-frontend
```

I checked the logs:

```bash
docker logs frontend
```

And opened [http://localhost:5173](http://localhost:5173) — 🎉 the frontend was running! But… when I tried to create an employee record, nothing happened. That’s because the backend wasn’t up yet.

---

## 🗄 Step 2: Running MongoDB as a Container

Here’s an important lesson: the backend tries to connect to MongoDB as soon as it starts. So if MongoDB isn’t running yet, the backend fails.

So, I started MongoDB **before** the backend:

```bash
docker run --network=mern --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest
```

Here:

* `--network=mern` ensures MongoDB can talk to frontend/backend.
* `-v ~/opt/data:/data/db` mounts a local folder to persist database files.
* `-p 27017:27017` exposes the database on localhost.

Visiting [http://localhost:27017](http://localhost:27017) confirmed MongoDB was alive.

---

## ⚙️ Step 3: Containerizing the Backend

With MongoDB running, I moved into the `backend/` folder and created another `Dockerfile`. (Again, this installed dependencies, added CORS handling, and exposed port `5050`).

I built the backend image:

```bash
docker build -t mern-backend .
```

Then ran the container:

```bash
docker run --network=mern --name=backend -d -p 5050:5050 mern-backend
```

Now, the big test: I went back to the frontend, created a new employee record, and… ✅ it worked! The frontend was talking to the backend, and the backend was saving data into MongoDB.

At this point, all three tiers were alive:

* React (frontend) → `localhost:5173`
* Express/Node (backend) → `localhost:5050`
* MongoDB → `localhost:27017`

---

## 🧩 Step 4: Bringing It All Together with Docker Compose

Manually building networks, running containers, and ordering them correctly works… but it’s messy. That’s where **Docker Compose** shines.

At the project root, I created a `docker-compose.yml`. Inside it, I defined:

* `frontend`
* `backend` (with `depends_on: mongodb` so it waits for the DB)
* `mongodb`
* a shared `network`
* a `volume` for MongoDB’s data

Now, instead of running multiple commands, I just needed:

```bash
docker compose up -d
```

This:

* Created the network automatically
* Started MongoDB first, then backend, then frontend
* Mounted my database volume

A quick `docker ps` showed all three containers happily running together.

---

## ✅ Testing the Final App

I went back to [http://localhost:5173](http://localhost:5173) and created a few employee records. Everything worked: add, update, delete. Data was persisting in MongoDB too.

It felt great to see the full MERN stack running in containers — and all orchestrated by one `docker compose up` command.

---

## 🐞 Troubleshooting: My First Error

The first time I ran everything, I forgot to start MongoDB before the backend. Result? The backend threw errors because it couldn’t connect.

**Fix:** Start the DB container first. In Compose, this problem goes away because `depends_on` ensures the database is up before the backend tries to connect.

---

## 🎯 Wrap-Up

What we achieved:

* Learned how MERN fits into a **3-tier architecture**
* Containerized frontend, backend, and database separately
* Connected them with a shared Docker network
* Persisted MongoDB data with a volume
* Brought it all together with **Docker Compose**

👉 The biggest win? Going from multiple manual steps to a single command:

```bash
docker compose up -d
```

This journey showed me why Docker Compose is such a game-changer for multi-container applications. When apps scale, this simplicity is priceless.

---

✨ Thanks for following along! If you’d like to try it yourself, clone the repo and give it a shot. Containerizing your own MERN app is a fantastic way to learn Docker, networks, and 3-tier architecture in action.
