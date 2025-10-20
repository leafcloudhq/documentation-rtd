# Your Account Dashboard

After signing up for Leafcloud, you're automatically set up with everything you need to start building. This guide will help you understand what was created for you and how to navigate your account dashboard at [my.leaf.cloud](https://my.leaf.cloud).

---

## What Was Created for You

When you completed the signup process, Leafcloud automatically created:

- **✅ Your Account** - Your personal Leafcloud account with authentication via Keycloak
- **✅ Default Project** - An OpenStack project where all your resources will be organized
- **✅ Default Team** - A team structure for managing access and collaboration
- **✅ OpenStack Access** - Full access to the OpenStack dashboard at [create.leaf.cloud](https://create.leaf.cloud)
- **✅ Kubernetes Access** - Access to create managed Kubernetes clusters via Gardener
- **✅ Billing Setup** - Your billing and payment configuration

You're ready to start building right away!

---

## Accessing Your Account Dashboard

1. Navigate to **[my.leaf.cloud](https://my.leaf.cloud)**
2. If you're not already logged in, sign in with your credentials
3. You'll land on your account dashboard

!!! tip "Single Sign-On"
    Once logged in to my.leaf.cloud, you can seamlessly access the other Leafcloud portals (OpenStack, Gardener, Identity) without signing in again.

---

## Dashboard Overview

Your account dashboard at **my.leaf.cloud** is your central hub for managing your Leafcloud organization.

### Main Navigation

The dashboard is organized into several key sections:

#### **Home**
Your main dashboard showing an overview of your account and quick access to common tasks:

- **OpenStack** - Quick link to deploy instances
- **Kubernetes** - Quick link to create clusters
- **Documentation and tutorials** - Links to guides
- **Profile** - Manage your personal information
- **Need Help?** - Access to FAQ and support

#### **Projects**
View and manage your OpenStack projects:

- See your default project
- Create additional projects if needed
- View project quotas and resource limits
- Access project-specific settings

#### **Teams**
Manage your organization's teams:

- View your default team members
- Invite new team members
- Assign users to projects
- Manage roles and permissions

#### **Users**
Manage user accounts within your organization:

- View all users in your organization
- See team memberships
- Manage user access

#### **Usage**
Monitor your resource usage:

- View current usage across projects
- Track compute, storage, and network usage
- Monitor costs

#### **Invoices**
Access your billing information:

- View past invoices
- Download invoice PDFs
- Track payment history

#### **Profiles**
Manage billing profiles:

- Configure payment methods
- Update billing information
- Set up multiple billing profiles if needed

---

## Your Default Project

Your **default project** was automatically created during signup. This project contains:

### Project Information
- **Project Name** - Usually named after your organization or "Default"
- **Project ID** - A unique identifier for API access
- **Quotas** - Resource limits for instances, volumes, networks, etc.

### What You Can Do
- Launch virtual machines (instances)
- Create and manage storage volumes
- Set up networking (routers, networks, security groups)
- Use object storage (S3-compatible)
- Deploy load balancers
- Configure DNS records

!!! info "Multiple Projects"
    While most users start with one project, you can create additional projects to organize resources by environment (dev, staging, production) or by team/department.

---

## Your Default Team

Your **default team** includes you as the initial member with full permissions.

### Team Structure
- **You** - Team owner with admin rights
- **Permissions** - Full access to the default project
- **Roles** - Admin, allowing you to invite others and manage resources

### Growing Your Team
You can invite team members to collaborate:
- **Admin** - Full control over projects and team
- **Member** - Can create and manage resources
- **Viewer** - Read-only access

Learn more: [Managing Your Team →](managing-your-team.md)

---

## Navigating to Other Portals

From **my.leaf.cloud**, you can access the other Leafcloud portals:

### OpenStack Dashboard
- Click **"Deploy an Instance"** on the home dashboard, or
- Navigate directly to **[create.leaf.cloud](https://create.leaf.cloud)**
- Manage virtual machines, networks, storage, and more

### Kubernetes Dashboard (Gardener)
- Click **"Deploy a Cluster"** on the home dashboard, or
- Navigate directly to **[dashboard.gardener.leaf.cloud](https://dashboard.gardener.leaf.cloud)**
- Create and manage Kubernetes clusters

### Identity & Security (Keycloak)
- Click on your profile name → **"Settings"** → **"Signing in"**, or
- Navigate directly to **[identity.leaf.cloud](https://identity.leaf.cloud)**
- Manage passwords, set up passkeys, view security settings

---

## Common Tasks

Here are some common tasks you can perform from your account dashboard:

### View Your Projects
1. Click **"Projects"** in the left sidebar
2. You'll see your default project listed
3. Click on a project to view details and settings

### Check Your Usage
1. Click **"Usage"** in the left sidebar
2. View current resource consumption
3. Monitor costs and usage trends

### Invite a Team Member
1. Click **"Teams"** in the left sidebar
2. Select your team
3. Click **"Invite User"**
4. Enter their email address and assign a role

Learn more: [Managing Your Team →](managing-your-team.md)

### Create an Additional Project
1. Click **"Projects"** in the left sidebar
2. Click **"Create Project"**
3. Give it a name and description
4. Configure quotas and team assignments

Learn more: [Creating Additional Projects →](creating-additional-projects.md)

---

## Account Settings

To manage your personal account settings:

1. Click on your **profile name** in the top right corner
2. Select **"Settings"**
3. Update your:
   - First and last name
   - Email address (note: email changes may require verification)
   - Phone number

---

## Next Steps

Now that you're familiar with your account dashboard, here's what to do next:

<div class="grid cards" markdown>

-   :material-key:{ .lg .middle } **Set Up Passkey**
    
    ---
    
    Enable passwordless authentication for easier, more secure access
    
    [:octicons-arrow-right-24: Set Up Passkey](setup-passkey.md)

-   :material-account-multiple:{ .lg .middle } **Invite Your Team**
    
    ---
    
    Add team members and start collaborating
    
    [:octicons-arrow-right-24: Managing Your Team](managing-your-team.md)

-   :material-rocket-launch:{ .lg .middle } **Launch Your First Instance**
    
    ---
    
    Deploy your first virtual machine on OpenStack
    
    [:octicons-arrow-right-24: Accessing OpenStack Dashboard](accessing-openstack-dashboard.md)

-   :material-kubernetes:{ .lg .middle } **Create a Kubernetes Cluster**
    
    ---
    
    Deploy a managed Kubernetes cluster with Gardener
    
    [:octicons-arrow-right-24: Creating a Kubernetes Cluster](../gardener/creating-a-kubernetes-cluster.md)

</div>

---

## Need Help?

If you have questions about your account dashboard:

- **[Visit our FAQ](https://leaf.cloud/faq)** - Common questions answered
- **[Contact Support](https://leaf.cloud/contact)** - Email or call our support team
- **[Check Status](https://status.leaf.cloud)** - Platform status and updates

