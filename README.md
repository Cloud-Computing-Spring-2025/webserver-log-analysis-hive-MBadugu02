# Web Server Log Analysis Using Apache Hive

## Project Overview
The purpose of this assignment is to implement and execute a Hive-based analytical process for analyzing web server logs. You will create and query a Hive table to process a dataset of web server logs in CSV format and extract meaningful insights about website traffic patterns.

Our task is to analyze the web server logs using Apache Hive and perform the following tasks:

Count Total Web Requests: Calculate the total number of requests processed.

Analyze Status Codes: Determine the frequency of HTTP status codes (e.g., 200, 404, 500).

Identify Most Visited Pages: Extract the top 3 most visited URLs.

Traffic Source Analysis: Identify the most common user agents (browsers).

Detect Suspicious Activity: Identify IP addresses with more than 3 failed requests (status 404 or 500).

Analyze Traffic Trends: Calculate the number of requests per minute to observe traffic patterns.

Implement Partitioning: Use partitioning by status code to optimize query performance.

---

## Implementation Approach
The following HiveQL queries were used to perform the analysis tasks:

### 1. Count Total Web Requests
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/total_requests'
SELECT COUNT(*) FROM hue__tmp_web_server_logs;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/total_requests total_requests.txt
```

### 2. Export Status Code Analysis
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/status_codes'
SELECT status, COUNT(*) 
FROM hue__tmp_web_server_logs
GROUP BY status 
ORDER BY status;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/status_codes status_codes.txt
```

### 3. Export Most Visited Pages
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/most_visited_pages'
SELECT url, COUNT(*) AS visit_count
FROM hue__tmp_web_server_logs
GROUP BY url
ORDER BY visit_count DESC
LIMIT 3;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/most_visited_pages most_visited_pages.txt
```

### 4. Export Traffic Source Analysis
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/user_agents'
SELECT user_agent, COUNT(*) AS user_count
FROM hue__tmp_web_server_logs
GROUP BY user_agent
ORDER BY user_count DESC;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/user_agents user_agents.txt
```

### 5. Export Suspicious IP Addresses
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/suspicious_ips'
SELECT ip, COUNT(*) AS failed_requests
FROM hue__tmp_web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3
ORDER BY failed_requests DESC;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/suspicious_ips suspicious_ips.txt
```

### 6. Export Suspicious IP Addresses
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/traffic_trends'
SELECT DATE_FORMAT(`timestamp`, 'yyyy-MM-dd HH:mm') AS request_minute, COUNT(*) AS request_count
FROM hue__tmp_web_server_logs
GROUP BY DATE_FORMAT(`timestamp`, 'yyyy-MM-dd HH:mm')
ORDER BY request_minute;
```
Retrieve the output in bash
```bash
hdfs dfs -getmerge /user/hue/output/traffic_trends traffic_trends.txt
```

## Execution Steps
Now run the following commands:

```bash
hdfs dfs -getmerge /user/hue/output/total_requests total_requests.txt
```

```bash
hdfs dfs -getmerge /user/hue/output/status_codes status_codes.txt
```

```bash
hdfs dfs -getmerge /user/hue/output/most_visited_pages most_visited_pages.txt
```

```bash
hdfs dfs -getmerge /user/hue/output/user_agents user_agents.txt
```

```bash
hdfs dfs -getmerge /user/hue/output/suspicious_ips suspicious_ips.txt
```
```bash
hdfs dfs -getmerge /user/hue/output/traffic_trends traffic_trends.txt
```

Now exit and run these commands to save the output files in a new directory called output. If this folder doesn't exit, create one.

```bash
docker cp resourcemanager:/total_requests.txt output/
```

```bash
docker cp resourcemanager:/status_codes.txt output/
```

```bash
docker cp resourcemanager:/most_visited_pages.txt output/
```

```bash
docker cp resourcemanager:/user_agents.txt output/
```

```bash
docker cp resourcemanager:/suspicious_ips.txt output/
```

```bash
docker cp resourcemanager:/traffic_trends.txt output/
```

## Challenges Faced
#### HDFS Permissions:
Issue: Unable to write to HDFS from Hue due to restricted permissions.

Solution: Used Hue's File Browser to create directories and manually upload files.

#### Query Performance:
Issue: Queries were slow on large datasets.

Solution:  Improved efficiency by implementing partitioning based on status.

#### Exporting Results:
Issue: The INSERT OVERWRITE DIRECTORY command did not work in Hue.

Solution: Stored results in Hive tables and used Hue’s Download Results feature for exporting data.

## Sample Input
ip,timestamp,url,status,user_agent 192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0 192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0 192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1 192.168.1.10,2024-02-01 10:18:00,/home,500,Mozilla/5.0 192.168.1.15,2024-02-01 10:19:00,/products,404,Chrome/90.0

## Sample output
#### Total Web Requests:

Total Requests: 100

#### Status Code Analysis:

200: 80

404: 10

500: 10

#### Most Visited Pages:

/home: 50

/products: 30

/checkout: 20

#### Traffic Source Analysis:

Mozilla/5.0: 60

Chrome/90.0: 30

Safari/13.1: 10

#### Suspicious IP Addresses:

192.168.1.10: 5 failed requests

192.168.1.15: 4 failed requests

#### Traffic Trend Over Time:

2024-02-01 10:15: 5 requests

2024-02-01 10:16: 7 requests