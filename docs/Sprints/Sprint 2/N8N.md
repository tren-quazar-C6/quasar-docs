# n8n Installation & Hello World

n8n is a workflow automation tool. It lets you connect apps, APIs, and services together using a visual interface — without writing code. In this project, n8n runs as a Docker container behind Nginx.

## 1. Make sure the nginx-network exists

```bash
docker network create nginx-network
```

If it already exists, Docker will show an error message — that's fine, just continue.

## 2. Run the n8n container

```bash
docker run -d \
  --name n8n \
  --network nginx-network \
  -e N8N_HOST=YOUR_SERVER_IP \
  -e WEBHOOK_URL=http://YOUR_SERVER_IP/ \
  n8nio/n8n
```

:::note
Do **not** add `-p 5678:5678`. Nginx handles all traffic — n8n should never be exposed directly to the internet.
:::

## 3. Verify n8n is running

```bash
docker ps | grep n8n
```

You should see the container with status `Up`.

## 4. Access n8n

Open your browser and go to:

```
http://YOUR_SERVER_IP
```

You will be prompted to create an admin account on first access.

## 5. Hello World — Webhook Workflow

This is a simple workflow to verify n8n is working correctly. It creates an endpoint that responds with a JSON message when visited.

1. Click **"Add first workflow"** on the n8n dashboard
2. Click the **"+"** button to add a node and search for **Webhook**
3. Configure the Webhook node:
   - **HTTP Method:** GET
   - **Path:** `hello`
4. Click **"+"** again and search for **Respond to Webhook**
5. Configure the Respond to Webhook node:
   - **Respond With:** JSON
   - **Response Body:**
     ```json
     { "message": "Hello from n8n!" }
     ```
6. Connect the two nodes by dragging from the Webhook output to the Respond to Webhook input
7. Click **"Save"** and then toggle the workflow to **Active**

## 6. Test the workflow

Run this command from the server terminal:

```bash
curl http://YOUR_SERVER_IP/webhook/hello
```

Expected response:

```json
{ "message": "Hello from n8n!" }
```

If you see this response, n8n is fully installed, running behind Nginx, and processing workflows correctly.