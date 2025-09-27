# Docker A → Z (up to Swarm)

These are full lecture-style explanations, written in plain English flow instead of just notes. Commands, filenames, and paths are in **bold** for copy-paste.

---

## 1. Tiny Node/Express app (context)

We start with a barebones Express server so we have something to containerize. Locally, create **package.json** with **npm init**, install Express with **npm install express**, write **index.js**, and run **node index.js** just once to confirm it says “hi there” on **http://localhost:3000**. The exact code isn’t the point; the Docker workflow is.

---

## 2. Build an image with a Dockerfile

We create **Dockerfile**. It uses the official Node image as a base, sets a working directory, installs dependencies from **package.json**, copies the app, and defines how the container starts.

Docker builds images in layers. Splitting “copy **package.json** + **npm install**” from “copy the rest of the source” makes rebuilds faster because dependency layers get cached.

Build with **docker build -t node-app-image .**  
List images with **docker image ls**.

**Extra note:** each Dockerfile line becomes a cached layer. If **package.json** doesn’t change, Docker reuses the expensive **npm install** layer.

---

## 3. Run a container and map a port

Start a container from the image: **docker run -d --name node-app -p 3000:3000 node-app-image**.

That **-p HOST:CONTAINER** opens a hole on the host and forwards to the container. In **docker ps** you’ll see **0.0.0.0:3000 → 3000/tcp**. Read it like this: “accept any host interface (**0.0.0.0**) on port **3000**, forward to container port **3000** (TCP).”

**Note:** containers can reach out to the world by default, but the world can’t reach them unless you publish a port with **-p**. The **EXPOSE** line in the Dockerfile is documentation, not a firewall rule.

---

## 4. Peek inside the container

Shell in with **docker exec -it node-app bash**, list files with **ls**, and confirm the app lives in **/app** (that’s the WORKDIR). Exit with **exit**.

---

## 5. Ignore files you don’t want in the image

Add **.dockerignore** to speed builds and avoid leaking junk:

- **node_modules/**
- **Dockerfile**
- **.dockerignore**
- **.git**
- **.gitignore**
- any **.env** or secret files (never bake secrets into images)

Rebuild: **docker build -t node-app-image .**

---

## 6. Changing code without rebuilding (bind mount)

Rebuilding the image on every save is slow. Bind-mount the source into the container so edits appear instantly:

**docker run -d --name node-app -p 3000:3000 -v "$(pwd)":/app node-app-image**

On Windows Command Prompt use **%cd%** instead of **$(pwd)**. On PowerShell use **${pwd}**.

Now the container reads host files live.

**Why edits didn’t show at first:** Node needs a restart. Add **nodemon** as a dev dependency, set **npm run dev** to run it, and change the container’s start command. Rebuild the image once; then bind mount + nodemon gives live reloads.

---

## 7. Protect **/app/node_modules** from being overwritten

When **node_modules** was deleted on the host, the bind mount mirrored that deletion into the container and **nodemon** vanished. Fix it by layering a more specific anonymous volume:

- Bind mount: **-v "$(pwd)":/app** (syncs code)
- Anonymous volume to protect deps: **-v /app/node_modules**

Because **/app/node_modules** is more specific, Docker preserves that folder inside the container even if absent on the host.

---

## 8. Make the source mount read-only

Prevent the container from modifying code. Add **:ro** to the bind mount:

**-v "$(pwd)":/app:ro**

Now attempts to write in **/app** fail, but **/app/node_modules** still works.

---

## 9. Environment variables (defaults vs overrides)

Set a sensible default port in the image and allow overrides at runtime.

- Dockerfile default: **ENV PORT=3000**  
- App reads **process.env.PORT** (fallback to 3000 in code if needed)
- Run with override: **-e PORT=4000**
- Remember to map host traffic correctly: **-p 3000:4000** means “browser hits host **3000**, app listens on **4000** inside.”

For many vars, use a file (e.g., **.env**) and run with **--env-file .env**. Inside, **printenv** shows final values.

