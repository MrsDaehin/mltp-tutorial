# PromQL Workshop – Part 2: Cheat Sheet Deep Dive

This workshop is based on the [PromLabs PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/). Each section below introduces a PromQL concept or function, provides a brief explanation, and includes practical examples.

---

## 1. **Selectors**

### 1.1. **Instant Vector Selector**
```promql
http_requests_total
```
Returns the latest value for each time series of the metric.

### 1.2. **Range Vector Selector**
```promql
http_requests_total[5m]
```
Returns all values in the last 5 minutes for each time series.

### 1.3. **Label Matchers**
```promql
http_requests_total{job="api", status!="500"}
```
Selects time series where `job` is "api" and `status` is not "500".

---

## 2. **Operators**

### 2.1. **Arithmetic Operators**
```promql
rate(http_requests_total[5m]) * 100
```
Multiplies the per-second rate by 100.

### 2.2. **Comparison Operators**
```promql
up == 0
```
Returns 1 for targets that are down, 0 otherwise.

### 2.3. **Boolean Modifier**
```promql
up == bool 0
```
Returns 1 for targets that are down, ignores the value otherwise.

### 2.4. **Set Operators**
```promql
up or http_requests_total
up and http_requests_total
up unless http_requests_total
```
Combines or filters time series sets.

---

## 3. **Aggregation Operators**

### 3.1. **sum()**
```promql
sum(http_requests_total)
```
Sums all values.

### 3.2. **sum by (label)**
```promql
sum by (job) (http_requests_total)
```
Sums values grouped by `job`.

### 3.3. **count()**
```promql
count(up)
```
Counts the number of time series.

### 3.4. **avg()**
```promql
avg(http_requests_total)
```
Averages all values.

### 3.5. **min() / max()**
```promql
min(up)
max(up)
```
Finds the minimum or maximum value.

### 3.6. **stddev() / stdvar()**
```promql
stddev(http_requests_total)
stdvar(http_requests_total)
```
Standard deviation and variance.

### 3.7. **topk() / bottomk()**
```promql
topk(3, http_requests_total)
bottomk(3, http_requests_total)
```
Top or bottom 3 time series by value.

---

## 4. Functions: Rates of Increase for Counters

Prometheus counters (like `http_requests_total`) only ever increase (except when they reset, e.g., on process restart). PromQL provides several functions to analyze how fast these counters are increasing over time. The most common are `rate()`, `irate()`, and `increase()`.

---

### **rate()**

- **Purpose:** Calculates the average per-second rate of increase of a counter over a specified time range.
- **Usage:** Best for dashboards and alerting, as it smooths out short-term fluctuations.
- **Example:**
  ```promql
  rate(http_requests_total[5m])
  ```
  > Returns the average number of requests per second over the last 5 minutes.

---

### **irate()**

- **Purpose:** Calculates the "instantaneous" per-second rate of increase, using only the two most recent data points in the range.
- **Usage:** Best for detecting sudden spikes or drops; more sensitive to short-term changes.
- **Example:**
  ```promql
  irate(http_requests_total[5m])
  ```
  > Returns the per-second rate based on the last two samples in the last 5 minutes.

---

### **increase()**

- **Purpose:** Calculates the total increase in the counter over the specified time range.
- **Usage:** Useful for totals over a period (e.g., "How many requests in the last hour?").
- **Example:**
  ```promql
  increase(http_requests_total[1h])
  ```
  > Returns the total number of requests that occurred in the last hour.

---

### **Comparison Table**

| Function   | What it shows                        | Use case example                        |
|------------|--------------------------------------|-----------------------------------------|
| `rate()`   | Average per-second rate (smoothed)   | Dashboards, alerting, trends            |
| `irate()`  | Instantaneous per-second rate        | Detecting spikes, troubleshooting       |
| `increase()`| Total increase over time range      | Counting events in a period             |

---

### **Practical Examples**

1. **Show the average request rate per job:**
   ```promql
   sum by (job) (rate(http_requests_total[5m]))
   ```

2. **Detect sudden spikes in error responses:**
   ```promql
   irate(http_requests_total{status="500"}[1m])
   ```

3. **Count total requests in the last 10 minutes:**
   ```promql
   increase(http_requests_total[10m])
   ```

---

**Tip:**  
- Use `rate()` for most monitoring and alerting scenarios.
- Use `irate()` when you need to see immediate changes.
- Use `increase()` when you want a total count over a period.

