# AWS CloudFormation

## Overview

AWS CloudFormation allows you to model and provision AWS infrastructure using template files. The workflow is:
```
Template → Upload to S3 Bucket → Reference in CloudFormation → Create Stack → Resources Created
```

---

## Template Components

### Core Structure

CloudFormation templates consist of several key components:

!!! example "📝 Template Components"
    - **AWSTemplateFormatVersion**: Specifies the template format version (capability declaration)
    - **Description**: Comments about the template's purpose
    - **Resources** (MANDATORY): AWS resources to be created
    - **Parameters**: Dynamic inputs for the template
    - **Mappings**: Static variables for the template
    - **Outputs**: References to created resources that can be exported
    - **Conditions**: Logic to conditionally create resources

---

## Resources

!!! info "Information"
    Resources are the core building blocks of your CloudFormation template. They declare the AWS services you want to create and can reference each other.

### Resource Syntax

Resources follow this naming convention:
```
Service-Provider::Service-Name::Data-Type-Name
```

**Examples:**
- `AWS::EC2::Subnet`
- `AWS::EC2::Instance`
- `AWS::RDS::DBInstance`

!!! tip "Tips"
    - You can create a dynamic number of resources
    - If a service isn't available, use **CloudFormation Custom Resources** as a workaround

---

## Parameters

Parameters provide a way to input values into your CloudFormation template at runtime.

!!! warning "Important"
    Use parameters when:
    - Configuration is likely to change in the future
    - You don't know the value at template creation time

### Parameter Features

Parameters are not limited to strings. You can specify:
- **Allowed Values**: Constrain inputs to specific options
- **Default Values**: Provide sensible defaults

**Example:**
```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
```

### Referencing Parameters

Use the `!Ref` function (or `Fn::Ref`):
```yaml
InstanceType: !Ref InstanceTypeParameter
```

!!! note "Notes"
    You use `!Ref` the same way for both parameters and resources:
    - `!Ref <ParameterName>`
    - `!Ref <ResourceName>`

---

## Mappings

Mappings define fixed variables within your template, useful for differentiating between:
- Environments (dev, prod)
- Regions
- AMI types

!!! example "📝 Key Points"
    - All values are hardcoded in mappings
    - Use mappings when values can be deduced from variables like region, availability zone, or AWS account
    - Use parameters when values need to be provided at runtime

### Using Mappings

Use `Fn::FindInMap` or `!FindInMap`:
```yaml
!FindInMap [MapName, TopLevelKey, SecondLevelKey]
```

**Example:**
```yaml
!FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
```

---

## Outputs

Outputs are optional values that can be exported and referenced by other stacks.

!!! info "Information"
    Outputs are commonly used to export resource identifiers like security groups, VPC IDs, or subnet IDs that other stacks may need.

### Output Syntax
```yaml
Outputs:
  StackSSHSecurityGroup:
    Description: Security Group for SSH access
    Value: !Ref MySecurityGroup
    Export:
      Name: SSHSecurityGroup
```

### Importing Outputs

Use `!ImportValue` to reference exported outputs:
```yaml
SecurityGroup: !ImportValue SSHSecurityGroup
```

!!! danger "Important"
    You cannot delete a stack with exported outputs until all references to those outputs are removed from other stacks.

---

## Conditions

Conditions allow you to control resource creation based on logical evaluation.

!!! tip "Tips"
    Use conditions to create resources only in specific environments (dev/prod) or regions.

### Defining Conditions
```yaml
Conditions:
  CreateProdResources: !Equals [!Ref EnvType, prod]
```

### Using Conditions
```yaml
Resources:
  MyResource:
    Type: AWS::EC2::Instance
    Condition: CreateProdResources
```

---

## Intrinsic Functions

CloudFormation provides built-in functions for dynamic template behavior:

| Function | Shorthand | Purpose |
|----------|-----------|---------|
| `Fn::Ref` | `!Ref` | Reference parameters or resources |
| `Fn::GetAtt` | `!GetAtt` | Get attributes from resources (e.g., `!GetAtt EC2Instance.AvailabilityZone`) |
| `Fn::FindInMap` | `!FindInMap` | Retrieve values from mappings |
| `Fn::ImportValue` | `!ImportValue` | Import exported values from other stacks |
| `Fn::Base64` | `!Base64` | Convert string to Base64 (useful for UserData) |

### Condition Functions

- `Fn::And`
- `Fn::Equals`
- `Fn::If`
- `Fn::Not`
- `Fn::Or`

!!! example "📝 GetAtt Example"
```yaml
    AvailabilityZone: !GetAtt EC2Instance.AvailabilityZone
    PublicIP: !GetAtt EC2Instance.PublicIp
    PrivateDNS: !GetAtt EC2Instance.PrivateDnsName
```

---

## Stack Updates and Deletion

### Updating Stacks

!!! note "Notes"
    To update a template:
    1. Create a new version of the template
    2. Re-upload the template
    3. CloudFormation updates the stack with the changes

---

## CloudFormation Rollback

### Stack Creation Failures

!!! warning "Important"
    **Default behavior**: Everything rolls back (gets deleted) on failure
    
    You can disable automatic rollback to troubleshoot issues.

### Stack Update Failures

When a stack update fails, CloudFormation automatically rolls back to the last known working state.

!!! danger "Important"
    **Rollback failures** typically occur when:
    - Resources were manually changed outside of CloudFormation
    - The stack is in an inconsistent state
    - Manual intervention is required to resolve conflicts

---

## CloudFormation StackSets

!!! info "Information"
    AWS CloudFormation StackSets extends stack functionality by enabling you to create, update, or delete stacks across **multiple accounts and regions** with a single operation.

### How StackSets Work

1. Use an **administrator account** to define and manage a CloudFormation template
2. The template becomes the basis for provisioning stacks
3. Deploy to selected **target accounts** across specified **regions**

!!! tip "Tips"
    StackSets are ideal for:
    - Multi-account AWS Organizations setups
    - Standardized resource deployment across regions
    - Centralized governance and compliance

---

## Best Practices

!!! example "📝 Key Points"
    - Always use **parameters** for values that may change
    - Use **mappings** for environment-specific or region-specific values
    - Export commonly used resources via **outputs**
    - Implement **conditions** to manage environment-specific resources
    - Enable **termination protection** on production stacks
    - Use **change sets** to preview changes before applying them
    - Tag your resources for better organization and cost tracking

!!! question "Questions to Keep in Mind"
    - Does this configuration need to be flexible? → Use **parameters**
    - Is this value fixed per environment/region? → Use **mappings**
    - Will other stacks need this resource? → Export via **outputs**
    - Should this only exist in certain environments? → Use **conditions**
    - How will I handle failures? → Plan your **rollback strategy**