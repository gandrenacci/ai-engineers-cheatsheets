Keeping a Linux Ubuntu server clean of Docker clutter is essential for saving disk space and avoiding performance hiccups. Over time, old images, stopped containers, and unused volumes can swallow up gigabytes of storage.

Here is a step-by-step guide to inspect what’s running and safely purge the leftovers.

---

## 1. Check Your Current Storage Usage

Before deleting anything, see how much space Docker is actually consuming:

```bash
docker system df

```

This gives you a neat breakdown of space used by images, containers, local volumes, and build cache.

---

## 2. Check Running and Stopped Containers

To see only **currently running** containers:

```bash
docker ps

```

To see **all** containers (including stopped, exited, or crashed ones):

```bash
docker ps -a

```

> 💡 **Tip:** Look for containers with a status like `Exited (0) 3 weeks ago`. If you don't need them anymore, they are just wasting space.

---

## 3. Check and Understand Images

To list all the Docker images currently stored on your Ubuntu server:

```bash
docker images -a

```

Look out for two types of images you can clean:

* **Dangling Images (`<none>:<none>`):** These are layers left behind when you build or update an image. They have no relationship to any tagged images anymore.
* **Unused Images:** These are perfectly valid, tagged images (like `nginx:latest`), but they are not currently associated with any running or stopped container.

---

## 4. The Clean-Up Phase

### The "Nuclear Option" (The Safe Way)

If you just want to wipe away everything that isn't actively being used, Docker has a brilliant built-in command. This will remove all stopped containers, dangling images, and unused build caches:

```bash
docker system prune

```

If you want to be extra thorough and also remove **any unused image** (not just dangling ones), add the `-a` flag:

```bash
docker system prune -a

```

*Note: This will prompt you for confirmation `(y/N)` before deleting.*

### Target Clean-Up (If you prefer surgical precision)

* **To remove specific stopped containers:**
```bash
docker rm <container_id_or_name>
```


* **To remove all stopped containers at once:**
```bash
docker container prune

```


* **To remove only dangling images:**
```bash
docker image prune

```


* **To remove specific images by ID or name:**
```bash
docker rmi <image_id_or_name>

```



---

## 5. Don't Forget the Docker Volumes!

Volumes hold persistent data. When you delete a container, its volume stays behind by default. These can accumulate quickly.

To see unused volumes:

```bash
docker volume ls -q -f dangling=true

```

To wipe out **all** unused volumes:

```bash
docker volume prune

```

> ⚠️ **Warning:** Be careful with this one! If you have an inactive database container but need its data later, pruning the volume will permanently delete that data.