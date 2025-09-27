# Docker A–Z (Node/Express, Compose, Nginx, Redis, Prod, and Swarm)

> A single, practical walkthrough from a blank Express app to a production deployment with rolling updates via Docker Swarm. The focus stays on Docker/containerization. Non‑Docker application code is kept minimal.

---

## 1) Building a simple Express app (baseline)

We start with a tiny Express server so we have something to containerize.

**Files to create**
- **package.json** (via **npm init -y**)
- **index.js** (minimal Express “Hello” route)

Example `index.js` (kept tiny so the Docker bits shine):
```js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => res.send('<h2>hi there</h2>'));
app.listen(PORT, () => console.log(`listening on ${PORT}`));
```

**Run locally once (optional sanity check):** **node index.js** and browse **http://localhost:3000**.

---

## 2) Create a Docker image (Dockerfile)

We base our image on the official Node image and copy our code into it. We also support **dev vs prod** installs using **ARG** (you asked for this).

**Dockerfile**
```dockerfile
# 1) Choose a base image (pick a slim, LTS tag in real life)
FROM node:18-alpine

# 2) Set work directory (all subsequent paths are relative here)
WORKDIR /app

# 3) Add only package manifests first (layer cache optimization)
COPY package*.json ./

# 4) Dev/Prod install control (build-arg + env)
ARG NODE_ENV=development
ENV NODE_ENV=$NODE_ENV

# Use npm ci in CI/Prod; npm install in dev. Omit dev deps in prod.
RUN if [ "$NODE_ENV" = "production" ]; then \
      npm ci --omit=dev; \
    else \
      npm install; \
    fi

# 5) Now copy the rest of the app (only after deps are cached)
COPY . .

# 6) Document the port (for humans/tools; not a real publish)
EXPOSE 3000

# 7) Default command (dev will override with nodemon later)
CMD ["node","index.js"]
```

**Build (dev):** **docker build -t node-app-image .**  
**Build (prod):** **docker build -t node-app-image --build-arg NODE_ENV=production .**

> Why copy `package*.json` first? It lets Docker cache dependency layers so rebuilds are fast when you only change app code.

---

## 3) Docker networking & port forwarding

Containers can reach the outside world by default, but nothing can reach *in* unless you publish a port.

**Run the container (publish 3000):**  
**docker run -d --name node-app -p 3000:3000 node-app-image**

- Left side (**3000**) is the host port (`0.0.0.0:3000` = “listen on all host interfaces”).  
- Right side (**3000**) is the container port your app listens on.

You’ll see **0.0.0.0:3000 → 3000/tcp** in **docker ps** meaning: “Host port 3000 forwards to container TCP port 3000.”

---

## 4) .dockerignore (don’t copy junk into the image)

Keep images lean and avoid leaking secrets.

**.dockerignore**
```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
.env
```
Rebuild after you add this so unneeded files aren’t baked into layers.

---

## 5) Bind-mount volume (live dev without rebuilding)

Bind mounts sync a host folder into the container so edits are instant.

**Start with a bind mount (Windows CMD):**  
**docker run -d --name node-app -p 3000:3000 -v %cd%:/app node-app-image**

**Windows PowerShell:** **-v ${PWD}:/app**  
**macOS/Linux:** **-v $(pwd):/app**

Now edits on your host reflect inside **/app** immediately.

To auto-restart Node on changes, use nodemon in dev:
- **npm i -D nodemon**
- Add scripts in **package.json**:
  ```json
  { "scripts": { "start": "node index.js", "dev": "nodemon -L index.js" } }
  ```
- Override the image command at run time:  
  **docker run -d --name node-app -p 3000:3000 -v $(pwd):/app node-app-image sh -c "npm run dev"**

> On Windows with Docker, nodemon sometimes needs **-L** (legacy watch).

---

## 6) Anonymous volume to protect **/app/node_modules**

Bind mounts mirror your host directory into **/app**, which can *wipe* the container’s **/app/node_modules** if the host doesn’t have it. Protect it by layering a more specific volume path that “wins”.

**Run with bind mount + anonymous volume:**
- **docker run -d --name node-app -p 3000:3000 -v $(pwd):/app -v /app/node_modules node-app-image**