---

## 5. Functions: Changes in Gauges

Gauges are metrics that can go up and down (e.g., memory usage, temperature). PromQL provides functions to analyze how these values change over time.

---

### **delta()**

- **Purpose:** Calculates the difference between the first and last value of a gauge within a specified time range.
- **Usage:** Useful for understanding the net change over a period.
- **Example:**
  ```promql
  delta(memory_usage_bytes[10m])
  ```
  > Returns the net change in memory usage over the last 10 minutes.

---

### **deriv()**

- **Purpose:** Estimates the per-second derivative (rate of change) of a gauge, using linear regression.
- **Usage:** Useful for identifying trends or the speed of change.
- **Example:**
  ```promql
  deriv(memory_usage_bytes[5m])
  ```
  > Returns the per-second rate of change in memory usage over the last 5 minutes.

---

### **idelta()**

- **Purpose:** Calculates the difference between the last two samples in the range.
- **Usage:** Useful for detecting sudden jumps or drops.
- **Example:**
  ```promql
  idelta(memory_usage_bytes[5m])
  ```
  > Returns the difference between the last two memory usage samples in the last 5 minutes.

---

### **Practical Examples**

1. **Show the net change in CPU usage over the last 15 minutes:**
   ```promql
   delta(process_cpu_seconds_total[15m])
   ```

2. **Show the rate of change in active connections:**
   ```promql
   deriv(active_connections[10m])
   ```

3. **Detect sudden drops in available memory:**
   ```promql
   idelta(node_memory_MemAvailable_bytes[2m])
   ```

---
## 6. Functions: Histogram Quantile

Histograms are used to observe the distribution of values (like request durations or response sizes). Prometheus stores histogram data as multiple time series with bucket boundaries. The `histogram_quantile()` function estimates quantiles (e.g., median, 95th percentile) from these histograms.

---

### **histogram_quantile()**

- **Purpose:** Calculates an estimated quantile (e.g., 0.5 for median, 0.95 for 95th percentile) from histogram buckets.
- **Usage:** Useful for understanding latency or size distributions.
- **Syntax:**
  ```promql
  histogram_quantile(φ, sum(rate(<histogram_metric>_bucket[range])) by (le))
  ```
  Where `φ` is the desired quantile (e.g., 0.95 for the 95th percentile).

---

### **Examples**

1. **Calculate the 95th percentile request duration:**
   ```promql
   histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
   ```
   > Estimates the 95th percentile of HTTP request durations over the last 5 minutes.

2. **Calculate the median (50th percentile) response size:**
   ```promql
   histogram_quantile(0.5, sum(rate(http_response_size_bytes_bucket[5m])) by (le))
   ```
   > Estimates the median response size over the last 5 minutes.

3. **Calculate the 99th percentile latency per job:**
   ```promql
   histogram_quantile(0.99, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m])))
   ```
   > Estimates the 99th percentile latency for each job.

---

**Tip:**  
- Always use the `_bucket` metric for `histogram_quantile()`.
- Use `sum(rate(...)) by (le)` to aggregate across all relevant labels except `le`.

---
## 7. Aggregation Over Multiple Series

Aggregation in PromQL allows you to combine multiple time series into a single series or a smaller set of series, using functions like `sum`, `avg`, `min`, `max`, `count`, and more. Aggregation is essential for summarizing data across different dimensions (labels), such as combining metrics from all instances of a service or across all endpoints.

---

### **How Aggregation Works**

- **Without grouping:** Aggregates all matching series into a single value.
- **With `by` clause:** Groups series by the specified label(s) and aggregates within each group.

---

### **Examples**

1. **Sum all HTTP requests across all jobs:**
   ```promql
   sum(http_requests_total)
   ```
   > Returns the total number of HTTP requests across all jobs.

2. **Sum HTTP requests per job:**
   ```promql
   sum by (job) (http_requests_total)
   ```
   > Returns the total number of HTTP requests for each job.

3. **Average CPU usage per instance:**
   ```promql
   avg by (instance) (process_cpu_seconds_total)
   ```
   > Returns the average CPU usage for each instance.

4. **Count the number of up targets per job:**
   ```promql
   count by (job) (up)
   ```
   > Returns the number of targets that are up for each job.

---
### **Comparison Table: Aggregation Over Multiple Series**