**Rule:** Dockerfile **ENV** sets defaults; runtime **-e/--env-file** wins.

---

## 10. Docker Compose for multi-container dev

Typing **docker run** lines for Node + DB + Redis is tedious. Compose describes services once and starts them together.

- Start: **docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d**
- Stop: **docker compose down**
- Rebuild when deps change: **docker compose up -d --build**
- Avoid stale anonymous volumes: **docker compose up -d --build -V**

**depends_on** expresses startup order in dev, but not health-checks.

---

## 11. Volumes and data persistence

Databases keep data in named volumes so containers can be recreated without losing state. For app live-reload, still use a bind mount. For **node_modules**, use the anonymous volume trick or a named volume.

---

## 12. Networking and service discovery

Compose creates a project network. Containers talk by service name: app → **mongodb:27017**, not an IP. Publish only what the outside needs (like Nginx). Databases stay internal.

---

## 13. Nginx as load balancer / reverse proxy

Run Nginx to expose one port and proxy to Node instances. Requests to **/api/** go to **node-app:3000**.

Mount **nginx/default.conf** into **/etc/nginx/conf.d/default.conf:ro**. In dev publish **-p 3000:80** on Nginx and stop publishing Node ports.

**Why:** single entrypoint, and Nginx round-robins across replicas.

---

## 14. CORS in the API

When frontend and backend are on different origins, the browser blocks requests unless CORS middleware is enabled. Install, add **app.use(cors())**, and keep minimal in dev. In prod, scope origins/methods tightly.

---

## 15. Production server basics

Provision a small Ubuntu VM, SSH in (**ssh root@IP**), install Docker (**sh get-docker.sh**), install **docker compose**, clone repo, and run with prod files.

Environment variables live on host (not git). Store in **/root/.env** (or safe path). Compose references them like **${MYSQL_PASSWORD}**.

---

## 16. Pushing code changes (compose workflow)

Dev pushes code to GitHub. On server **git pull**, then:

- Build on server (not ideal): **docker compose up -d --build**
- Faster, service-only: **docker compose up -d --build --no-deps node-app**
- Force recreate without image change: **docker compose up -d --force-recreate --no-deps node-app**

**Why not build on prod:** it burns CPU/RAM needed for traffic.

---

## 17. Better workflow: build & push images from dev

Login to Docker Hub (**docker login**), tag, and push:

- Build from compose: **docker compose -f docker-compose.yml -f docker-compose.prod.yml build node-app**
- Push: **docker compose -f ... push node-app**

On server:

- Pull: **docker compose -f ... pull node-app**
- Recreate: **docker compose -f ... up -d --no-deps node-app**

Builds happen off-server; prod just pulls.

---

## 18. Swarm (rolling updates)

Compose is great for dev, but can’t orchestrate rolling updates. Swarm is built into Docker.

Enable: **docker swarm init --advertise-addr <PUBLIC_IP>**

Swarm runs “services.” Reuse Compose files with a **deploy** section:

- **replicas:** how many
- **restart_policy:** on failure
- **update_config:** e.g., **parallelism: 2**, **delay: 15s**

Deploy: **docker stack deploy -c docker-compose.yml -c docker-compose.prod.yml myapp**

Observe:

- **docker stack ls**
- **docker stack services myapp**
- **docker stack ps myapp**
- **docker node ls**

Rolling update: build/push new image, then redeploy. Swarm replaces containers gradually with minimal downtime.

---

## 19. Security & housekeeping

- Don’t publish DB ports to internet. Only app talks to DB/Redis.
- Keep secrets out of images/git. Use env vars.
- Keep code mount **:ro** in dev.
- Commit Compose files; separate prod overrides.

---

## 20. Mental model recap

- Image = recipe (layers); Container = running instance.  
- **-p** maps host traffic; **0.0.0.0** = all interfaces.  
- Bind mount for fast dev; volume for data; volume specificity avoids overwrites.  
- Compose for stacks; Swarm for rolling updates.

---
