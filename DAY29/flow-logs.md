# 1. What is VPC Flow Log

VPC Flow Logs is an AWS feature used to capture network traffic information for a **VPC**, **subnet**, or **Elastic Network Interface (ENI)**. It helps us understand which traffic is flowing, which traffic is allowed, which traffic is rejected, and between which source and destination IPs the communication happened. ([AWS Documentation][1])

Simple explanation for students:

**VPC Flow Logs = network activity history for your AWS network.**

It does not show full packet content. It shows connection-level metadata such as:

* source IP
* destination IP
* source port
* destination port
* protocol
* ACCEPT or REJECT
* packet count
* byte count
* interface details
* timestamps ([AWS Documentation][2])

---

# 2. Why do we use VPC Flow Logs

We use VPC Flow Logs for:

* troubleshooting connectivity issues
* checking whether traffic is accepted or rejected
* validating Security Group and NACL behavior
* identifying unexpected IP communication
* monitoring internet exposure
* investigating suspicious traffic
* understanding subnet or instance communication patterns
* audit and forensic review after incidents ([AWS Documentation][1])

Classroom line:

**When a server is not reachable, VPC Flow Logs help us answer: is traffic coming, where is it going, and is AWS accepting or rejecting it?** ([AWS Documentation][1])

---

# 3. What VPC Flow Logs can and cannot do

VPC Flow Logs can tell you **traffic metadata**. For example, whether traffic from `10.0.1.10` to `10.0.2.25` on port `443` was accepted or rejected. They are useful for traffic analysis and troubleshooting. ([AWS Documentation][2])

VPC Flow Logs do **not** capture the actual packet payload, request body, SQL query text, HTTP response body, or application logs. So they cannot replace web server logs, OS logs, or packet capture tools. ([AWS Documentation][1])

---

# 4. Where can we enable Flow Logs

AWS allows flow logs at these levels:

* **VPC**
* **Subnet**
* **Network Interface (ENI)** ([AWS Documentation][3])

How to explain:

* **VPC-level flow log** = captures broad traffic view for the full VPC
* **Subnet-level flow log** = captures traffic related to that subnet
* **ENI-level flow log** = captures traffic for one specific resource interface, like one EC2 instance ENI ([AWS Documentation][3])

---

# 5. Why store VPC Flow Logs in CloudWatch Logs

When flow logs are sent to **CloudWatch Logs**, you can:

* view logs quickly in AWS console
* search raw records
* run **CloudWatch Logs Insights** queries
* build operational troubleshooting workflows
* keep logs in centralized log groups
* create dashboards and alarms around log-derived insights if needed ([AWS Documentation][4])

For live demos and training, **CloudWatch Logs is usually the easiest option**, because students can enable logs and query them from the console immediately. ([AWS Documentation][4])

---

# 6. High-level architecture flow

The practical flow is:

1. Traffic happens in your VPC
2. AWS Flow Logs service captures metadata
3. Flow log records are delivered to a CloudWatch Log Group
4. You open CloudWatch Logs Insights
5. You query the traffic records and analyze accepted/rejected flows ([AWS Documentation][1])

Teaching line:

**Application logs tell what the app did. VPC Flow Logs tell what the network did.**

---

# 7. Important prerequisite for CloudWatch destination

To publish VPC Flow Logs to **CloudWatch Logs**, AWS requires permissions through an **IAM role** that allows the flow logs service to create log streams and put log events into the log group. AWS documents a dedicated IAM role flow for this use case. ([AWS Documentation][5])

So before creating the flow log, you need either:

* an **existing IAM service role** for VPC Flow Logs, or
* let AWS help create a new one during setup in supported console flow ([AWS Documentation][6])

---

# 8. Step 1: Create a CloudWatch Log Group

Go to:

**CloudWatch Console → Logs → Log groups → Create log group**

Recommended naming:

* `/aws/vpc/flowlogs/demo-vpc`
* `/aws/vpc/flowlogs/prod-vpc`
* `/aws/vpc/flowlogs/subnet-public-1`

This keeps logs organized clearly by VPC, subnet, or environment. CloudWatch Logs stores logs in **log groups**, and each flow log delivery writes records into log streams under that group. ([AWS Documentation][7])

Good practice:

* choose a clear retention period
* avoid indefinite retention unless you really need it
* keep naming standardized for environments like dev, test, prod

CloudWatch Logs supports retention settings at the log group level. ([AWS Documentation][7])

---

# 9. Step 2: Create IAM role for Flow Logs to publish into CloudWatch

Go to:

**IAM Console → Roles → Create role**

