# engage-plus-metrics-infra
Docker setup that allows Engage+ to visualize metrics on Prometheus and Grafana

# How to Run

1. Modify host file (`C:\Windows\System32\drivers\etc\hosts` for Windows and `/etc/hosts` for Linux/MacOS)
    ```
    127.0.0.1 engage-plus-grafana
    127.0.0.1 smf-grafana
    ```
2. Modify `smf/docker-compose.yml` to replace `kafka-ui` environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` to access Kafka on other environments
3. Run `docker-compose up -d` in both `engage-plus` and `smf` folders.
4. Navigate to localhost:9090 (for SMF) or localhost:9091 (for Engage+) to access Prometheus.
5. Navigate to smf-grafana:3000 (for SMF) or engage-plus-grafana:3001 (for Engage+) to access Grafana, using both `admin` as the username and password to login.
6. PES .NET SDK usage metrics should be available after collecting metrics using Prometheus in .NET SDK and exposing the metrics to the correct endpoint in Engage+.
7. Ensure that Prometheus is added as datasource with url `http://prometheus:9090` in Grafana. ![alt text](/images/grafana-datasource.png)

# Adding PES .NET SDK Usage Metrics

1. Perform any PES activities (e.g. Campaign Launch, Send etc.)
2. Refer to https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/create-dashboard/ on how to create a new dashboard. When ask to select datasource, select "Prometheus" as the datasource that is created.
3. Refer to https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/ on how to add visualizations, we will need to add the following metrics:

    a. `rate(pes_sdk_<channel>_send[1m]) * 60` to visualize the number of channel sent in a minute

    b. `pes_sdk_<channel>_success` to visualize the total number of channel sent successfully

    b. `pes_sdk_<channel>_error` to visualize the total number of channel is not sent due to errors

    b. `pes_sdk_<channel>_invalid` to visualize the total number of channel is not sent due to invalid errors

4. Feel free to play around the metrics that's collected from PES .NET SDK

# Configure Prometheus to Scrape from New Metric Endpoint in Engage+

1. Navigate to `engage-plus/prometheus/prometheus.yml`
2. Add the following to `scrape_configs`:
    ```
      - job_name: 'xyz_cms'             # can change to any job name
        metrics_path: '/cms/metrics'    # can change according to the metric endpoint that is exposed
        scheme: 'http'
        static_configs:
        - targets: ['host.docker.internal:80']
        tls_config:
        insecure_skip_verify: true
    ```