# Day 32 - Commands: Docker Production Deployment and Reverse Proxy

## 1. Create the Nginx Directory

```bash
mkdir nginx
```

Create `nginx/nginx.conf` and `nginx/Dockerfile`.

## 2. Validate Compose Configuration

```bash
docker compose config
docker compose config --services
```

## 3. Build and Start the Stack

```bash
docker compose up -d --build
```

## 4. Check Service State

```bash
docker compose ps
docker compose ps -a
```

## 5. View Nginx Logs

```bash
docker compose logs nginx
docker compose logs --tail 100 nginx
docker compose logs -f nginx
```

## 6. View API Logs

```bash
docker compose logs api
docker compose logs --tail 100 api
docker compose logs -f api
```

## 7. Test the Public API Path

```bash
curl http://localhost/api/health
```

This tests the client-to-Nginx-to-API path.

## 8. Test the API Directly During Development

If the API publishes a development port:

```bash
curl http://localhost:3000/health
```

In a production-style setup, the API may not publish a host port because Nginx can reach it internally.

## 9. Inspect Networks

```bash
docker network ls
docker network inspect <project>_mern-network
```

Confirm that Nginx and API share the network.

## 10. Test Service DNS From Nginx

```bash
docker compose exec nginx getent hosts api
```

## 11. Test API Connectivity From Nginx

If the Nginx image includes `wget`:

```bash
docker compose exec nginx wget -qO- http://api:3000/health
```

## 12. Inspect API Configuration

```bash
docker compose exec api printenv MONGO_URI
docker inspect <api-container>
```

The expected internal hostname is `mongodb`, not `localhost`.

## 13. Inspect MongoDB Logs

```bash
docker compose logs --tail 100 mongodb
```

## 14. Inspect Health State

```bash
docker inspect --format='{{json .State.Health}}' <container>
```

## 15. Deliberately Create a 502

Temporarily set:

```nginx
proxy_pass http://wrong-api:3000;
```

Rebuild Nginx:

```bash
docker compose up -d --build nginx
```

Test:

```bash
curl http://localhost/api/health
```

Restore:

```nginx
proxy_pass http://api:3000;
```

Then rebuild and retest.

## 16. 502 Investigation Sequence

```bash
docker compose ps -a
docker compose logs --tail 100 nginx
docker compose logs --tail 100 api
docker network ls
docker network inspect <network>
docker compose exec nginx getent hosts api
docker compose exec nginx wget -qO- http://api:3000/health
curl http://localhost/api/health
```

## 17. Inspect Resource Usage

```bash
docker stats
docker stats --no-stream
docker system df
```

## 18. Check Listening Ports in the API Container

If the image contains the tool:

```bash
docker compose exec api ss -lntp
```

The API should listen on the expected internal port and interface.

## 19. Inspect Nginx Configuration

```bash
docker compose exec nginx nginx -t
docker compose exec nginx cat /etc/nginx/conf.d/default.conf
```

## 20. Run the Diagnostic Script

Linux/macOS:

```bash
chmod +x diagnose-stack.sh
./diagnose-stack.sh
```

Adapt commands such as `grep` or service-state checks for PowerShell on Windows.
