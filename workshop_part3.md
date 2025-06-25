--

# PromQL Workshop – Part 3: Advanced Patterns and Use Cases

Welcome to Part 3 of the PromQL Workshop! This section covers advanced query patterns, real-world use cases, and tips for building powerful dashboards and alerts.

---

## 1. Math Between Series

PromQL allows you to perform mathematical operations between different time series. This is useful for calculating ratios, percentages, or combining metrics for deeper insights.

### **Supported Operators**

| Operator | Description           | Example Usage                                 |
|----------|-----------------------|-----------------------------------------------|
| `+`      | Addition              | `metric_a + metric_b`                        |
| `-`      | Subtraction           | `metric_a - metric_b`                        |
| `*`      | Multiplication        | `metric_a * metric_b`                        |
| `/`      | Division              | `metric_a / metric_b`                        |
| `%`      | Modulo (remainder)    | `metric_a % metric_b`                        |
| `^`      | Exponentiation        | `metric_a ^ 2`                               |
| `==`     | Equal                 | `metric_a == metric_b`                       |
| `!=`     | Not equal             | `metric_a != metric_b`                       |
| `>`      | Greater than          | `metric_a > metric_b`                        |
| `<`      | Less than             | `metric_a < metric_b`                        |
| `>=`     | Greater or equal      | `metric_a >= metric_b`                       |
| `<=`     | Less or equal         | `metric_a <= metric_b`                       |

### **Examples**

- **Addition:**  
  ```promql
  http_requests_total{job="api"} + http_requests_total{job="frontend"}
  ```
  > Sums requests from two jobs.

- **Add all equally-labelled series from both sides:**  
  ```promql
  metric_a + metric_b
  ```
  > Adds all series from `metric_a` and `metric_b` that have exactly the same set of labels.

- **Add series, matching only on the instance and job labels:**  
  ```promql
  metric_a + on(instance, job) metric_b
  ```
  > Adds series from `metric_a` and `metric_b` where the `instance` and `job` labels match, ignoring other labels.

- **Add series, ignoring the instance and job labels for matching:**  
  ```promql
  metric_a + ignoring(instance, job) metric_b
  ```
  > Adds series from `metric_a` and `metric_b` matching on all labels except `instance` and `job`.

- **Explicitly allow many-to-one matching:**  
  ```promql
  metric_a + on(instance) group_left metric_b
  ```
  > Allows matching multiple `metric_a` series to a single `metric_b` series based on the `instance` label.

- **Include the version label from "one" (right) side in the result:**  
  ```promql
  metric_a + on(instance) group_left(version) metric_b
  ```
  > Adds series matching on `instance` and includes the `version` label from `metric_b` in the result.

- **Division (Ratio):**  
  ```promql
  sum(rate(http_requests_total{status="500"}[5m])) 
    / sum(rate(http_requests_total[5m]))
  ```
  > Calculates the error rate ratio.

- **Multiplication (Scaling):**  
  ```promql
  rate(cpu_seconds_total[5m]) * 100
  ```
  > Converts CPU usage to percentage.

- **Comparison:**  
  ```promql
  http_requests_total > 1000
  ```
  > Returns series where the value is greater than 1000.

**Note:**  
When combining series, PromQL matches series by their labels. Use `on()` or `ignoring()` to control label matching if needed.

---

---

## 2. Set Operations

PromQL supports set operations that allow you to compare and filter time series based on their labels. These operations are useful for finding overlaps, differences, or intersections between sets of series.

### **Supported Set Operators**

| Operator | Description                                      | Example Usage                                 |
|----------|--------------------------------------------------|-----------------------------------------------|
| `and`    | Intersection: series present in both sides       | `up and http_requests_total`                  |
| `or`     | Union: series present in either side             | `up or http_requests_total`                   |
| `unless` | Exclusion: series on left, not present on right  | `up unless http_requests_total`               |

### **Examples**

- **Intersection (`and`):**  
  ```promql
  up and http_requests_total
  ```
  > Returns series that exist in both `up` and `http_requests_total` with matching labels.

- **Union (`or`):**  
  ```promql
  up or http_requests_total
  ```
  > Returns all series from both `up` and `http_requests_total`.

- **Exclusion (`unless`):**  
  ```promql
  up unless http_requests_total
  ```
  > Returns series from `up` that do not have a matching series in `http_requests_total`.

**Note:**  
Set operators match series by their labels. You can use `on()` or `ignoring()` to control which labels are used for matching, similar to arithmetic




## 3. Subqueries

Subqueries in PromQL allow you to run a query over the result of another query, enabling more advanced and flexible analysis. They are especially useful for calculating trends, aggregations, or transformations over dynamically generated data.

### **Syntax**

```promql
<query>[<range>:<resolution>]
```

- `<range>`: The lookback window for each evaluation point. It defines how much historical data is considered for each calculation. For example, `[5m]` means each data point is calculated using the last 5 minutes of data.
- `<resolution>`: The step between each evaluation point within the subquery range. For example, `:1m` means the subquery will return a value every 1 minute within the specified range.

**Example:**  
```promql
rate(http_requests_total[5m])[30m:5m]
```
- Here, `rate(http_requests_total[5m])` is evaluated every 5 minutes (`:5m`) over the last 30 minutes (`[30m]`), using a 5-minute lookback window for each calculation.

This allows you to control both how much data is used for each calculation and how frequently those calculations are performed within the time window.

### **Examples**

- **Calculate the 5-minute rate of requests, then view its trend over the last 30 minutes at 5-minute intervals:**  
  ```promql
  rate(http_requests_total[5m])[30m:5m]
  ```
  > Returns the 5-minute rate of `http_requests_total` at every 5-minute step, for the last 30 minutes.

