# Welcome to Leafcloud

Leafcloud is Europe's first truly sustainable public cloud platform, built on open-source OpenStack technology and powered by green energy. Our innovative Leaf Sites reuse server heat to warm buildings, reducing CO₂ emissions while providing you with powerful, secure, and flexible cloud infrastructure.

This documentation will guide you through everything you need to know to build, deploy, and scale your applications on Leafcloud.

**New to Leafcloud?** Start with [Getting Started](Getting-Started/) to set up your team and launch your first instance.

---

## The Leafcloud Platform

Leafcloud provides three integrated portals for managing your cloud infrastructure:

### 🏠 Account & Organization Management
**Portal:** [my.leaf.cloud](https://my.leaf.cloud)

Your central hub for account and organization management. Here you can:

- **Create and manage Projects** - Organize your OpenStack resources
- **Build Teams** - Invite and manage team members
- **Assign Users** - Control access and permissions
- **Manage Billing** - View usage and invoices
- **Configure Account Settings** - Update personal information

!!! tip "Start Here"
    If you're new to Leafcloud, begin at my.leaf.cloud to set up your organization and create your first project.

### ☁️ OpenStack Dashboard (Horizon)
**Portal:** [create.leaf.cloud](https://create.leaf.cloud)

The OpenStack dashboard where you manage your infrastructure resources:

- **Launch Virtual Machines** - Deploy instances with various flavors
- **Configure Networks** - Set up networking, routers, and load balancers
- **Manage Storage** - Create volumes and use object storage (S3)
- **Set up Security** - Configure security groups and key pairs
- **Monitor Resources** - Track usage and performance

### ⎈ Managed Kubernetes (Gardener)
**Portal:** [dashboard.gardener.leaf.cloud](https://dashboard.gardener.leaf.cloud)

Manage your Kubernetes clusters with Gardener:

- **Create Kubernetes Clusters** - Deploy production-ready clusters in minutes
- **Auto-scaling** - Configure worker groups with automatic scaling
- **Advanced Networking** - Set up ingress, DNS, and networking
- **Monitor Clusters** - View cluster health and resource usage
- **Access Control** - Manage kubeconfigs and permissions

!!! info "Kubernetes & OpenStack Integration"
    Kubernetes cluster resources (instances, volumes, load balancers) will appear in your OpenStack project, giving you complete visibility.

---

## Authentication & Security

All portals use secure authentication powered by Keycloak. You can manage your security settings at [identity.leaf.cloud](https://identity.leaf.cloud/realms/leafcloud/account/account-security/signing-in)

- **Password Management** - Change and reset passwords
- **Two-Factor Authentication (2FA)** - Set up authenticator apps
- **Passkey Support** - Enable passwordless authentication
- **Security Settings** - Manage sessions and device activity

