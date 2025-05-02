# Local Observability Lab: Test Your Logs with Loki & Grafana

This project provides a local stack using Docker Compose that includes **Grafana**, **Loki**, and **Promtail** to help you test and fine-tune your custom log parsing rules.

You can feed your own log file (e.g., `app.log`) and define multiline or structured parsing configurations in the `promtail.yaml`. It’s a great way to validate how your logs will appear and behave before sending them to production observability platforms.

---

## Features

- Spin up Grafana, Loki, and Promtail locally via Docker.
- Automatically preconfigure Grafana to use Loki as a data source.
- Read logs from a local file (`app.log`) and parse them using custom Promtail pipeline stages.
- Easily update `app.log` to simulate continuous log streaming.
- Define and test custom parsing rules in `promtail.yaml`.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/) installed

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/log-parsing-playground.git
cd log-parsing-playground

```

### 2. Prepare Your Log File

Edit `app.log` with your own application logs. For example:

```
An exception occurred: Error: This is a test exception
    at file.js:12:11
    at processTicksAndRejections (node:internal/process/task_queues:96:5)

```

You can keep appending logs here during runtime.

### 3. Define Parsing Logic in Promtail

Update `promtail.yaml` under `pipeline_stages` to suit your log format. For multiline logs:

```yaml
pipeline_stages:
  - multiline:
      firstline: "^An exception occurred:"
      max_wait_time: 3s
      max_lines: 128
  - regex:
      expression: '^An exception occurred: (?P<error_message>.*)'

```

### 4. Start the Stack

```bash
docker-compose up --build

```

This will launch:

- Grafana (at [http://localhost:3000](http://localhost:3000/))
    - Username: `admin`
    - Password: `admin`
- Loki (on port `3100`)
- Promtail (tailing `app.log`)

---

## Continuous Testing Instructions

Promtail continuously watches `app.log`. To simulate streaming logs:

### Option 1: Manually Append Logs

```bash
echo "An exception occurred: Error: Disk is full" >> app.log

```

### Option 2: Simulate a Log Generator

You can use a script to append logs every few seconds:

```bash
watch -n 2 "echo 'An exception occurred: Error: Simulated failure' >> logs/app.log"

```

This will add a new log every 2 seconds.

### Option 3: Replace `app.log` with Your Log File

You can also symlink your own log file:

```bash
ln -sf /path/to/your/logs/app.log logs/app.log
```

Promtail will continue tailing it as long as the file path remains valid.

---

## How to View Logs in Grafana

1. Open your browser at [http://localhost:3000](http://localhost:3000/)
2. Login with `admin` / `admin`
3. Go to **Explore**
4. Select `Loki` as your data source
5. Run the following query:

```
{job="app"}

```

1. You should see your parsed log entries. Click on an entry to inspect extracted fields.

---

## File Structure

```
.
├── app.log                  # Your sample log file
├── docker-compose.yaml      # Stack definition
├── promtail.yaml            # Log tailing and parsing config
├── grafana/provisioning/    # Loki data source is preconfigured here
│   └── datasources/
│       └── loki.yaml

```

---

## Stopping the Stack

```bash
docker-compose down

```

---

## Notes

- Ensure that `app.log` exists before starting the stack. Promtail expects the file to be present.
- Changes to `promtail.yaml` require restarting the stack.

---