AWS documentation for VPC Flow Logs to CloudWatch shows that the role needs permission to write to CloudWatch Logs, specifically actions such as creating log streams and putting log events. ([AWS Documentation][5])

Typical permission purpose:

* `logs:CreateLogGroup`
* `logs:CreateLogStream`
* `logs:PutLogEvents`
* `logs:DescribeLogGroups`
* `logs:DescribeLogStreams` ([AWS Documentation][5])

Recommended role name:

* `vpc-flowlogs-to-cloudwatch-role`

Teaching note:
This role is not for users. This role is for the **AWS service** so that Flow Logs can publish data into CloudWatch. ([AWS Documentation][5])

---

# 10. Step 3: Create the VPC Flow Log

Go to:

**VPC Console → Your VPCs**
Select the VPC
Choose **Flow logs** tab
Click **Create flow log** ([AWS Documentation][6])

Now fill these important fields:

### Filter

You can choose:

* **All**
* **Accept**
* **Reject**

Meaning:

* **All** = capture both accepted and rejected traffic
* **Accept** = only successful/allowed traffic
* **Reject** = only denied traffic ([AWS Documentation][6])

For training, choose **All**, because students can compare accepted and rejected traffic easily. ([AWS Documentation][6])

### Maximum aggregation interval

This controls how flow records are aggregated before publishing. AWS documents that after the aggregation interval, logs still take additional time to process and publish. Typical delivery to CloudWatch Logs is about **5 minutes**, though AWS states delivery is on a **best-effort basis** and might be delayed. ([AWS Documentation][2])

### Destination

Choose:

* **Send to CloudWatch Logs** ([AWS Documentation][6])

### Destination log group

Choose the log group you created.

### IAM role

Choose the IAM role created for flow log publishing.

### Log record format

Choose either:

* **AWS default format**
* **Custom format** ([AWS Documentation][6])

For beginners, use **AWS default format** first.
For advanced use, use **custom format** so you include exactly the fields you want.

---

# 11. Best recommendation for log format

For your first live class, use **AWS default format** because it is simple and AWS-supported out of the box. ([AWS Documentation][6])

After students understand the basics, show **custom format**. This is very useful because you can include only the fields needed for operations, for example:

* account-id
* interface-id
* srcaddr
* dstaddr
* srcport
* dstport
* protocol
* packets
* bytes
* action
* log-status
* vpc-id
* subnet-id
* instance-id
* tcp-flags
* flow-direction
* traffic-path ([AWS Documentation][2])

Trainer tip:
For real projects, a **custom format** is often better because it reduces confusion and keeps only the operationally useful fields. ([AWS Documentation][2])

---

# 12. Create a practical live environment for demo

You said you do not have anything in hand. Below is the easiest live setup:

* 1 VPC
* 2 subnets
* 2 EC2 instances
* 1 Security Group allowing HTTP/SSH as needed
* 1 VPC Flow Log to CloudWatch
* Generate traffic using curl, ping alternatives where supported, or browser access

Then show how the traffic appears in CloudWatch Logs and how to query it. This lets students see traffic generation and flow log analysis together. ([AWS Documentation][1])

Best practice for teaching:

* create one **accepted traffic** example
* create one **rejected traffic** example
* query both in CloudWatch Logs Insights

---

# 13. How to generate accepted traffic for demo

Accepted traffic examples:

* Browser opens EC2 public web page on port 80
* `curl http://<public-ip>`
* One EC2 instance connects to another on an allowed port
* EC2 accesses internet on 443 if route and SG/NACL allow it

Once traffic happens, wait a few minutes and then query the log group. AWS documents that CloudWatch delivery is typically around 5 minutes but can be delayed. ([AWS Documentation][2])

---

# 14. How to generate rejected traffic for demo

Rejected traffic demo is very useful.

Example:

* EC2 instance tries to connect to another EC2 on a port blocked by Security Group or NACL
* A public request is attempted to a blocked port like `8080` or `22` from an unauthorized source
* Remove or tighten an inbound rule and test again

Then query logs for `action = REJECT`. This is the easiest way to teach how flow logs help in troubleshooting. Flow log records include the **action** field, such as ACCEPT or REJECT. ([AWS Documentation][2])

---

# 15. How to view raw flow logs in CloudWatch

Go to:

**CloudWatch Console → Logs → Log groups → your flow log group**
Open a log stream and you will see space-separated log records. AWS documents this viewing path for flow logs published to CloudWatch Logs. ([AWS Documentation][8])

This is useful to show students the actual record structure before teaching query language.

---

# 16. Understanding a basic flow log record

