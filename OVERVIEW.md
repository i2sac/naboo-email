# Naboo Email Server

Naboo Email Server is a gRPC-based Go application for securely sending emails via SMTP with TLS encryption. It uses an SMTP client connection pooling mechanism to improve performance, reliability, and scalability.

The project is open-source on GitHub and is ideal for public use. Lightweight multi-platform Docker images are published on Docker Hub under [`isaacpeterson/naboomail`](https://hub.docker.com/r/isaacpeterson/naboomail).

---

## Features

- **gRPC-based email API** for sending emails from your services
- **TLS 1.2+ encryption** for all SMTP connections
- **Connection pooling** for efficient SMTP client reuse
- **Ultra-lightweight image** (~15MB) using a scratch-based multi-stage build
- **Multi-platform support**: `linux/amd64`, `linux/arm64`
- **Non-root container** for improved security

---

## Quick Start

Always specify a tag when pulling the image (e.g. `:latest` or a version like `:v1.0.0`).

Pull the latest image:

```bash
docker pull isaacpeterson/naboomail:latest
```

Or pull a specific version:

```bash
docker pull isaacpeterson/naboomail:v1.0.0
```

Run the server (using `:latest`, or replace with your desired tag such as `:v1.0.0`):

```bash
docker run -d \
  --name naboomail \
  -p 50051:50051 \
  -e EMAIL_ADDRESS="your-email@example.com" \
  -e EMAIL_PWD="your-password" \
  -e SMTP_HOST="smtp.example.com" \
  -e SMTP_PORT="465" \
  -e POOL_SIZE="10" \
  isaacpeterson/naboomail:latest
```

The gRPC server listens on `0.0.0.0:50051`.

---

## Configuration

The server is configured via environment variables:

- **EMAIL_ADDRESS**  
  Sender email address (e.g. `your-email@example.com`).

- **EMAIL_PWD**  
  Password or app-specific password for the sender account.

- **SMTP_HOST**  
  SMTP server hostname (e.g. `smtp.example.com`).

- **SMTP_PORT**  
  SMTP server port (e.g. `465` for SMTPS).

- **POOL_SIZE** *(optional)*  
  Size of the SMTP connection pool. Defaults to `5` if not set.

Example `.env` file:

```bash
EMAIL_ADDRESS="your-email@example.com"
EMAIL_PWD="your-email-password"
SMTP_HOST="smtp.example.com"
SMTP_PORT="465"
POOL_SIZE="10"  # Optional, defaults to 5 if not set
```

---

## Available Tags

Commonly used tags:

- `latest` – Latest stable release (recommended if you want the most recent version)
- `v1.0.0` – Specific pinned version

Always use an explicit tag such as `:latest` or a version like `:v1.0.0` when pulling or running the image.

---

## Architecture Overview

- **gRPC Server**  
  Exposes a `SendEmail` method to send emails via RPC.

- **EmailService**  
  - Manages SMTP client connections with TLS encryption  
  - Handles SMTP authentication  
  - Implements connection pooling for efficient resource use  
  - Executes SMTP commands (`MAIL FROM`, `RCPT TO`, `DATA`) to send emails

- **SMTP Integration**  
  Uses Go’s standard `net/smtp` and `crypto/tls` packages.

---

## API

### Proto (excerpt)

```proto
message SendEmailRequest {
    string email_target = 1;
    string subject = 2;
    string message = 3;
}

message SendEmailReply {
    string message = 1;
}
```

### Go Client Example

```go
client.SendEmail(ctx, &pb.SendEmailRequest{
    EmailTarget: "recipient@example.com",
    Subject:     "Greetings from Naboo",
    Message:     "<h1>Hello!</h1><p>This is a test email from Naboo Email Server.</p>",
})
```

---

## Building from Source (Optional)

If you prefer not to use the pre-built Docker image:

```bash
git clone https://github.com/i2sac/naboo-email.git
cd naboo-email

go mod tidy

cd cmd
go build -o naboo-email-server main.go
./naboo-email-server
```

The server will listen on `0.0.0.0:50051`.

To build and run a local Docker image:

```bash
docker build -t naboo-email:local .

docker run -d \
  --name naboo-email \
  -p 50051:50051 \
  --env-file .env \
  naboo-email:local
```

---

## License

Naboo Email Server is licensed under the MIT License. See the LICENSE file in the GitHub repository for details.