| Aggregation Function | Description                                 | Example Usage                                      | Typical Use Case                         |
|----------------------|---------------------------------------------|----------------------------------------------------|------------------------------------------|
| `sum`                | Adds up all values                          | `sum(http_requests_total)`                         | Total requests across all series         |
| `avg`                | Calculates the average                      | `avg by (job) (process_cpu_seconds_total)`         | Average CPU usage per job                |
| `min`                | Finds the minimum value                     | `min by (instance) (up)`                           | Find the lowest value per instance       |
| `max`                | Finds the maximum value                     | `max by (job) (http_requests_total)`               | Find the highest value per job           |
| `count`              | Counts the number of series                 | `count by (job) (up)`                              | Number of up targets per job             |
| `stddev`             | Standard deviation                          | `stddev by (job) (process_cpu_seconds_total)`      | Variability of CPU usage per job         |
| `stdvar`             | Variance                                    | `stdvar by (job) (process_cpu_seconds_total)`      | Variance of CPU usage per job            |
| `topk`               | Top K series by value                       | `topk(3, sum by (job) (http_requests_total))`      | Top 3 jobs by request count              |
| `bottomk`            | Bottom K series by value                    | `bottomk(3, sum by (job) (http_requests_total))`   | Bottom 3 jobs by request count           |

---

**Tip:**  
- Use the `by` clause to control the grouping of your aggregation by one or more labels.
- You can use multiple labels in the `by` clause for multi-dimensional grouping, e.g., `sum by (job,instance) (...)`.
- Omitting the `by` clause aggregates across all series into a single value.

## 8. Aggregation Over Time

Aggregation over time in PromQL uses functions that summarize or transform the values of a single time series within a specified time window. These functions are useful for smoothing data, detecting trends, or extracting statistical information from time series.

---

### **Common Over Time Functions**

| Function                | Description                                      | Example Usage                                         | Typical Use Case                                 |
|-------------------------|--------------------------------------------------|-------------------------------------------------------|--------------------------------------------------|
| `sum_over_time()`       | Sums all values in the range                     | `sum_over_time(http_requests_total[10m])`             | Total requests in the last 10 minutes            |
| `avg_over_time()`       | Averages all values in the range                 | `avg_over_time(process_cpu_seconds_total[5m])`        | Average CPU usage in the last 5 minutes          |
| `min_over_time()`       | Finds the minimum value in the range             | `min_over_time(node_memory_MemAvailable_bytes[1h])`   | Lowest available memory in the last hour         |
| `max_over_time()`       | Finds the maximum value in the range             | `max_over_time(temperature_celsius[30m])`             | Highest temperature in the last 30 minutes       |
| `count_over_time()`     | Counts the number of values in the range         | `count_over_time(up[1h])`                             | Number of samples in the last hour               |
| `quantile_over_time()`  | Calculates a quantile (e.g., median) over range  | `quantile_over_time(0.5, temperature_celsius[30m])`   | Median temperature in the last 30 minutes        |
| `stddev_over_time()`    | Standard deviation over the range                | `stddev_over_time(process_cpu_seconds_total[5m])`     | Variability of CPU usage in the last 5 minutes   |
| `stdvar_over_time()`    | Variance over the range                          | `stdvar_over_time(process_cpu_seconds_total[5m])`     | Variance of CPU usage in the last 5 minutes      |

---

### **Examples**

1. **Sum of requests over the last 10 minutes:**
   ```promql
   sum_over_time(http_requests_total[10m])
   ```
   > Returns the total number of requests in the last 10 minutes for each series.

2. **Average CPU usage over the last 5 minutes:**
   ```promql
   avg_over_time(process_cpu_seconds_total[5m])
   ```
   > Returns the average CPU usage in the last 5 minutes for each series.

3. **Minimum available memory over the last hour:**
   ```promql
   min_over_time(node_memory_MemAvailable_bytes[1h])
   ```
   > Returns the lowest available memory value in the last hour for each series.

4. **Median value of a gauge over the last 30 minutes:**
   ```promql
   quantile_over_time(0.5, temperature_celsius[30m])
   ```
   > Returns the median temperature in the last 30 minutes for each series.

---

**Tip:**  
- Over time functions always require a range vector selector (e.g., `[5m]`).
- These functions operate on each time series independently.

[Continue to Part 3: Advanced Patterns and Use Cases →](./workshop_part3.md)