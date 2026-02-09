For this part, which is important because we need to get the compute and serverless issue resolved.

On all of the environments (dev, test, prod) we have the all purpose compute:

DataEngineeringCluster:

Vendor : Databricks 
Creator: 68c8c909-ed0e-4770-8f09-4b9c03e3c1a6 
ClusterName: DataEngineeringCluster 
ClusterId: 0720-012626-pen66ygb 
Application: dw2023 
Environment: dev 
Type: workspace

Vendor: Databricks
Creator: 68c8c909-ed0e-4770-8f09-4b9c03e3c1a6
ClusterName: DataEngineeringCluster
ClusterId: 0720-015200-svl8c91v
Application: dw2023
Environment: test
Type: workspace

Vendor: Databricks
Creator: 30679f56-cbe6-4aff-b7ca-fadae1196fb5
ClusterName: DataEngineeringCluster
ClusterId: 0720-160151-np5pb9p5
Application: dw2023
Environment: prod
Type: workspace

Then we also have the serverless compute for the data warehouse jobs:

DEV
Status: Stopped
Name: sql-warehouse-dw2023-dev-sdt (ID: 0ce33986b0810363)
Type: Serverless
Cluster size: 2X-Small
Auto stop: After 10 minutes of inactivity
Scaling: Cluster count: Active 0 Min 1 Max 1
Channel: Current: (v 2025.35)
Created by: 68c8c909-ed0e-4770-8f09-4b9c03e3c1a6

TEST
Status: Stopped
Name: sql-warehouse-dw2023-test-sdt (ID: df378c87894172b0)
Type: Serverless
Cluster size: Small
Auto stop: After 30 minutes of inactivity
Scaling: Cluster count: Active 0 Min 1 Max 2
Channel: Current: (v 2025.35)
Created by: 68c8c909-ed0e-4770-8f09-4b9c03e3c1a6

PROD
Status: Running
Name: sql-warehouse-dw2023-prod-sdt (ID: 0e60233a0ac64392)
Type: Serverless
Cluster size: X-Small
Auto stop: After 30 minutes of inactivity
Scaling: Cluster count: Active 1 Min 1 Max 1
Channel: Current: (v 2025.35)
Created by: 30679f56-cbe6-4aff-b7ca-fadae1196fb5