- **Aggregate over a subquery:**  
  ```promql
  sum(rate(http_requests_total[1m])[10m:1m])
  ```
  > For each minute in the last 10 minutes, calculates the per-minute rate and then sums across all series.

- **Apply a function to a subquery:**  
  ```promql
  max_over_time(rate(cpu_seconds_total[1m])[15m:1m])
  ```
  > For each minute in the last 15 minutes, finds the maximum rate of CPU usage.

- **Nested subqueries for advanced analysis:**  
  ```promql
  avg_over_time(
    sum(rate(http_requests_total[1m]))[30m:5m]
  [2h:30m])
  ```
  > Calculates the average of the sum of 1-minute request rates, sampled every 30 minutes, over the last 2 hours.

**Note:**  
Subqueries are powerful for dashboarding and alerting, but can be computationally expensive. Use them thoughtfully to

## 4. Manipulating Labels

PromQL provides several operators and functions to manipulate labels in your queries. This is useful for renaming, grouping, or filtering series based on label values, and for shaping your query results for dashboards and alerts.

### **Label Modifiers in Binary Operations**

- **on()**  
  Specifies which labels must match for the operation to be applied.
  ```promql
  metric_a + on(instance, job) metric_b
  ```
  > Only series with matching `instance` and `job` labels are combined.

- **ignoring()**  
  Specifies which labels to ignore when matching series.
  ```promql
  metric_a + ignoring(instance) metric_b
  ```
  > Combines series, ignoring the `instance` label.

- **group_left / group_right**  
  Allows many-to-one or one-to-many matching and can include extra labels from one side.
  ```promql
  metric_a + on(instance) group_left(version) metric_b
  ```
  > Combines series matching on `instance` and adds the `version` label from `metric_b` to the result.

### **Label Replacement with label_replace()**

The `label_replace()` function lets you create or modify labels using regular expressions.

```promql
label_replace(up, "new_label", "$1", "instance", "(.*):.*")
```
> Extracts the value before the colon in the `instance` label and stores it in a new label called `new_label`.

### **Changing Label Sets with aggregation**

Aggregation operators like `sum`, `avg`, `min`, `max`, etc., allow you to group or remove labels:

```promql
sum(rate(http_requests_total[5m])) by (job)
```
> Sums the rate of requests, grouping by the `job` label.

```promql
sum(rate(http_requests_total[5m])) without (instance)
```
> Sums the rate of requests, removing the `instance` label from the result.

### **Dropping or Keeping Labels**

- **by()**: Keeps only the listed labels.
- **without()**: Drops the listed labels.

**Examples:**
```promql
avg by (job, datacenter) (rate(http_requests_total[5m]))
```
> Averages the rate, keeping only `job` and `datacenter` labels.

```promql
sum without (instance) (rate(http_requests_total[5m]))
```
> Sums the rate, dropping the `instance` label.

---

**Tip:**  
Manipulating labels is essential for shaping your PromQL results for meaningful dashboards, alerts,

## 5. Dealing with Missing Data

In real-world scenarios, time series data may have gaps due to network issues, service downtime, or scraping failures. PromQL provides several strategies and functions to help you handle missing data gracefully.

### **Functions for Handling Missing Data**

- **absent()**  
  Returns a series with value 1 if the input series does not exist, or no output if it does.
  ```promql
  absent(up{job="api"})
  ```
  > Returns 1 if there are no `up` series for the `api` job.

- **absent_over_time()**  
  Checks for the absence of data over a range vector.
  ```promql
  absent_over_time(up{job="api"}[5m])
  ```
  > Returns 1 if there was no data for `up{job="api"}` in the last 5 minutes.

- **or operator**  
  You can use the `or` operator to provide fallback values when data is missing.
  ```promql
  (up == 1) or on(instance) vector(0)
  ```
  > Returns 1 if `up` is present, otherwise 0 for missing instances.

### **Filling Gaps and Smoothing Data**

- **increase() and rate()**  
  These functions interpolate missing points within the range window, smoothing over short gaps.
  ```promql
  rate(http_requests_total[5m])
  ```
  > Handles short gaps by interpolating between available points.

- **last_over_time()**  
  Returns the last value in a range, which can be used to fill gaps in dashboards.
  ```promql
  last_over_time(up[10m])
  ```
  > Shows the last known value in the last 10 minutes.

### **Best Practices**

- Use `absent()` and `absent_over_time()` for alerting on missing data.
- Use the `or` operator to provide default or fallback values in dashboards.
- For visualization, consider using functions like `last_over_time()` to avoid gaps in graphs.

**Tip:**  
Always consider the impact of missing data on your alerts and dashboards, and use these tools to make your monitoring more



## 6. Recording Rules

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

## 7. Alerting Rules

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

## 8. Combining Metrics

You can combine different metrics using arithmetic and logical operators.

### **Example: Error Rate Percentage**
```promql
100 * sum(rate(http_requests_total{status="500"}[5m])) 
  / sum(rate(http_requests_total[5m]))
```
> Calculates the percentage of requests that resulted in errors.

---

## 9. Useful Dashboard Patterns

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

## 10. Tips and Best Practices

- Use `rate()` for counters, not `gauge` metrics.
- Always specify a range for rate functions (e.g., `[5m]`).
- Use `by` clauses to group results for dashboards.
- Precompute expensive queries with recording rules.
- Use subqueries for advanced time-based analysis.

---

- [Prometheus Querying Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [PromLabs PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
- [Awesome Prometheus Alerts](https://samber.github.io/awesome-prometheus-alerts/)