A flow log record is a line with multiple fields separated by spaces. AWS provides the field structure and examples in its flow log record documentation. ([AWS Documentation][2])

A typical default-style record contains fields like:

* version
* account-id
* interface-id
* srcaddr
* dstaddr
* srcport
* dstport
* protocol
* packets
* bytes
* start
* end
* action
* log-status ([AWS Documentation][2])

Simple meaning:

* **srcaddr** = source IP
* **dstaddr** = destination IP
* **srcport** = source port
* **dstport** = destination port
* **protocol** = TCP/UDP/ICMP number
* **packets** = number of packets
* **bytes** = number of bytes
* **action** = ACCEPT or REJECT
* **log-status** = whether logging was OK or skipped/suboptimal in some cases ([AWS Documentation][2])

---

# 17. Key fields you should explain to students

## srcaddr and dstaddr

These tell you where traffic came from and where it was going. Very useful to identify who talked to whom. ([AWS Documentation][2])

## srcport and dstport

These tell you the source and destination ports. Useful for checking whether traffic targeted port 22, 80, 443, 3306, 5432, 6379, and so on. ([AWS Documentation][2])

## action

This is one of the most important fields.

* `ACCEPT` means traffic was allowed
* `REJECT` means traffic was denied ([AWS Documentation][2])

## packets and bytes

These help estimate traffic volume and whether the communication was meaningful or minimal. ([AWS Documentation][2])

## interface-id

This identifies the ENI that observed the traffic. Useful for mapping traffic to a particular EC2 interface or other AWS-managed interface. ([AWS Documentation][2])

## start and end

These are epoch timestamps showing the flow record time window. ([AWS Documentation][2])

---

# 18. How to query VPC Flow Logs in CloudWatch Logs Insights

Go to:

**CloudWatch Console → Logs Insights**
Select the flow log group
Set the time range
Write query
Run query ([AWS Documentation][9])

AWS documents that CloudWatch Logs Insights supports interactive log analysis and provides a query language with commands like `fields`, `filter`, `sort`, `stats`, `parse`, `limit`, and more. ([AWS Documentation][10])

---

# 19. Important note before running queries

If your log format is the default space-separated flow log record, you usually need to **parse** the message into named fields, unless CloudWatch has already extracted them in a compatible way for your setup. Since query behavior can vary by record format, teaching students with explicit `parse` commands is the safest method. AWS documents both the flow log field layout and the general `parse` capability in Logs Insights. ([AWS Documentation][2])

---

# 20. Query 1: Show recent VPC Flow Log records

```sql
fields @timestamp, @message
| sort @timestamp desc
| limit 20
```

Use this first so students can simply see the newest raw records in the selected log group. This is the easiest starting point in Logs Insights. AWS documents `fields`, `sort`, and `limit` as Logs Insights commands. ([AWS Documentation][10])

---

# 21. Query 2: Parse the default VPC Flow Log format

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| fields @timestamp, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, action, logStatus
| sort @timestamp desc
| limit 20
```

This query assumes a default-style 14-field format. If you use a **custom format**, you must adjust the parse order to match your chosen field order. AWS clearly documents that you can use default or custom record formats, and the fields must be interpreted according to the selected format. ([AWS Documentation][6])

---

# 22. Query 3: Show only rejected traffic

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter action = "REJECT"
| fields @timestamp, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, action
| sort @timestamp desc
| limit 50
```

This is one of the best operational queries because it immediately shows blocked traffic. It is very useful when debugging Security Group or NACL issues. The `action` field in flow logs explicitly indicates ACCEPT or REJECT. ([AWS Documentation][2])

---

# 23. Query 4: Show only accepted traffic

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter action = "ACCEPT"
| fields @timestamp, interfaceId, srcAddr, dstAddr, srcPort, dstPort, packets, bytes
| sort @timestamp desc
| limit 50
```

Use this to show successful traffic reaching or leaving the interface scope covered by the flow log. ([AWS Documentation][2])

---

# 24. Query 5: Find traffic to a specific destination port

Example for SSH:

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter dstPort = "22"
| fields @timestamp, srcAddr, dstAddr, srcPort, dstPort, action, interfaceId
| sort @timestamp desc
| limit 50
```

You can replace `22` with:

* `80`
* `443`
* `3306`
* `5432`
* `6379`
* `8080`

This helps students trace who is trying to connect to which service port. ([AWS Documentation][2])

---

