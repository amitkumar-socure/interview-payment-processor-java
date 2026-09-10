# Payment Processor

A Spring Boot application that processes payment messages from an in-memory queue using multiple worker threads. No external services are required: the queue and database (H2) are in-memory.

## Prerequisites

- Java 17+
- No Maven install required — this repo includes the Maven Wrapper (`mvnw` / `mvnw.cmd`), which downloads the correct Maven version automatically.

## Build

```bash
./mvnw clean install
```

On Windows (Command Prompt / PowerShell), use `mvnw.cmd clean install` instead.

## Run

```bash
./mvnw spring-boot:run
```

The app will:

1. Start an in-memory H2 database and create the `payments` table.
2. Start four worker threads that poll the queue and process messages.
3. Expose REST API at `http://localhost:8080`.

## API

- **Submit a payment for processing** (async; workers will process it):

  ```bash
  curl -X POST http://localhost:8080/api/payments/submit \
    -H "Content-Type: application/json" \
    -d '{"paymentId": "99", "amount": 50.00}'
  ```

  Returns `202 Accepted` with `{"status":"accepted","paymentId":"99"}`.

- **Get the most recent payments** (ordered by creation time, newest first):

  ```bash
  curl -s http://localhost:8080/api/payments/recent | jq
  ```

  Optional query param: `limit` (default 10, max 100). Example: `curl -s "http://localhost:8080/api/payments/recent?limit=25" | jq`

  Returns `200 OK` with a JSON array of payment objects (`id`, `paymentId`, `amount`, `createdAt`).

## H2 Console (optional)

When the app is running, you can inspect the database at:

- URL: http://localhost:8080/h2-console  
- JDBC URL: `jdbc:h2:mem:paymentdb`  
- Username: `sa`  
- Password: (leave empty)

Use the H2 console to query `SELECT * FROM payments;` and see payment rows.

## Configuration

Config is in `src/main/resources/application.yml`.

| Property | Description | Default |
|---|---|---|
| `server.port` | HTTP listen port | `8080` |
| `app.queue.visibility-timeout-seconds` | Visibility timeout for received messages | `30` |
| `app.worker.thread-count` | Number of worker threads | `4` |
| `app.worker.poll-interval-ms` | Delay between polls when queue is empty | `500` |

To override the port without editing the file:

```bash
./mvnw spring-boot:run -Dspring-boot.run.arguments=--server.port=9090
```
