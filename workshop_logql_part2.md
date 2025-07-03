# Advanced LogQL Tutorial: Filtering and Analyzing JSON Logs in Grafana Loki

## Introduction

As log data grows in complexity, especially with microservices and distributed systems, advanced filtering and analysis become essential. Loki and Grafana, when combined with JSON logs, provide powerful tools for deep log analysis. This tutorial covers advanced LogQL techniques for filtering, extracting, and visualizing JSON log data.

---

## 1. Understanding JSON Logs in Loki

- **JSON logs** are structured, human-readable, and flexible.
- Loki indexes only metadata (labels), not the full log content, making it efficient for large-scale log storage.
- Grafana visualizes and queries logs from Loki, automatically detecting fields in well-structured JSON logs.

**Example JSON log:**
```json
{
  "timestamp": "2024-09-25T10:15:30Z",
  "level": "error",
  "message": "Failed to process payment",
  "service": "payment-service",
  "request_id": "abc123",
  "user": { "id": "user_98765", "email": "user@example.com" },
  "transaction": { "id": "txn_456789", "amount": 100.50, "currency": "USD" },
  "http": { "method": "POST", "status_code": 500, "path": "/process-payment", "client_ip": "192.168.1.1" }
}
```

---

## 2. Extracting and Filtering JSON Fields

### a. Extract All JSON Fields

Use the `| json` operator to parse and extract all fields:
```logql
{job="python-server"} | json
```

### b. Filter by Specific Field Value

Filter logs where a JSON field matches a value:
```logql
{job="python-server"} | json | level="error"
```

### c. Combine Multiple Field Filters

Chain filters for more precise queries:
```logql
{job="python-server"} | json | level="error" | method="POST"
```

---

## 3. Advanced Filtering Techniques

### a. Regular Expressions on JSON Fields

Use regex to match patterns in JSON fields:
```logql
{job="python-server"} | json | message =~ "Home.*"
```

### b. Filtering Nested Fields

Access nested fields using dot notation:
```logql
{app="myapp"} | json | error.code == 500
```

### c. Dynamic Filtering with Variables

Grafana dashboard variables can be used in queries:
```logql
{job="python-server"} | json | method="$httpMethod"
```
Where `$httpMethod` is a Grafana variable (e.g., GET, POST).

### d. Derived Fields and Custom Output

Format output using `line_format`:
```logql
{job="python-server"} | json | line_format "{{ .timestamp }} | {{ .level }} | {{ .http.method }} | {{ .message }}"
```

---

## 4. Best Practices for Efficient JSON Log Filtering

- **Use labels for pre-filtering**: Always start with label selectors to reduce the data scanned.
- **Limit time range**: Query only the necessary time window.
- **Optimize regex**: Use specific patterns to minimize processing.
- **Consistent JSON structure**: Ensure logs are well-formed and fields are named consistently.
- **Index logs efficiently**: Use meaningful labels in your log pipeline.

---

## 5. Troubleshooting Common Issues

- **Inconsistent JSON**: Ensure all logs follow a consistent structure.
- **Nested objects**: Use dot notation for nested fields.
- **Field detection issues**: Use explicit parsing (`| json field="message"`) if needed.
- **Slow queries**: Use label filters and time range limits.

---

## 6. Example Queries

- All error logs for a service:
  ```logql
  {service="payment-service"} | json | level="error"
  ```
- All POST requests with status 500:
  ```logql
  {service="payment-service"} | json | method="POST" | http.status_code=500
  ```
- Regex match on message:
  ```logql
  {service="payment-service"} | json | message =~ "timeout.*"
  ```
- Nested field filter:
  ```logql
  {service="payment-service"} | json | user.id="user_98765"
  ```

---

## 7. Key Takeaways

- Use `| json` to extract fields for advanced filtering.
- Combine multiple filters and regex for complex queries.
- Use labels and time ranges for efficient queries.
- Leverage Grafana variables for dynamic dashboards.

---

## 8. More LogQL Examples and Comparisons