# 25. Query 6: Find all communication for one EC2 private IP

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter srcAddr = "10.0.1.10" or dstAddr = "10.0.1.10"
| fields @timestamp, interfaceId, srcAddr, dstAddr, srcPort, dstPort, packets, bytes, action
| sort @timestamp desc
| limit 100
```

This is a strong real-time troubleshooting query when one server is failing to connect to another service. ([AWS Documentation][2])

---

# 26. Query 7: Top talkers by source IP

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| stats sum(bytes) as totalBytes, sum(packets) as totalPackets by srcAddr
| sort totalBytes desc
| limit 20
```

This query helps identify which source IPs are generating the most traffic volume. Logs Insights supports `stats` aggregation for this use case. ([AWS Documentation][10])

---

# 27. Query 8: Top destination ports being used

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| stats count() as flowCount, sum(bytes) as totalBytes by dstPort
| sort flowCount desc
| limit 20
```

This is very useful to understand which services are most used in your VPC during a time window. ([AWS Documentation][10])

---

# 28. Query 9: Show rejected traffic by destination port

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter action = "REJECT"
| stats count() as rejectedCount by dstPort
| sort rejectedCount desc
| limit 20
```

This is a very strong security and troubleshooting query because it quickly shows which blocked service ports are seeing the most denied attempts. ([AWS Documentation][2])

---

# 29. Query 10: Show communication pairs

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| stats count() as flows, sum(bytes) as totalBytes by srcAddr, dstAddr
| sort totalBytes desc
| limit 30
```

This query is useful when you want to understand which source-destination pairs are actively communicating the most. ([AWS Documentation][10])

---

# 30. Query 11: Find traffic for HTTPS only

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter dstPort = "443"
| fields @timestamp, srcAddr, dstAddr, srcPort, dstPort, bytes, action
| sort @timestamp desc
| limit 50
```

Good for showing traffic going to secure endpoints or internet destinations over HTTPS. ([AWS Documentation][2])

---

# 31. Query 12: Filter traffic for one ENI

```sql
parse @message "* * * * * * * * * * * * * *" 
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, startTime, endTime, action, logStatus
| filter interfaceId = "eni-xxxxxxxxxxxxxxxxx"
| fields @timestamp, srcAddr, dstAddr, srcPort, dstPort, bytes, action
| sort @timestamp desc
| limit 100
```

This is extremely useful when troubleshooting one specific EC2 instance or one specific network interface resource. ([AWS Documentation][2])

---

# 32. Practical demo flow for your class

Use this classroom sequence:

### Demo 1: Enable VPC Flow Log

Create flow log for full VPC to CloudWatch Logs. ([AWS Documentation][6])

### Demo 2: Generate accepted traffic

Open website hosted on EC2 or run `curl` from one instance to another on an allowed port. ([AWS Documentation][2])

### Demo 3: Generate rejected traffic

Remove an inbound SG rule or test on a blocked port and then query for `REJECT`. ([AWS Documentation][2])

### Demo 4: Query traffic in Logs Insights

Run:

* recent records
* accepted
* rejected
* top ports
* top source IPs ([AWS Documentation][9])

This full cycle makes the topic very easy for students.

---

# 33. Important operational points to teach

## Flow logs are not real-time packet-by-packet capture

AWS states logs are aggregated and then delivered, with CloudWatch delivery typically around 5 minutes and on a best-effort basis. So students should not expect instant per-packet output. ([AWS Documentation][2])

## Flow logs are for network metadata

They are not a replacement for:

* Apache/Nginx access logs
* application logs
* database logs
* tcpdump/pcap payload analysis ([AWS Documentation][1])

## Query results depend on selected time range

In Logs Insights, if the wrong time window is selected, the query may return nothing even when traffic exists. Logs Insights documentation emphasizes selecting log groups and time intervals when querying. ([AWS Documentation][9])

---

# 34. Common troubleshooting cases

## Case 1: No logs are appearing in CloudWatch

Check:

* flow log status created successfully
* correct destination log group selected
* IAM role has correct CloudWatch write permissions
* traffic actually occurred
* enough time has passed for delivery ([AWS Documentation][5])

## Case 2: Query returns no data

Check:

* right log group selected
* right time range selected
* parse pattern matches your actual log format
* traffic exists for that period ([AWS Documentation][9])

## Case 3: You only see ACCEPT, not REJECT

Possible reasons:

* flow log filter was set to ACCEPT only
* traffic was genuinely allowed
* your rejected test was not reaching the monitored scope you enabled logging for ([AWS Documentation][6])

## Case 4: Students are confused between SG/NACL issue and flow log output

