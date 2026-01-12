# Platform Landing Zone with ALZ-Bicep - Simple Guide

## What is This Repository?

This repository provides **ready-to-use Bicep code modules** to deploy Azure Landing Zones (ALZ) - a proven enterprise-scale platform foundation for your Azure environment. Think of it as a collection of building blocks that help you set up a secure, well-architected Azure platform following Microsoft's best practices.

## Why Use This for Platform Landing Zones?

When creating a **platform landing zone** using Bicep, this repository gives you:

- ✅ **Pre-built, tested modules** - No need to write everything from scratch
- ✅ **Microsoft best practices built-in** - Follows Cloud Adoption Framework guidelines
- ✅ **Modular approach** - Reuse only what you need
- ✅ **Production-ready code** - Actively maintained by Microsoft and the community
- ✅ **Governance included** - Azure Policies, RBAC, and management groups configured
- ✅ **Flexible deployment** - Deploy pieces individually or orchestrate together

## What Can You Deploy?

This repository provides Bicep modules to create your entire platform landing zone infrastructure:

### Core Platform Components
- **Management Groups** - Hierarchical organization structure for your Azure subscriptions
- **Logging & Monitoring** - Centralized Log Analytics workspace and monitoring solutions
- **Policy Management** - Azure Policy definitions and assignments for governance
- **Role Definitions** - Custom RBAC roles tailored for Azure Landing Zones
- **Role Assignments** - Assign appropriate permissions across your platform

### Networking Components
- **Hub Networking** - Central hub for shared services (Hub-and-Spoke topology)
- **Virtual WAN** - Cloud-native global transit network architecture
- **Spoke Networks** - Workload landing zone networks
- **Network Peering** - Connect hub and spoke virtual networks
- **Private DNS Zones** - DNS resolution for private endpoints

### Landing Zone Operations
- **Subscription Management** - Create and organize subscriptions
- **Resource Groups** - Deploy and manage resource groups
- **Diagnostic Settings** - Configure logging at management group level

## How to Get Started Reusing This Code

### Prerequisites

Before you begin, ensure you have:
- **Azure subscription** with appropriate permissions (Owner or Contributor + User Access Administrator)
- **Azure CLI** or **PowerShell** installed with Bicep CLI
- **Understanding of your requirements** - Network topology (Hub-Spoke or Virtual WAN), naming conventions, etc.

### Step-by-Step Approach

1. **Fork or Clone This Repository**
   ```bash
   git clone https://github.com/Azure/ALZ-Bicep.git
   cd ALZ-Bicep
   ```

