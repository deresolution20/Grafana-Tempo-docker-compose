# Grafana and Tempo Docker Setup

This repository contains a simple Docker Compose setup for Grafana and Tempo, designed for easy replication and testing.

## **Usage**

### **Prerequisites**

- Docker and Docker Compose installed on your system.

### **Setup Steps**

1. **Clone the Repository**

   ```bash
   git clone https://github.com/yourusername/your-repo-name.git
   cd your-repo-name
Start the Docker Compose Stack

bash
Copy code
docker-compose up -d
Send the Sample Trace

bash
Copy code
curl -X POST http://localhost:4318/v1/traces \
     -H "Content-Type: application/json" \
     -d @sample_trace.json
Access Grafana

Navigate to http://localhost:3000 in your browser.
Login credentials:
Username: admin
Password: admin
View the Trace

Click on the Explore icon (Compass).
Select Tempo as the data source.
Enter the Trace ID: 00000000000000000000000000000001.
Run the query to view the trace and its attributes.
Files Included
docker-compose.yml: Docker Compose configuration file.
tempo.yaml: Tempo configuration file.
tempo-datasource.yml: Grafana data source provisioning file.
sample_trace.json: Sample trace data.
Notes
Data Persistence: Docker volumes are used to persist data across restarts.
No Extra Folders: All necessary files are in the root directory for simplicity.