Explain that flow logs do not directly say “Security Group blocked this” or “NACL blocked this” in plain text. They show traffic metadata and ACCEPT/REJECT result. You still correlate the result with SG, NACL, routes, and instance/service behavior. AWS positions flow logs as a network-traffic analysis and troubleshooting tool, not a full root-cause explanation engine. ([AWS Documentation][1])

---

# 35. Best real-time use cases for production

You can explain these practical examples:

### Website not opening

Query for traffic to port 80 or 443 and check whether records are ACCEPT or REJECT. ([AWS Documentation][2])

### SSH not working

Query destination port 22 and see which source IP attempted access and whether it was rejected. ([AWS Documentation][2])

### Database not reachable

Query database port like 3306 or 5432 and inspect whether traffic from application subnet is reaching the database ENI or subnet. ([AWS Documentation][2])

### Suspicious scanning activity

Use top source IP and rejected-port queries to see if one IP is repeatedly attempting many ports. ([AWS Documentation][2])

### Unexpected outbound communication

Find top destination IPs and traffic volume by bytes to identify unusual egress patterns. ([AWS Documentation][2])

---

# 36. How to explain ACCEPT and REJECT properly

`ACCEPT` means AWS recorded the traffic as allowed at the flow log observation point.
`REJECT` means AWS recorded the traffic as denied. ([AWS Documentation][2])

Simple student explanation:

* ACCEPT = network allowed the flow
* REJECT = network blocked the flow

But still verify application/service health separately, because accepted network traffic does not guarantee the application returned a valid response. Flow logs are network metadata, not application success logs. ([AWS Documentation][1])

---

# 37. CloudWatch Logs Insights commands you should teach

The most useful commands for VPC Flow Logs are:

* `fields`
* `parse`
* `filter`
* `sort`
* `stats`
* `limit` ([AWS Documentation][10])

Meaning:

* `fields` = choose columns to display
* `parse` = split raw log line into named fields
* `filter` = keep only matching records
* `sort` = order results
* `stats` = aggregate counts/sums
* `limit` = show top N rows

This is enough for most VPC Flow Log analysis in class or interview rounds. ([AWS Documentation][10])

---

# 38. CLI example to create flow log

AWS documents `create-flow-logs` for the CLI. A typical pattern includes the resource type, resource ID, traffic type, destination log group, and the IAM role ARN used to publish to CloudWatch Logs. ([AWS Documentation][6])

Example structure:

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxxxxxxxxxxxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs/demo-vpc \
  --deliver-logs-permission-arn arn:aws:iam::<account-id>:role/vpc-flowlogs-to-cloudwatch-role
```

Use your actual:

* VPC ID
* log group name
* IAM role ARN

The CLI parameters shown here align with AWS’s Flow Logs API and CloudWatch destination setup pattern. ([AWS Documentation][6])

---

# 39. How to explain subnet-level vs VPC-level flow log

If you enable at **VPC level**, you get a broader view and usually better for full-environment monitoring. If you enable at **subnet level**, it is good for isolating public subnet, private subnet, or database subnet behavior. If you enable at **ENI level**, it is best for targeted troubleshooting of one resource. AWS supports all three scopes. ([AWS Documentation][3])

Teaching recommendation:

* VPC-level for broad learning
* ENI-level for focused troubleshooting demo

---

# 40. Best teaching structure for your session

Use this flow:

### Part 1

What is VPC Flow Log
Why network metadata matters
Difference between app logs and network logs ([AWS Documentation][1])

### Part 2

Create CloudWatch log group
Create IAM role
Create VPC Flow Log to CloudWatch ([AWS Documentation][5])

### Part 3

Generate accepted traffic
Generate rejected traffic ([AWS Documentation][2])

### Part 4

Open raw flow records
Run Logs Insights queries
Interpret ACCEPT/REJECT and top talkers/ports ([AWS Documentation][8])

### Part 5

Troubleshoot real examples
SSH issue
Website issue
DB issue
Suspicious traffic issue ([AWS Documentation][11])

---

# 41. Short explanation 

“VPC Flow Logs capture network traffic metadata for a VPC, subnet, or ENI. We can store them in CloudWatch Logs and query them using CloudWatch Logs Insights. They help us see source IP, destination IP, ports, protocol, bytes, packets, and whether the traffic was accepted or rejected. This is very useful for troubleshooting Security Group, NACL, and connectivity issues.” ([AWS Documentation][1])

---

# 42. Final recommendation from me

For your live session, do this minimum practical:

1. Create one EC2 web server
2. Create VPC Flow Log for the VPC to CloudWatch
3. Access website from browser
4. Try one blocked port test
5. Query:

   * recent logs
   * ACCEPT
   * REJECT
   * port 80
   * top source IPs

