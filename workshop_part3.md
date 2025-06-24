# PromQL Workshop – Part 3: Advanced Patterns and Use Cases

Welcome to Part 3 of the PromQL Workshop! This section covers advanced query patterns, real-world use cases, and tips for building powerful dashboards and alerts.

---

## 1. Subqueries

Subqueries allow you to apply a query over the result of another query, enabling more complex analysis.

### **Syntax**
```promql
<query>[<range>:<resolution>]
```
- `<range>`: How far back to look.
- `<resolution>`: Step between data points.

### **Example**
```promql
sum(rate(http_requests_total[5m]))[30m:5m]
```
> Calculates the sum of request rates every 5 minutes, over the last 30 minutes.

---

## 2. Recording Rules

Recording rules let you precompute frequently used or expensive queries and save them as new time series.

### **Example**
```yaml
groups:
  - name: example.rules
    rules:
      - record: job:http_requests:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)
```
> This creates a new metric `job:http_requests:rate5m` for use in dashboards and alerts.

---

## 3. Alerting Rules

Alerting rules evaluate queries and trigger alerts when conditions are met.

### **Example**
```yaml
groups:
  - name: example.rules
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status="500"}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
```
> Triggers an alert if the error rate exceeds 0.05 requests/sec for 5 minutes.

---

## 4. Combining Metrics

You can combine different metrics using arithmetic and logical operators.

### **Example: Error Rate Percentage**
```promql
100 * sum(rate(http_requests_total{status="500"}[5m])) 
  / sum(rate(http_requests_total[5m]))
```
> Calculates the percentage of requests that resulted in errors.

---

## 5. Useful Dashboard Patterns

- **Current value:**  
  ```promql
  http_requests_total
  ```
- **Rate over time:**  
  ```promql
  rate(http_requests_total[5m])
  ```
- **Rolling average:**  
  ```promql
  avg_over_time(rate(http_requests_total[1m])[30m:1m])
  ```
- **Percentile latency:**  
  ```promql
  histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
  ```

---

## 6. Tips and Best Practices

- Use `rate()` for counters, not `gauge` metrics.
- Always specify a range for rate functions (e.g., `[5m]`).
- Use `by` clauses to group results for dashboards.
- Precompute expensive queries with recording rules.
- Use subqueries for advanced time-based analysis.

---

## 7. Resources

- [Prometheus Querying Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [PromLabs PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
- [Awesome Prometheus Alerts](https://samber.github.io/awesome-prometheus-alerts/)