2. **Review the Deployment Flow Documentation**
   - Start here: [Deployment Flow Wiki](https://github.com/Azure/ALZ-Bicep/wiki/DeploymentFlow)
   - Choose your network topology:
     - [Hub and Spoke](https://github.com/Azure/ALZ-Bicep/wiki/DeploymentFlowHS)
     - [Virtual WAN](https://github.com/Azure/ALZ-Bicep/wiki/DeploymentFlowVWAN)

3. **Customize Parameters**
   - Navigate to the module you want to deploy (e.g., `infra-as-code/bicep/modules/managementGroups`)
   - Review the `.bicepparam` or `.parameters.json` files
   - Modify parameters to match your requirements (naming, regions, networking, etc.)

4. **Deploy Modules Sequentially**
   
   Each module is designed to be deployed independently. Typical deployment order:
   
   ```bash
   # 1. Create Management Group hierarchy
   az deployment tenant create --location <region> \
     --template-file infra-as-code/bicep/modules/managementGroups/managementGroups.bicep \
     --parameters @infra-as-code/bicep/modules/managementGroups/parameters/managementGroups.parameters.all.json
   
   # 2. Deploy Custom Role Definitions
   az deployment mg create --location <region> \
     --management-group-id <mg-id> \
     --template-file infra-as-code/bicep/modules/customRoleDefinitions/customRoleDefinitions.bicep \
     --parameters @infra-as-code/bicep/modules/customRoleDefinitions/parameters/customRoleDefinitions.parameters.all.json
   
   # 3. Deploy Logging (Log Analytics)
   az deployment sub create --location <region> \
     --template-file infra-as-code/bicep/modules/logging/logging.bicep \
     --parameters @infra-as-code/bicep/modules/logging/parameters/logging.parameters.all.json
   
   # 4. Deploy Policy Definitions and Assignments
   az deployment mg create --location <region> \
     --management-group-id <mg-id> \
     --template-file infra-as-code/bicep/modules/policy/definitions/customPolicyDefinitions.bicep \
     --parameters @infra-as-code/bicep/modules/policy/definitions/parameters/customPolicyDefinitions.parameters.all.json
   
   # 5. Deploy Hub Networking
   az deployment sub create --location <region> \
     --template-file infra-as-code/bicep/modules/hubNetworking/hubNetworking.bicep \
     --parameters @infra-as-code/bicep/modules/hubNetworking/parameters/hubNetworking.parameters.all.json
   
   # ... Continue with remaining modules
   ```

5. **Use Orchestration Modules** (Optional)
   - For streamlined deployment, use orchestration modules in `infra-as-code/bicep/orchestration/`
   - These combine multiple modules for common scenarios

### Repository Structure

```
ALZ-Bicep/
├── infra-as-code/bicep/
│   ├── modules/              # Individual reusable Bicep modules
│   │   ├── managementGroups/
│   │   ├── logging/
│   │   ├── hubNetworking/
│   │   ├── policy/
│   │   ├── customRoleDefinitions/
│   │   ├── spokeNetworking/
│   │   └── ...
│   ├── orchestration/        # Combined deployment modules
│   │   ├── hubPeeredSpoke/
│   │   ├── mgDiagSettingsAll/
│   │   └── ...
│   └── CRML/                 # Cross-Region Module Library
├── docs/                     # Documentation and wiki content
└── accelerator/              # Accelerator tools for faster deployment
```

## Key Modules for Platform Landing Zones

### Essential Modules to Reuse

1. **Management Groups** (`modules/managementGroups`) - Create your organizational hierarchy
2. **Logging** (`modules/logging`) - Set up centralized logging
3. **Policy** (`modules/policy`) - Implement governance policies
4. **Hub Networking** (`modules/hubNetworking`) - Deploy your network hub
5. **Custom Roles** (`modules/customRoleDefinitions`) - Define custom RBAC roles
6. **Role Assignments** (`modules/roleAssignments`) - Assign permissions

### Optional Enhancement Modules

- **Spoke Networking** (`modules/spokeNetworking`) - For application landing zones
- **Virtual WAN** (`modules/vwanConnectivity`) - Alternative to hub-spoke
- **Private DNS** (`modules/privateDnsZones`) - For private endpoint DNS
- **Subscription Placement** (`modules/subscriptionPlacement`) - Organize subscriptions

## Customization Tips

### 1. Modify Parameters Files
Each module includes sample `.parameters.json` files. Copy and customize:
- Naming conventions (prefixes, suffixes)
- Azure regions
- IP address ranges for networking
- Log retention periods
- Policy assignments

### 2. Extend Modules
You can import and reference these modules in your own Bicep files:
```bicep
// Reference a module from this repository
module managementGroups './infra-as-code/bicep/modules/managementGroups/managementGroups.bicep' = {
  name: 'myMgDeployment'
  params: {
    parTopLevelManagementGroupPrefix: 'myorg'
    parTopLevelManagementGroupDisplayName: 'My Organization'
    // Your other custom parameters
  }
}

// Or use orchestration modules for combined deployments
module hubSpoke './infra-as-code/bicep/orchestration/hubPeeredSpoke/hubPeeredSpoke.bicep' = {
  name: 'hubSpokeDeployment'
  params: {
    // Your parameters
  }
}
```

### 3. Add Custom Policies
Place your custom policy definitions in `modules/policy/definitions/` and reference them in assignments.

### 4. Use Accelerator for Quick Start
The repository includes an **Accelerator** tool (`accelerator/` directory) that guides you through the setup with a wizard-like experience.

## Deployment Options

### Option 1: Manual CLI Deployment
Deploy each module individually using Azure CLI or PowerShell (shown above)

### Option 2: Use Accelerator
Navigate to `accelerator/` and follow the interactive setup

### Option 3: CI/CD Pipeline
Use sample pipelines:
- GitHub Actions: See `docs/wiki` for GitHub Actions examples
- Azure DevOps: See `docs/wiki` for Azure DevOps pipelines

### Option 4: Azure Container Registry (Private Registry)
Deploy modules to your own Azure Container Registry for private module hosting

## Important Considerations

### 1. Prerequisites
- Ensure you have the right permissions (Owner at tenant root or management group level)
- Plan your management group structure before deployment
- Design your network topology (address spaces, regions, etc.)

### 2. Deployment Order Matters
Some modules depend on others. Follow the recommended deployment sequence in the wiki.

### 3. Policy and Compliance
Review all policy assignments before applying them to production - some may block certain resource types or configurations.

### 4. Naming Conventions
Establish and document your naming conventions before deployment for consistency.

### 5. Backup and Version Control
- Keep your customized parameter files in version control
- Test deployments in a non-production environment first

## Next Steps

1. **Read the Wiki** - [ALZ-Bicep Wiki](https://github.com/Azure/ALZ-Bicep/wiki/home)
2. **Watch Tutorial Videos** - Links available in the main README
3. **Review the Deployment Flow** - [Deployment Flow Guide](https://github.com/Azure/ALZ-Bicep/wiki/DeploymentFlow)
4. **Try the Accelerator** - Fastest way to get started
5. **Join the Community** - Contribute or ask questions via GitHub issues

## Support and Contributing

- **Support**: See [SUPPORT.md](SUPPORT.md) for Microsoft support policies
- **Contributing**: See [Contributing Guide](https://github.com/Azure/ALZ-Bicep/wiki/Contributing)
- **Issues**: Report issues or request features via GitHub Issues
- **Security**: Report security vulnerabilities per [SECURITY.md](SECURITY.md)

## Additional Resources

- [Cloud Adoption Framework - Azure Landing Zones](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/)
- [Azure Architecture Center - ALZ Bicep](https://learn.microsoft.com/azure/architecture/landing-zones/bicep/landing-zone-bicep)
- [Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure Landing Zones - Overview](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/)

---

## Quick Reference Commands

```bash
# Check your Bicep installation
az bicep version

# Upgrade Bicep
az bicep upgrade

# Validate a Bicep template
az bicep build --file <template.bicep>

# Deploy to tenant scope (for management groups)
az deployment tenant create --location <region> --template-file <file.bicep> --parameters <params.json>

# Deploy to management group scope (for policies)
az deployment mg create --location <region> --management-group-id <mg-id> --template-file <file.bicep>

# Deploy to subscription scope (for resources)
az deployment sub create --location <region> --template-file <file.bicep> --parameters <params.json>
```

---

**Remember**: This repository is maintained by Microsoft and the community. Always check for the latest version and updates before deploying to production environments.
