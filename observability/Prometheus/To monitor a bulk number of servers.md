### To monitor a bulk number of servers

To monitor a bulk number of servers (e.g., 100 systems) with Prometheus to check if their HTTP services are up or not, you can use **static configurations**, **file-based service discovery**, or **dynamic service discovery** (e.g., DNS, Consul, etc.). Below, I’ll explain each method and how to configure the `prometheus.yml` file.

---

### Method 1: Static Configuration (Manual Listing)
In this approach, you manually define all 100 systems in the `static_configs`.

#### `prometheus.yml` Configuration:
```yaml
global:
  scrape_interval: 15s  # Scrape every 15 seconds

scrape_configs:
  - job_name: 'http_services'
    metrics_path: '/metrics' # Default scrape path
    static_configs:
      - targets:
          - '192.168.1.1:80'
          - '192.168.1.2:80'
          - '192.168.1.3:80'
          - '192.168.1.4:80'
          # Continue for all 100 servers
      labels:
        environment: 'production'
        service: 'http'
```

Here:
- Each `target` specifies the server's IP and port where the HTTP service runs.
- Custom labels (`environment` and `service`) help to organize and query metrics.

---

### Method 2: File-Based Service Discovery
If your target list changes frequently, you can use a file-based service discovery method. Prometheus watches a file containing the list of servers.

#### Step 1: Create a Target File
Create a file named `targets.json` with the following content:

```json
[
  {"targets": ["192.168.1.1:80", "192.168.1.2:80", "192.168.1.3:80"], "labels": {"environment": "production", "service": "http"}},
  {"targets": ["192.168.2.1:80", "192.168.2.2:80"], "labels": {"environment": "staging", "service": "http"}}
]
```

#### Step 2: Update `prometheus.yml`
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'http_services'
    file_sd_configs:
      - files:
          - '/etc/prometheus/targets.json'
```

Here:
- Prometheus dynamically loads targets from the `targets.json` file.
- You can update `targets.json` without restarting Prometheus.

---

### Method 3: Dynamic Service Discovery
If the server list is maintained in a DNS record or a service discovery tool like **Consul**, **Kubernetes**, or **EC2 Auto Scaling**, Prometheus can scrape dynamically.

#### Example: DNS-Based Service Discovery
Assume your 100 servers are registered under a DNS name `http-services.example.com`.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'http_services'
    dns_sd_configs:
      - names:
          - 'http-services.example.com'
        type: 'A'
        port: 80
    labels:
      environment: 'production'
      service: 'http'
```

Here:
- Prometheus resolves `http-services.example.com` and scrapes all associated IPs.

---

### Configuring Metrics for HTTP Health Checks
To monitor whether HTTP services are up, Prometheus uses the `up` metric, which is generated for all targets. 

#### Query Example:
```promql
up{job="http_services"}
```
- `1` indicates the target is up.
- `0` indicates the target is down.

You can also create an alert for when any target is down:

#### Example Alert Rule:
```yaml
groups:
  - name: http_alerts
    rules:
      - alert: HTTPServiceDown
        expr: up{job="http_services"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "HTTP Service is down on {{ $labels.instance }}"
          description: "The HTTP service on {{ $labels.instance }} is not responding."
```

---

### Best Practices
1. **Automation:**
   - Use tools like Ansible or Terraform to generate the target list dynamically.
2. **Use Labels:**
   - Add meaningful labels (e.g., `region`, `team`) to organize and filter metrics easily.
3. **Scaling Prometheus:**
   - If monitoring 100+ targets leads to performance issues, consider using Prometheus Federation to divide the workload across multiple Prometheus instances.

By leveraging these methods, you can efficiently monitor a large number of systems with Prometheus and ensure all HTTP services are up.