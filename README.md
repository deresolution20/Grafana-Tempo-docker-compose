# Grafana and Tempo Docker Setup

This repository contains a simple and self-contained Docker Compose setup for running Grafana and Tempo locally. It is designed to replicate and test issues related to string array attributes in spans, similar to the customer's scenario you've described.

## Prerequisites

- **Docker** installed on your machine.
- **Docker Compose** installed.

## Setup Instructions

1. ### **Clone the Repository**

   Since this is a local setup, ensure all the necessary files are in the same directory. If you're copying the files manually, create a directory and place the following files inside:

   - `docker-compose.yml`
   - `tempo.yaml`
   - `tempo-datasource.yml`
   - `sample_trace.json`

2. ### **Start the Docker Compose Stack**

   Open a terminal, navigate to your project directory, and run:

   ```bash
   docker-compose up -d
Verify Services Are Running
Check that both Grafana and Tempo services are up:

bash
Copy code
docker-compose ps
You should see output indicating that both services are running.

Send the Sample Trace
Use the following curl command to send the sample trace to Tempo:

bash
Copy code
curl -X POST http://localhost:4318/v1/traces \
     -H "Content-Type: application/json" \
     -d @sample_trace.json
Note: Ensure that sample_trace.json is in the same directory from which you're running the command.

Access Grafana
Open your browser and navigate to http://localhost:3000.
Log in with the following credentials:
Username: admin
Password: admin
Explore the Trace
Click on the Explore icon (the compass) in the left sidebar.
In the data source dropdown at the top, select Tempo.
Enter the Trace ID: 00000000000000000000000000000001.
Click Run Query to search for the trace.
Inspect the Span Attributes
In the trace view, expand the span named "handle watchdog message".

Under Attributes, verify that the ams.ds.ids attribute displays the array:

css
Copy code
["ds_uuid1", "ds_uuid2"]
This confirms that the string array attribute is displayed correctly.

Cleanup
To stop and remove the Docker containers, run the following command in your project directory:

bash
Copy code
docker-compose down
This will stop all running containers and remove them.

Notes
Data Persistence: This setup uses Docker volumes (grafana-data and tempo-data) to persist data across restarts. Your configurations and data will remain intact even after stopping the containers.
All-in-One Directory: All necessary files are included in the same directory for simplicity and ease of use.
Reproducibility: This setup allows you to quickly reproduce and test issues related to Grafana and Tempo, specifically for string array attributes in spans.
Files Included
Below are all the files included in this setup:

docker-compose.yml
yaml
Copy code
version: '3'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - '3000:3000'
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - tempo
    volumes:
      - grafana-data:/var/lib/grafana
      - ./tempo-datasource.yml:/etc/grafana/provisioning/datasources/tempo-datasource.yml
  tempo:
    image: grafana/tempo:latest
    ports:
      - '4318:4318'   # OTLP HTTP receiver port
      - '3100:3100'   # Tempo's query port
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml:ro
      - tempo-data:/tmp/traces

volumes:
  grafana-data:
  tempo-data:
tempo.yaml
yaml
Copy code
auth_enabled: false

server:
  http_listen_port: 3100

distributor:
  receivers:
    otlp:
      protocols:
        http:
          endpoint: "0.0.0.0:4318"

ingester:
  trace_idle_period: 10s
  max_block_duration: 5m

storage:
  trace:
    backend: local
    local:
      path: /tmp/traces

compactor:
  compaction:
    block_retention: 24h
tempo-datasource.yml
yaml
Copy code
apiVersion: 1

datasources:
  - name: Tempo
    type: tempo
    access: proxy
    orgId: 1
    url: http://tempo:3100
    isDefault: true
    version: 1
    editable: false
sample_trace.json
json
Copy code
{
  "resourceSpans": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.name",
            "value": { "stringValue": "test-service" }
          }
        ]
      },
      "scopeSpans": [
        {
          "scope": {
            "name": "test-instrumentation",
            "version": "1.0.0"
          },
          "spans": [
            {
              "traceId": "00000000000000000000000000000001",
              "spanId": "0000000000000001",
              "name": "handle watchdog message",
              "kind": "SPAN_KIND_INTERNAL",
              "startTimeUnixNano": 1698876000000000000,
              "endTimeUnixNano": 1698876001000000000,
              "attributes": [
                {
                  "key": "ams.watchdog.message_type",
                  "value": { "stringValue": "hello" }
                },
                {
                  "key": "ams.ds.ids",
                  "value": {
                    "arrayValue": {
                      "values": [
                        { "stringValue": "ds_uuid1" },
                        { "stringValue": "ds_uuid2" }
                      ]
                    }
                  }
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
License
This project is licensed under the MIT License.
