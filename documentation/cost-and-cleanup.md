# AWS Cost Management and Resource Cleanup

## 1. Overview

This document describes the cost-management considerations and cleanup activities associated with the AWS Cloud Infrastructure learning lab.

The infrastructure was deployed temporarily in the AWS Mumbai region (`ap-south-1`) for implementation, connectivity testing, and monitoring validation.

After completing the lab, the AWS resources were deleted as part of the cleanup process.

## 2. AWS Cost Management

The project used AWS services that can incur charges depending on resource configuration, usage duration, and applicable pricing.

Services considered during the lab:

| AWS Service | Cost Consideration |
|---|---|
| Amazon EC2 | Instance running time and instance type |
| EBS | Provisioned storage volumes |
| Public IPv4 | Applicable public IPv4 charges |
| Data Transfer | Applicable data transfer usage |
| Amazon CloudWatch | Metrics, dashboards, alarms, and additional monitoring features |
| Internet Gateway | No separate hourly attachment charge; associated data transfer may incur charges |

Actual charges depend on AWS pricing, account eligibility, region, and usage.

## 3. Cost Awareness Practices

The following practices were followed:

- Used small EC2 instance types appropriate for the lab.
- Monitored AWS credit and billing information.
- Reviewed service-level cost information.
- Used basic EC2 monitoring rather than enabling detailed monitoring.
- Deleted temporary resources after testing.
- Considered storage and public IPv4 usage when planning cleanup.

## 4. Resource Cleanup

After implementation and validation, the temporary AWS resources were deleted.

The cleanup process covered the resources created for the lab, including:

- EC2 instances.
- Associated EBS volumes, where applicable.
- NAT-related network configuration.
- Security groups created for the project.
- Custom route tables.
- Internet Gateway.
- Public and private subnets.
- Custom VPC.

The final resource state should be verified in the AWS console and billing dashboard.

## 5. Post-Cleanup Verification

After deleting resources, the following checks should be performed:

| Verification | Purpose |
|---|---|
| EC2 Instances | Confirm project instances are terminated |
| EBS Volumes | Check for unattached volumes |
| Elastic IPs | Check for unused allocated addresses |
| NAT Resources | Check for remaining NAT-related resources |
| Load Balancers | Check for unused load balancers, if any were created |
| VPC Resources | Confirm unused project networking resources are removed |
| Billing Dashboard | Review remaining or delayed charges |

Billing information may take time to reflect recent resource deletion and usage.

## 6. Important Cost Considerations

Deleting an EC2 instance does not automatically mean every related resource has been removed.

For example:

- EBS volumes may remain if configured to be retained.
- Snapshots may continue to incur storage charges.
- Allocated Elastic IP addresses may incur charges depending on their state and applicable pricing.
- Other resources outside the project may remain active.

Always verify resources individually before considering cleanup complete.

## 7. Cleanup Limitations

The cleanup status in this document reflects the lab operator's reported deletion of the temporary project resources.

A final independent audit of all AWS account resources and billing records is outside the scope of this repository.

## 8. Conclusion

Cost awareness and cleanup are important parts of cloud infrastructure administration.

This lab provided practical experience with monitoring AWS usage, understanding potential cost sources, and managing the lifecycle of temporary cloud resources.

**Environment:** Personal AWS learning lab.

**Project status:** Implementation and connectivity validation completed; cleanup reported complete.