That second **-v** creates an **anonymous volume** at **/app/node_modules** (more specific path), so Docker ignores the bind mount for that folder.

---

## 7) Read‑only bind mount (safer dev)

Let the container read your source but not modify it.

**Run read-only:**  
**docker run -d --name node-app -p 3000:3000 -v $(pwd):/app:ro -v /app/node_modules node-app-image**

Now writes inside the container to **/app** fail (“read-only filesystem”).

---

## 8) Deleting volumes (and renewing anonymous ones)

- List volumes: **docker volume ls**  
- Remove a named/anon volume: **docker volume rm <VOLUME_NAME>**  
- Remove all dangling volumes: **docker volume prune**

With Compose, when you install new deps and want a *fresh* anonymous volume for `node_modules`:
- **docker compose up -d --build -V** (v1) or  
- **docker compose up -d --build --renew-anon-volumes** (v2)

---

## 9) Docker Compose (single file)

Compose lets you declare containers, networks, and volumes in YAML and bring them up together.

**docker-compose.yml (shared base)**
```yaml
services:
  node-app:
    build:
      context: .
      args:
        NODE_ENV: ${NODE_ENV:-development}
    image: ${DOCKERHUB_USER:-local}/node-app:latest
    container_name: node-app
    working_dir: /app
    volumes:
      - ./:/app:ro            # read-only bind for code
      - /app/node_modules     # anonymous volume for deps
    environment:
      - PORT=${PORT:-3000}
    ports:
      - "3000:3000"
```

**Start:** **docker compose up -d**  
**Stop:** **docker compose down**

---

## 10) Compose: dev & prod overrides

Keep common parts in **docker-compose.yml** and add environment‑specific files.

**docker-compose.dev.yml**
```yaml
services:
  node-app:
    command: npm run dev
    volumes:
      - ./:/app:ro
      - /app/node_modules
    environment:
      - NODE_ENV=development
```

**docker-compose.prod.yml**
```yaml
services:
  node-app:
    build:
      args:
        NODE_ENV: production
    environment:
      - NODE_ENV=production
    # no bind mounts in prod
    volumes: []
```

**Dev up:** **docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d**  
**Prod up:** **docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build**

> Later we’ll replace `build:` in prod with an **image:** pulled from Docker Hub.

---

## 11) MongoDB container (local dev)

Add Mongo as a separate service; don’t publish its port unless you *need* external access.

```yaml
services:
  mongo:
    image: mongo:6
    container_name: mongo
    restart: unless-stopped
    volumes:
      - mongo-data:/data/db
volumes:
  mongo-data:
```

Inside the **node-app** container, connect using Docker DNS: **mongodb://mongo:27017/dbname** (no need to publish 27017).

---

## 12) Docker networks & DNS (service discovery)

Compose puts your services on an isolated network where **service names are hostnames**. From **node-app**, you can reach **mongo**, **redis**, **nginx**, etc., by those names—no IPs required.

If you truly need host access to a DB (e.g., for a GUI), then add a published port to that service *temporarily*.

---

## 13) Express environment variables (in containers)

Prefer environment variables for config (ports, URIs, secrets). Two ways to set them when running containers:

- Per‑var: **-e PORT=4000**
- From file: **--env-file ./.env** (lines like `PORT=4000`)

With Compose, you can pass values via an **.env** file next to your compose files or `environment:` blocks in YAML.

---

## 14) `depends_on` (and why it doesn’t solve readiness)

Compose can start one service before another, but it does **not** wait for the *app* to be ready. Databases often need ~seconds to accept connections. Implement retries in your app or add a **healthcheck** and gate on it.

```yaml
services:
  node-app:
    depends_on:
      - mongo
    healthcheck:
      test: ["CMD", "node", "-e", "process.exit(0)"] # placeholder
      interval: 10s
      timeout: 2s
      retries: 3
```

> In practice, let your app retry DB connections (Mongoose already retries by default for a bit).

---

## 15) CRUD demo (just enough for Docker)

You built simple **/api/v1/posts** routes with Mongoose models. From Docker’s POV, ensure the app connects using the service DNS (**mongo**) and that the container restarts on failure (Compose `restart:` or Swarm `restart_policy` later).

