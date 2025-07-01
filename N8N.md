# 🚀 Self-Hosting n8n - Complete Guide

This guide explains how to **self-host n8n** (the powerful workflow automation tool) using Docker, manage execution history, secure access, and optionally run it behind a custom domain with SSL.

---

## 📌 What is n8n?

**n8n** is an open-source workflow automation tool that allows you to connect apps, APIs, and services with no-code or low-code workflows.

- Source: [https://n8n.io](https://n8n.io)
- License: [Sustainable Use License (SUL)](https://github.com/n8n-io/n8n/blob/master/packages/cli/LICENSE.md)

---

## 💸 Is Self-Hosting Free?

Yes! You can **self-host n8n for free** if:

✅ You use it for **internal automations**  
❌ You **do not** expose it to customers as a service or embed it in your SaaS

If you're offering workflow features to your end users, you’ll need a **commercial license**.

---

## 🐳 Docker Run Command (with history pruning and auth)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e EXECUTIONS_DATA_PRUNE=true \
  -e EXECUTIONS_DATA_MAX_AGE=168 \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=strongpassword \
  n8nio/n8n
```

### 🔐 Auth Details

* Username: `admin`
* Password: `strongpassword` (change this!)

### 🧹 Execution History Cleanup

* `EXECUTIONS_DATA_PRUNE=true`: enables auto-deletion
* `EXECUTIONS_DATA_MAX_AGE=168`: delete executions older than 7 days (in hours)

---

## 📂 Persistent Data Location

Your workflows, credentials, and settings are stored at:

```bash
~/.n8n
```

Make sure this directory is **backed up regularly** if using in production.

---

## 🔧 Optional: Disable Execution Logging

To reduce disk usage further:

```env
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
EXECUTIONS_DATA_SAVE_ON_ERROR=none
```

Add as environment variables in your `docker run` or `docker-compose.yml`.

---

## 🧼 Manual Cleanup via SQL (if needed)

If auto-prune is not enabled, you can delete old executions manually:

### For SQLite:

```sql
DELETE FROM execution_entity WHERE startedAt < datetime('now', '-7 days');
```

### For PostgreSQL:

```sql
DELETE FROM execution_entity WHERE "startedAt" < NOW() - INTERVAL '7 days';
```

**⚠️ Always backup your DB before executing delete queries.**

---

## 🔐 Securing n8n

### Enable Basic Auth (already in command):

```env
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=strongpassword
```

### HTTPS Setup (Optional)

Use NGINX/Traefik + Let's Encrypt or reverse proxy with SSL termination.

---

## 🧰 Docker Compose (Recommended for Prod)

Create a `docker-compose.yml`:

```yaml
version: '3.7'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=strongpassword
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168
    volumes:
      - ~/.n8n:/home/node/.n8n
```

Run with:

```bash
docker-compose up -d
```

---

## 🌐 Accessing n8n

Once running:

* Open [http://localhost:5678](http://localhost:5678)
* Or use your server IP/domain: `http://<your-ip>:5678`
* Login with your `admin` credentials

---

## ✅ Summary

| Feature                 | Status                |
| ----------------------- | --------------------- |
| Self-hosting cost       | ✅ Free                |
| Execution logging       | ✅ Pruned after 7 days |
| Basic auth              | ✅ Enabled             |
| Persistent storage      | ✅ Using volume        |
| Reverse proxy / SSL     | 🔄 Optional           |
| Commercial use allowed? | ❌ Not for SaaS resale |

---

## 🧠 Tips

* Use Postgres in production (more scalable than SQLite).
* Regularly back up `.n8n` folder or database.
* Monitor disk usage if you’re handling many executions.

---

## 📎 References

* [n8n Docs](https://docs.n8n.io)
* [GitHub](https://github.com/n8n-io/n8n)
