# AWS Athena Queries for Cost Attribution

This document provides sample AWS Athena queries for analyzing AWS Cost and Usage Report (CUR) data. By leveraging standardized resource tagging, these queries help identify the top cost contributors across different service categories.


## 1. Overall Top Cost Contributors
This query aggregates cost by the common resource tag used across your deployment.
```sql
SELECT 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```


2. SaaS IAM Users
Assuming SaaS-related resources are tagged with iam_user, this query identifies the top cost contributors for SaaS services.

```sql

SELECT 
  tag_value AS iam_user, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE tag_key = 'iam_user'
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY tag_value
ORDER BY total_cost DESC
LIMIT 10;
```

3. PaaS IAM Users/Tags
For PaaS services, use the tag (e.g., paas) to filter costs.

```sql
SELECT 
  tag_value AS paas_service, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE tag_key = 'paas'
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY tag_value
ORDER BY total_cost DESC
LIMIT 10;
```

4. Containers (ECS/EKS/Fargate)
Query for containerized services by filtering on the product codes associated with container services.

```sql
SELECT 
  product_code, 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE product_code IN ('AmazonEKS', 'AmazonECS', 'AWSFargate')
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY product_code, resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```

5. Compute Resources (EC2 & Fargate)
This query fetches cost details for EC2 instances and Fargate tasks.

```sql
SELECT 
  product_code, 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE product_code IN ('AmazonEC2', 'AWSFargate')
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY product_code, resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```

6. Storage (S3 and EBS)
Analyze storage costs by filtering for S3 and EBS service product codes.
```sql

SELECT 
  product_code, 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE product_code IN ('AmazonS3', 'AmazonEBS')
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY product_code, resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```

7. Elastic IPs
Extract costs associated with Elastic IPs by looking for specific usage types within the EC2 service.

```sql
SELECT 
  product_code, 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE product_code = 'AmazonEC2'
  AND usage_type LIKE '%ElasticIP%'
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY product_code, resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```
8. Redshift
Identify top contributors for Redshift usage.

```sql
SELECT 
  product_code, 
  resource_tag, 
  SUM(line_item_unblended_cost) AS total_cost
FROM aws_billing.cur_table
WHERE product_code = 'AmazonRedshift'
  AND line_item_usage_start_date BETWEEN date '2025-03-01' AND date '2025-03-31'
GROUP BY product_code, resource_tag
ORDER BY total_cost DESC
LIMIT 10;
```
**Summary**
By applying these Athena queries, you can quickly identify the highest cost drivers within your AWS environment. The queries leverage standardized tagging (for IAM users, PaaS, containers, EC2, Fargate, storage, Elastic IPs, and Redshift) so that any resource can be easily filtered and aggregated. This approach provides a unified view, enabling effective cost optimization and governance.

For further optimization, integrating these queries into dashboards (e.g., Grafana) and automating anomaly detection using ML models such as Isolation Forest or LSTM-based forecasting via AWS SageMaker can be done.
