# Managing your cloud costs

This document explains how to track your resource usage and optimize your costs in our cloud service.

## OpenStack Management Interface

To track your resource usage and costs, use the OpenStack management interface, which includes two key sections:

1. **Rating**: Provides an overview of your resource usage and associated costs. Metrics include:
    - `instance`: Cost of running virtual machines (VMs).
    - `image.size`: Storage costs for images.
    - `volume.size`, `volume.backup.size`, `volume.snapshot.size`: Costs for volumes and backups.
    - Other network and object storage usage metrics.

2. **Reporting**: Offers a graphical representation of your cumulative costs over time, including cost distribution across different services. Monitor costs per service per hour to understand your spending trends.

## Managing Virtual Machines (Instances)

When managing VMs, consider the following:

- **Shutting Down vs. Shelving**:
    - **Shutting Down**: Continues charging for reserved resources like CPU and RAM.
    - **Shelving**: Stops charging for running resources, but you still pay for storage.

## Costs of a Running Instance

When an instance is running, you pay for:

- **Instance Costs**: The cost of running the VM, including allocated CPU, RAM, and network resources.
- **Volume Size**: Any storage volumes attached to the instance.

## Saving Costs

To optimize your spending, shelve instances when they are not actively needed but still require data retention.

## Additional Information

- **Current Pricing**: For instance and storage prices, visit [Leaf Cloud Pricing](https://www.leaf.cloud/pricing).
- **Network and Load Balancers**: Currently, there are no charges for load balancer networks and network traffic.
