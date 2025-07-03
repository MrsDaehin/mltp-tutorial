# LogQL for Beginners: Workshop

Welcome to the **LogQL for Beginners** workshop! This guide will introduce you to the basics of LogQL, the query language for Grafana Loki, and help you get started with querying and analyzing your logs.

---

## Table of Contents

1. [What is LogQL?](#what-is-logql)
2. [Types of LogQL Queries](#types-of-logql-queries)
3. [Log Stream Selectors](#log-stream-selectors)
4. [Filter Expressions](#filter-expressions)
5. [Range and Instance Vectors](#range-and-instance-vectors)
6. [Aggregate Functions and Grouping](#aggregate-functions-and-grouping)
7. [Operators in LogQL](#operators-in-logql)
    - Arithmetic Operators
    - Logical and Set Operators
    - Comparison Operators
    - Pattern Match Filter Operators
8. [Advanced Matching: on, ignoring, group_left, group_right](#advanced-matching)
9. [Comments in LogQL](#comments-in-logql)
10. [Handling Pipeline Errors](#handling-pipeline-errors)
11. [Useful Functions](#useful-functions)
12. [Best Practices & Tips](#best-practices--tips)
13. [Further Resources](#further-resources)

---

## What is LogQL?

**LogQL** is the query language for [Grafana Loki](https://grafana.com/oss/loki/). It is inspired by PromQL and allows you to query, filter, and aggregate logs efficiently.

---

## Types of LogQL Queries

There are two main types of LogQL queries:

- **Log queries**: Return the contents of log lines as streams.
- **Metric queries**: Convert logs into value matrices for visualization and alerting.

A LogQL query consists of:
- The log stream selector
- Filter expression(s)

---

## Log Stream Selectors

Selectors filter logs by labels (or tags). Supported operators:
- `=` : equals
- `!=` : not equals
- `=~` : regex matches
- `!~` : regex does not match

**Examples:**
```logql
{job="systemd-journal"}
{unit="ssh.service"}
{job="systemd-journal",unit="cron.service"}
{job=~"qryn/systemd-journal|systemd-journal"}
```

---

## Filter Expressions

Filter expressions test text within log line streams. Supported operators:
- `|=` : line contains string
- `!=` : line does not contain string
- `|~` : line matches regex
- `!~` : line does not match regex

**Examples:**
```logql
{job="systemd-journal"} |= "error"
{job="systemd-journal"} != "error"
{job="systemd-journal"} |~ "error|info"
{job="systemd-journal"} !~ "error|info"
{job="systemd-journal"} |= "error" != "info"
{job="systemd-journal"} |~ "Invalid user (default|qryn)"
{job="systemd-journal"} |~ "status [45]03"
```

---

## Range and Instance Vectors

To visualize or aggregate logs, convert them to vectors:
- `count_over_time` : total count of log lines for time range
- `rate` : entries per second
- `bytes_over_time` : number of bytes in each log stream in the range
- `bytes_rate` : bytes per second

**Examples:**
```logql
count_over_time({job="systemd-journal"}[1m])
rate({job="systemd-journal"}[1m])
count_over_time({job="systemd-journal"} |= "error" [1h])
```

---

## Aggregate Functions and Grouping

Aggregate functions convert a range vector result into a single instance vector:
- `sum`, `min`, `max`, `avg`, `stddev`, `stdvar`, `count`

**Examples:**
```logql
sum(count_over_time({job="systemd-journal"}[1m]))
min(count_over_time({job="systemd-journal"}[1m]))
max(count_over_time({job="systemd-journal"}[1m]))
```

You can group by label(s):
```logql
sum(count_over_time({job="systemd-journal"}[1m])) by (unit)
sum(count_over_time({job=~"qryn/systemd-journal|systemd-journal"}[1m])) by (job)
sum(count_over_time({job=~"qryn/systemd-journal|systemd-journal"}[1m])) by (job,unit)
```

---

## Operators in LogQL

### Comparison Operators

Comparison operators test numeric values in scalars and vectors:
- `==` (equality)
- `!=` (inequality)
- `>` (greater than)
- `>=` (greater than or equal to)
- `<` (less than)
- `<=` (less than or equal to)

**Examples:**
```logql
sum(count_over_time({job="systemd-journal"}[1m])) > 4
sum(count_over_time({job="systemd-journal"}[1m])) <= 1
```

### Logical and Set Operators

Logical operators can be used to add filter conditions:
- `and` : Both sides must be true
- `or` : One on either side must be true
- `unless` : complement

**Examples:**
```logql
{job="systemd-journal"} | json | mem > 20 and mem < 30
{job="systemd-journal"} | json | mem < 30 or mem > 80
```

### Arithmetic Operators

- `+` (addition)
- `-` (subtraction)
- `*` (multiplication)
- `/` (division)
- `%` (modulo)
- `^` (power)

**Examples:**
```logql
1 + 1
sum(rate({app="foo"}[1m])) * 2
sum(rate({app="foo", level="warn"}[1m])) / sum(rate({app="foo", level="error"}[1m]))
```

### Pattern Match Filter Operators

- `|>` (line match pattern)
- `!>` (line match not pattern)

**Example:**
```logql
{service_name="distributor"} |> `<_> caller=http.go:194 level=debug <_> msg="POST /push.v1.PusherService/Push <_>`
```

---

## Advanced Matching

### on and ignoring

- `ignoring(<labels>)`: Ignores specified labels during matching.
- `on(<labels>)`: Only considers specified labels during matching.

**Example:**

```logql
max by(machine) (count_over_time({app="foo"}[1m])) > bool ignoring(machine) avg(count_over_time({app="foo"}[1m]))
```

### group_left and group_right

Used for many-to-one and one-to-many vector matches.

**Example:**

```logql
sum by (app, status) (
  rate({job="http-server"} | json [5m])
)
/ on (app) group_left
sum by (app) (
  rate({job="http-server"} | json [5m])
)
```

---

## Comments in LogQL

Use `#` to add comments:

```logql
{app="foo"} # This is a comment
```

Multi-line example:

```logql
{app="foo"}
  | json
  # this line will be ignored
  | bar="baz" # this checks if bar = "baz"
```

---

## Handling Pipeline Errors

If a pipeline stage fails (e.g., invalid JSON), Loki adds a `__error__` label. You can filter out errors:

```logql
{cluster="ops-tools1",container="ingress-nginx"}
  | json
  | __error__ != "JSONParserErr"
```

Or remove all errors:

```logql
| __error__ = ""
```

---

## Useful Functions

### label_replace

Replace or add labels using regex:

```logql
label_replace(rate({job="api-server",service="a:c"} |= "err" [1m]), "foo", "$1", "service", "(.*):.*")
```

---

## Best Practices & Tips

- Start with simple selectors and add filters incrementally.
- Use label selectors to reduce the amount of data scanned.
- Prefer `|=` and `!=` for simple string matches, and `|~`/`!~` for regex.
- Use aggregation and grouping to summarize large log volumes.
- Combine multiple filter expressions for precise results.
- Use time ranges (`[1m]`, `[5m]`, etc.) to control the window of analysis.
- Test queries in small time windows before scaling up.

---

## Further Resources

- [Official LogQL Documentation](https://grafana.com/docs/loki/latest/logql/)
- [LogQL Log Queries](https://grafana.com/docs/loki/latest/query/log_queries/)
- [LogQL Metric Queries](https://grafana.com/docs/loki/latest/query/metric_queries/)
- [Grafana Loki Tutorials](https://grafana.com/tutorials/)

---

Happy querying! 🚀

---