Here are additional advanced LogQL examples and comparisons, inspired by the [ruanbekker/cheatsheets](https://github.com/ruanbekker/cheatsheets/blob/master/loki/logql/README.md):

### Basic and Combined Filters

- View all logs for a job:
  ```logql
  {job="dev/logs"}
  ```
- View logs for a specific file:
  ```logql
  {job="dev/logs", filename="/var/log/app.log"}
  ```
- Search for logs with an exact match (case-insensitive):
  ```logql
  {job="dev/logs"} |= "(?i)this is a test"
  ```
- Exclude logs with a specific field:
  ```logql
  {job="dev/logs"} |= "This is a test" != "testerId=123"
  ```
- Include logs with a pattern and exclude device IDs:
  ```logql
  {job="dev/logs"} |= "This is a test" |= "accountId=000" !~ "deviceId=(001|209)"
  ```

### Aggregation and Metrics

- Log events per container:
  ```logql
  sum by(container_name) (rate({job="prod/dockerlogs"}[1m]))
  ```
- Count log events and sum by pod:
  ```logql
  sum by (app)(count_over_time({app=~"my-service"} | json | line_format "{{.log}}" |~ "Unable to acquire"[$__interval]))
  ```

### Parsing and Formatting

- Parse JSON after a regex match and filter:
  ```logql
  {job="dev/logs"} | regexp `\[(?P<timestamps>(.*))\] (?P<environment>(prod|dev)).(?P<loglevel>(INFO|DEBUG|ERROR|WARN)): (?P<jsonstring>(.*))` | line_format "{{.jsonstring}}" | json | __error__ != "JSONParserErr" | logTag="BalanceCheck"
  ```
  *Explanation*: This regex extracts multiple fields from log lines that start with a timestamp in brackets, followed by the environment (prod or dev), log level (INFO, DEBUG, ERROR, WARN), and a JSON string. The named group `jsonstring` captures the JSON part for further parsing.

- Show logs for balances more than 8:
  ```logql
  {job="dev/logs"} | regexp `\[(?P<timestamps>(.*))\] (?P<environment>(prod|dev)).(?P<loglevel>(INFO|DEBUG|ERROR|WARN)): (?P<jsonstring>(.*))` | line_format "{{.jsonstring}}" | json | __error__ != "JSONParserErr" | logTag="BalanceCheck" | balance > 8
  ```
  *Explanation*: This uses the same regex as above to extract the JSON string, which is then parsed to filter logs where the `balance` field is greater than 8.

- Custom line formatting:
  ```logql
  {job="containerlogs"} | json | line_format "timestamp={{ .time }} source_ip={{ .req_headers_x_real_ip }} method={{ .req_method }} path={{ .req_url }} status_code={{ .res_statusCode }}"
  ```
  *Explanation*: No regex is used in this example. Instead, it uses `line_format` to create a custom output from parsed JSON fields.

### Regex and Advanced Extraction

- Extract fields from NGINX logs using regex:
  ```logql
  {job="prod/nginx"} | regexp `(?P<ip>\\S+) (?P<identd>\\S+) (?P<user>\\S+) \\[(?P<timestamp>[\w:\/]+\s[+\\-]\d{4})\] "(?P<action>\\S+)\\s?(?P<path>\\S+)\\s?(?P<protocol>\\S+)?" (?P<status>\d{3}|-) (?P<size>\d+|-)\\s?"?(?P<referrer>[^\"]*)"?\\s?"?(?P<useragent>[^\"]*)"?`
  ```
  *Explanation*: This regex is designed to parse standard NGINX access log lines. It extracts fields such as IP address (`ip`), identd, user, timestamp, HTTP action (method), path, protocol, status code, response size, referrer, and user agent. Each part of the log line is captured using named groups for easy reference in LogQL.

- Filter by status code after regex extraction:
  ```logql
  {job="prod/logs"} |~ "doAPICall" | regexp "(?P<ip>\\d+.\\d+.\\d+.\\d+) (.*) (.*) (?P<date>\\[(.*)\\]) (\")(?P<verb>(\\w+)) (?P<request_path>(([^\"]*))) (?P<http_ver>(([^\"]*)))(\") (?P<status_code>\\d+) (?P<bytes>\\d+) (\")(?P<referrer>(([^\"]*)))(\") (\")(?P<user_agent>(([^\"]*)))(\")" | status_code=201
  ```
  *Explanation*: This regex extracts fields from log lines that include an IP address, date, HTTP verb, request path, HTTP version, status code, bytes, referrer, and user agent. The named group `status_code` is used to filter for logs where the status code is 201. This is useful for analyzing HTTP request logs and filtering by response status.

### Accessing Nested JSON and Error Filtering

- Access nested JSON and filter out parser errors:
  ```logql
  {namespace=~"$namespace", app=~"$app", pod=~"$pod"} | json  | line_format "{{.log}}" | json raw_body="message" | line_format "m: {{.raw_body}}" | __error__!="JSONParserErr"
  ```
- Filter out JSONParserErr and show message and stacktrace:
  ```logql
  {container="my-service"} |= `` | json | __error__!="JSONParserErr" | line_format "{{.message}} {{.stacktrace}}"
  ```

### Comparison: LogQL vs PromQL

- **LogQL** is designed for logs, supports parsing, filtering, and formatting log lines, and can extract fields from unstructured data.
- **PromQL** is for metrics, supports mathematical operations and aggregations on time series data, but does not parse log content.
- LogQL can combine log content parsing (e.g., `| json`, `| regexp`) with metric queries, which is not possible in PromQL.

---

For more, see the [full guide](https://signoz.io/guides/loki-json-logs-filter-by-detected-fields-from-grafana/).
