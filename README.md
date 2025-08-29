# Containerizing a MERN Stack Application with Docker Compose

![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white) ![Express.js](https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white) ![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB) ![Node.js](https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white) ![Docker Compose](https://img.shields.io/badge/Docker%20Compose-000000?style=for-the-badge&logo=docker&logoColor=2496ED)


This repo is my journey of containerizing a **MERN stack application** and running it using **Docker Compose**. Instead of just listing commands, I’ll explain the thought process behind each step — almost like a story of how I set it up, fixed errors, and got everything working together.

---

## Understanding the MERN Stack (and Why It’s Popular)

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

## Step 0: Cloning the Project

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

## Step 1: Containerizing the Frontend

First stop: the **frontend**. Inside the `frontend/` folder, I created a `Dockerfile`. (I won’t paste the full Dockerfile here, but it installs dependencies and runs React on port `5173`).

Then I built the image:

```bash
docker build -t mern-frontend .
```

<img width="2234" height="598" alt="image" src="https://github.com/user-attachments/assets/959f2754-39fe-4eab-a149-53583f9b9495" />

Now, before containers can talk to each other, they need a **common network**. I created one called `mern`:

```bash
docker network create mern
```

<img width="2234" height="128" alt="image" src="https://github.com/user-attachments/assets/80f59e58-829c-47d7-8000-0fc5f8116dfc" />

And ran the frontend container:

```bash
docker run --name=frontend --network=mern -d -p 5173:5173 mern-frontend
```

<img width="2234" height="86" alt="image" src="https://github.com/user-attachments/assets/a4fa86d4-e5bc-4bc2-90d3-cae003e5832f" />

I checked the logs:

```bash
docker logs frontend
```

<img width="2234" height="284" alt="image" src="https://github.com/user-attachments/assets/9ce014df-294a-4b35-b2e3-86eec2ae5934" />

And opened [http://localhost:5173](http://localhost:5173) —  the frontend was running! 

<img width="2936" height="1256" alt="image" src="https://github.com/user-attachments/assets/bec14cbb-3ad1-43d6-a799-e990a670c91b" />

But… when I tried to create an employee record, nothing happened. That’s because the backend wasn’t up yet.

<img width="2936" height="1732" alt="image" src="https://github.com/user-attachments/assets/18d2dcf5-dd31-4cff-a231-280c3e04a665" />
<img width="2936" height="812" alt="image" src="https://github.com/user-attachments/assets/cb976050-25c4-4f1b-bfac-7842e6e7003a" />

---

## Step 2: Running MongoDB as a Container

Here’s an important lesson: the backend tries to connect to MongoDB as soon as it starts. So if MongoDB isn’t running yet, the backend fails.

So, I started MongoDB **before** the backend:

```bash
docker run --network=mern --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest
```

<img width="2244" height="394" alt="image" src="https://github.com/user-attachments/assets/7f3aec56-c76e-4da4-9c4c-9be5b58f8097" />

Here:

* `--network=mern` ensures MongoDB can talk to frontend/backend.
* `-v ~/opt/data:/data/db` mounts a local folder to persist database files.
* `-p 27017:27017` exposes the database on localhost.

Visiting [http://localhost:27017](http://localhost:27017) confirmed MongoDB was alive.

<img width="2244" height="182" alt="image" src="https://github.com/user-attachments/assets/258358c2-1653-4bad-b0f9-e1da45b834e9" />

---

## Step 3: Containerizing the Backend

With MongoDB running, I moved into the `backend/` folder and created another `Dockerfile`. (Again, this installed dependencies, added CORS handling, and exposed port `5050`).

I built the backend image:

```bash
docker build -t mern-backend .
```

<img width="2244" height="676" alt="image" src="https://github.com/user-attachments/assets/901400b4-a13c-4ae1-80a8-cd432ce79bbc" />

Then ran the container:

```bash
docker run --network=mern --name=backend -d -p 5050:5050 mern-backend
```

<img width="2244" height="84" alt="image" src="https://github.com/user-attachments/assets/98bdf2bf-eda0-4625-b006-58955547f2cd" />

Now, the big test: I went back to the frontend, created a new employee record, 

<img width="2924" height="1310" alt="image" src="https://github.com/user-attachments/assets/d6fadc66-a4f8-41be-bfb6-aa2bc4337e18" />

and… it worked! The frontend was talking to the backend, and the backend was saving data into MongoDB.

<img width="2924" height="1310" alt="image" src="https://github.com/user-attachments/assets/d383670f-b1cb-490c-afe6-03b617d3ba1e" />

At this point, all three tiers were alive:

* React (frontend) → `localhost:5173`
* Express/Node (backend) → `localhost:5050`
* MongoDB → `localhost:27017`

---

## Step 4: Bringing It All Together with Docker Compose

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

<img width="2234" height="224" alt="image" src="https://github.com/user-attachments/assets/47f2e9bc-6859-4c41-842c-da70528e7675" />

This:

* Created the network automatically
* Started MongoDB first, then backend, then frontend
* Mounted my database volume

A quick `docker ps` showed all three containers happily running together.

<img width="2234" height="168" alt="image" src="https://github.com/user-attachments/assets/698fc7f2-6abd-457b-8382-a7e8aaa1c094" />

---

## Testing the Final App

I went back to [http://localhost:5173](http://localhost:5173) and created a few employee records. 
Everything worked: add, update, delete. Data was persisting in MongoDB too.

<img width="2898" height="894" alt="image" src="https://github.com/user-attachments/assets/d0e5b82f-5f22-473a-9983-ca152d01a45a" />

It felt great to see the full MERN stack running in containers — and all orchestrated by one `docker compose up` command.

---

## Troubleshooting: My First Error

The first time I ran everything, I forgot to start MongoDB before the backend. Result? The backend threw errors because it couldn’t connect.

**Fix:** Start the DB container first. In Compose, this problem goes away because `depends_on` ensures the database is up before the backend tries to connect.

---

## Wrap-Up

What I achieved:

* Learned how MERN fits into a **3-tier architecture**
* Containerized frontend, backend, and database separately
* Connected them with a shared Docker network
* Persisted MongoDB data with a volume
* Brought it all together with **Docker Compose**

The biggest win? Going from multiple manual steps to a single command:

```bash
docker compose up -d
```

This journey showed me why Docker Compose is such a game-changer for multi-container applications. When apps scale, this simplicity is priceless.

---

✨ Thanks for following along! If you’d like to try it yourself, clone the repo and give it a shot. Containerizing your own MERN app is a fantastic way to learn Docker, networks, and 3-tier architecture in action.
