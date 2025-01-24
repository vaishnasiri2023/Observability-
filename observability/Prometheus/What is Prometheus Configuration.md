### What is Prometheus Configuration?

Prometheus is an open-source monitoring and alerting toolkit. Its configuration is managed through a **YAML** file, usually named `prometheus.yml`. This file defines:

1. **Global configurations** (e.g., scrape intervals, timeouts).
2. **Scrape configurations** (e.g., the targets to monitor).
3. **Rules for alerts** (e.g., alerting thresholds).
4. **Remote write and read** (e.g., integration with long-term storage solutions).
5. **Other settings** like service discovery.

---

### Prometheus Configuration File Structure

Here's an example `prometheus.yml` file and an explanation of each line:

```yaml
# Global settings for Prometheus
global:
  scrape_interval: 15s        # How often Prometheus scrapes targets by default.
  evaluation_interval: 15s   # How often Prometheus evaluates alerting rules.

  # Optional: A default timeout for scraping targets.
  scrape_timeout: 10s

# Define alerting configurations (if alertmanager is used)
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']  # Alertmanager endpoint.

# Rule files for alerts or recording rules
rule_files:
  - 'alert.rules.yml'  # Include external rules for alerts.

# Define scrape configurations
scrape_configs:
  # Monitor Prometheus itself
  - job_name: 'prometheus'    # Name for the scrape job.
    static_configs:
      - targets: ['localhost:9090']  # Targets for this job (Prometheus itself).

  # Monitor a new server
  - job_name: 'new_server'    # Name for the scrape job.
    static_configs:
      - targets: ['192.168.1.100:9100']  # IP address and port of the new server.
        labels:
          environment: 'production'    # Custom labels to identify the target.
          server_role: 'web'           # Server role information.
```

---

### Explanation of Each Section

1. **Global Configuration (`global`):**
   - `scrape_interval`: Defines the default frequency at which Prometheus scrapes targets. 
   - `evaluation_interval`: Determines how often Prometheus evaluates rules (alerting or recording).
   - `scrape_timeout`: Maximum time Prometheus waits for a target to respond before marking it down.

2. **Alerting Configuration (`alerting`):**
   - Configures Prometheus to send alerts to Alertmanager.
   - `static_configs`: Lists the Alertmanager endpoints.

3. **Rule Files (`rule_files`):**
   - Includes external rule files for alerting or recording metrics.

4. **Scrape Configurations (`scrape_configs`):**
   - Defines how Prometheus scrapes targets and collects metrics.
   - `job_name`: A logical name for the group of targets.
   - `static_configs`: Manually defined targets. Each target is an IP or hostname with a port.
   - `labels`: Assign custom labels to metrics scraped from this job for better organization and querying.

---

### Example: Adding a New Server to Monitor

#### Step 1: Install Node Exporter on the New Server
Prometheus uses exporters to collect metrics. For instance, install the **Node Exporter** to monitor system metrics like CPU, memory, and disk usage.

1. SSH into the new server.
2. Download and run the Node Exporter:
   ```bash
   wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz
   tar xvfz node_exporter-*.linux-amd64.tar.gz
   cd node_exporter-*.linux-amd64
   ./node_exporter &
   ```

#### Step 2: Update `prometheus.yml` to Add the New Server
Add a new scrape job in the `scrape_configs` section:

```yaml
scrape_configs:
  - job_name: 'new_server'
    static_configs:
      - targets: ['<server-ip>:9100']
        labels:
          environment: 'staging'
          server_role: 'database'
```

Replace `<server-ip>` with the actual IP of the new server.

#### Step 3: Reload Prometheus Configuration
Prometheus can reload its configuration without restarting. Run the following command on the Prometheus server:

```bash
curl -X POST http://localhost:9090/-/reload
```

#### Step 4: Verify the Configuration
1. Go to the Prometheus web UI (`http://localhost:9090`).
2. Navigate to **Status > Targets** to see if the new server is listed and being scraped.

---

### Monitoring the New Server
Once added, metrics from the new server will be available in Prometheus. Examples include:

1. **CPU Usage:**
   ```promql
   rate(node_cpu_seconds_total{mode="user", instance="<server-ip>:9100"}[5m])
   ```

2. **Memory Usage:**
   ```promql
   node_memory_MemAvailable_bytes{instance="<server-ip>:9100"} / node_memory_MemTotal_bytes{instance="<server-ip>:9100"}
   ```

3. **Disk Usage:**
   ```promql
   1 - (node_filesystem_avail_bytes{instance="<server-ip>:9100"} / node_filesystem_size_bytes{instance="<server-ip>:9100"})
   ```

Note: Replace `<server-ip>` with the actual IP of the server.

