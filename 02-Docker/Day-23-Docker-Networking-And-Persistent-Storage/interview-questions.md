# Day 23 — Interview Questions: Docker Networking & Persistent Storage

## Beginner Level

### Q1. Why doesn't `localhost` work for MongoDB in a separate container?

Because `localhost` refers to the container where the app is running, not the MongoDB container.

### Q2. What is a Docker network used for?

A Docker network allows containers to communicate with each other using service names and internal DNS.

### Q3. What is a Docker volume?

A Docker volume is persistent storage managed by Docker, used to keep data independent from a container's filesystem.

---

## Intermediate Level

### Q4. Why don't we typically publish MongoDB on port `27017` to the host?

Because the application can talk to MongoDB over a private Docker network. Publishing the database port increases the attack surface without adding necessary value.

### Q5. What is the difference between persistence and backup?

Persistence keeps data alive when the container is removed or restarted. Backup is an independent recovery strategy for restoring data after failure or loss.

### Q6. What does `docker network inspect` help you verify?

It shows which containers are connected to the network and their IP addresses and network metadata.

---

## Advanced Level

### Q7. Your API is running but cannot connect to MongoDB. What is your debugging sequence?

1. Check if both containers are running
2. Check if they are on the same Docker network
3. Verify the hostname is `mongodb`
4. Check MongoDB logs
5. Check API logs
6. Verify environment variables
7. Test DNS from within the app container
8. Validate port and auth settings

### Q8. Why is `mongodb://mongodb:27017/mydb` usually preferred over `mongodb://localhost:27017/mydb` in Docker?

Because `mongodb` is the service name on the Docker network, while `localhost` points to the current container.

### Q9. What is the value of using a named volume like `mongo-data`?

It keeps MongoDB data alive even if the container is removed and allows a replacement container to reuse the same data.

### Q10. How does Docker networking differ from host networking?

Docker networking isolates containers and manages communication between them. Host networking allows a container to use the host's network namespace, which reduces isolation and can create port conflicts.

---

## Short Strong Answers

### "Why does localhost fail in Docker?"

Because `localhost` is local to the current container, not another container.

### "Why do we use a Docker network?"

To let containers talk to each other securely and predictably using names instead of raw IPs.

### "Why is MongoDB not publicly exposed?"

Because it is a database and should remain on a private network unless a strong operational requirement says otherwise.

### "What is the difference between a volume and a backup?"

A volume keeps data available; a backup allows restoration after data loss or disaster.