---

## 16) Signup & login (bcrypt, minimal)

App bits (users collection, hashing, simple controllers) are not Docker-specific. Key Docker note: when you add packages, remember to rebuild images or renew anonymous volumes for dev containers to pick up **node_modules** changes.

- **docker compose up -d --build --renew-anon-volumes**

---

## 17) Sessions with Redis (auth)

Add Redis and wire **express-session** + **connect-redis**.

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
```

Minimal server glue (conceptual):
```js
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const { createClient } = require('redis');

const redisClient = createClient({ url: `redis://redis:6379` });
await redisClient.connect();

app.set('trust proxy', 1); // behind Nginx later
app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, secure: false, maxAge: 60_000 } // demo
}));
```

> In Postman you’ll see a `connect.sid` cookie. In Redis, the key looks like **sess:...**—different formats but linked.

---

## 18) Quick architecture review

- Only public‑facing service should expose a port (later **nginx**).  
- Internal services (**mongo**, **redis**) stay private on the Docker network.  
- Scale **node-app** by running multiple replicas; front them with Nginx.

---

## 19) Nginx container (reverse proxy & load balancer)

**nginx/default.conf**
```nginx
server {
  listen 80;

  # API only (prefix)
  location /api/ {
    proxy_pass http://node-app:3000;
    proxy_http_version 1.1;

    # preserve client details
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  # (Optional) serve static frontend here for non-/api requests
}
```

**Compose snippet**
```yaml
services:
  nginx:
    image: nginx:stable-alpine
    ports:
      - "3000:80"   # dev: host:3000 -> nginx:80
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - node-app
```

> Requests hit Nginx, which proxies to **http://node-app:3000** (Docker DNS). Nginx round‑robins across multiple **node-app** replicas.

---

## 20) CORS in Express (quick)

If your frontend is on another origin, enable CORS on the API:

- **npm i cors**  
- In app: `app.use(require('cors')())` (fine for demos; tighten origins in prod).

---

## 21) Production Ubuntu server setup

- Get a VM (DO/AWS/GCP/Azure or local VM).  
- SSH in: **ssh root@<SERVER_IP>**  
- Install Docker the quick way:  
  **curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh**
- Install Compose plugin (if missing): follow Docker docs, or **apt-get install docker-compose-plugin** on newer distros.
- Verify: **docker --version** and **docker compose version**.

---

## 22) Git setup (to fetch app code)

On the server:
- **mkdir -p /app && cd /app**
- **git clone <YOUR_REPO_URL> .**
- **ls** to confirm files.

> Do **not** store secrets in the repo. Use env vars on the server.

---

## 23) Server environment variables (persist through reboot)

Create **/root/.env** with your real secrets (example):
```
NODE_ENV=production
PORT=80
SESSION_SECRET=some-long-random-string
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=supersecret
DB_USER=appuser
DB_PASSWORD=apppass
```

Auto‑export them by sourcing in your shell profile:

- Edit **/root/.profile** (or **~/.bashrc**) and add to the end:
  ```bash
  set -o allexport
  source /root/.env
  set +o allexport
  ```
- Re-login or **source ~/.profile**.  
- Verify: **printenv | grep -E 'PORT|SESSION_SECRET'**.

In Compose, refer to them like `${SESSION_SECRET}` etc. (no secrets in YAML).

---

## 24) First deploy to the production server

From **/app** on the server:

- **docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build**

Visit **http://SERVER_IP/** or the published port you chose (Nginx -> Node).

---

## 25) Pushing app changes to prod

Typical (unsafe) path we started with:
1) Dev pushes commits to GitHub.  
2) On the server: **git pull**.  
3) Recreate containers: **docker compose -f ... up -d --build**.  
   - To update just one service: **docker compose -f ... up -d --build --no-deps node-app**.

> Building on prod consumes CPU/RAM; we’ll fix that with a Docker Hub image workflow.

---

## 26) Forcing container rebuilds (even if image/config unchanged)

- **docker compose up -d --force-recreate node-app**  
- Avoid touching dependencies with **--no-deps** if you don’t want DB/Redis restarted.

---

## 27) Dev → Prod workflow (problem statement)

Building images on the production server is risky (resource spikes, longer outages). Prefer **build on dev → push image to registry → pull on prod → recreate container**.

---

## 28) Docker Hub image workflow

On your **dev** machine:

- Tag/push once (or let Compose do it):
  - **docker login**
  - **docker image tag node-app-image YOUR_USER/node-app:latest**
  - **docker push YOUR_USER/node-app:latest**

- In **docker-compose.prod.yml**, prefer the prebuilt image:
  ```yaml
  services:
    node-app:
      image: YOUR_USER/node-app:latest
      # (keep build: only if you still want local prod builds)
  ```

- Compose can build & push for you:
  - **docker compose -f docker-compose.yml -f docker-compose.prod.yml build node-app**
  - **docker compose -f docker-compose.yml -f docker-compose.prod.yml push node-app**

On **prod**:
- **docker compose -f ... pull node-app**
- **docker compose -f ... up -d --no-deps node-app**

---

## 29) Watchtower (optional auto-puller)

Automate “pull new image & restart container when a registry update appears.”

**Run (demo settings):**  
**docker run -d --name watchtower -e WATCHTOWER_TRACE=true -e WATCHTOWER_DEBUG=true -e WATCHTOWER_POLL_INTERVAL=50 -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower node-app**

- If your images are private, **docker login** on the server first.
- Remove later: **docker rm -f watchtower**.

---

## 30) Do you *need* Watchtower? (trade‑offs)

- 👍 Great for hobby apps and quick demos.  
- 👎 For serious prod, many teams prefer manual or CI/CD‑driven deploys with checks and canaries. Auto‑restarts can surprise you.

---

## 31) Docker Swarm basics (rolling updates & scaling)

Swarm is Docker’s built‑in orchestrator. It can manage replicas, do rolling updates, and (across multiple nodes) distribute services.

**Enable Swarm on the server:**
- Check: **docker info** (look for `Swarm: inactive`)
- Init: **docker swarm init --advertise-addr <PUBLIC_IP>**

> *advertise-addr* = the address other nodes would use to reach this manager (use the server’s public IP on a single‑node demo).

Compose already works with Swarm—just add a `deploy:` section.

**docker-compose.prod.yml (add deploy)**
```yaml
services:
  node-app:
    image: YOUR_USER/node-app:latest
    deploy:
      replicas: 8
      restart_policy:
        condition: any
      update_config:
        parallelism: 2
        delay: 15s
        failure_action: rollback
        order: start-first
```

**Deploy the stack:**
- **docker stack deploy -c docker-compose.yml -c docker-compose.prod.yml myapp**

Useful commands:
- **docker stack ls** (stacks)  
- **docker stack services myapp** (services)  
- **docker stack ps myapp** (tasks)  
- **docker service ls** (all services)  
- **docker node ls** (swarm nodes)

> Nginx still fronts traffic; Swarm updates your **node-app** replicas behind it.

---

## 32) Updating the app in Swarm (rolling updates)

1) On **dev**, build & push a new image to Docker Hub (section 28).  
2) On **prod**, redeploy the stack:  
   **docker stack deploy -c docker-compose.yml -c docker-compose.prod.yml myapp**

Swarm pulls the newer image and performs a rolling update obeying `update_config` (e.g., **2 at a time, 15s delay, rollback if failures**). During the update, Nginx keeps routing to the still‑healthy replicas.

---

## Appendix: Common commands (quick copy)

- **docker build -t node-app-image .**  
- **docker run -d --name node-app -p 3000:3000 node-app-image**  
- **docker ps** / **docker logs -f node-app**  
- **docker rm -f node-app**  
- **docker volume ls** / **docker volume prune**  
- **docker compose up -d** / **docker compose down**  
- **docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build**  
- **docker compose up -d --build --renew-anon-volumes**  
- **docker compose up -d --force-recreate --no-deps node-app**  
- **docker login && docker push YOUR_USER/node-app:latest**  
- **docker swarm init --advertise-addr <PUBLIC_IP>**  
- **docker stack deploy -c docker-compose.yml -c docker-compose.prod.yml myapp**
