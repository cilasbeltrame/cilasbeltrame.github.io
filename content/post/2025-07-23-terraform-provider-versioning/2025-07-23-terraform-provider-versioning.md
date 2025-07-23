+++
date = '2025-07-23T09:37:35-03:00'
title = 'Terraform Provider Versioning'
subtitle = 'When to pin at composition and when to pin at base/root modules'
tags = ["terraform", "devops"]
+++

## Introduction

Provider versioning in Terraform is a critical aspect of infrastructure management that often gets overlooked until something breaks. The question isn't whether to pin provider versions, but **where** and **how** to pin them effectively. This article explores the two main approaches: pinning at the composition level versus pinning at base/root modules, and provides guidance on when to use each strategy.

## Understanding provider versioning strategies

### Base/root module pinning

In this approach, you specify provider version constraints directly in your base modules or root configurations:

```hcl
# modules/ec2-instance/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

### Composition-level pinning

Here, you define provider versions in your composition files (the files that call your modules):

```hcl
# environments/production/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"  # Exact version pinning
    }
  }
}

module "web_servers" {
  source = "../../modules/ec2-instance"
  # Module doesn't specify provider version
}
```

## When to use each approach

### Use base/root module pinning when:

**1. Building reusable modules**
- Your modules will be used across different projects or teams
- You need to ensure compatibility with specific provider features
- The module has dependencies on particular provider capabilities

```hcl
# modules/kubernetes-cluster/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0, < 6.0"  # Allows flexibility while ensuring compatibility
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }
}
```

**2. Protecting against breaking changes**
- The module uses provider features that frequently change
- You've experienced issues with provider updates in the past
- The module is complex and testing with new provider versions is time-consuming

### Use composition-level pinning when:

**1. Managing environment consistency**
- You want identical provider versions across all environments
- You're following a controlled update process
- You need to coordinate provider updates with application deployments

```hcl
# environments/production/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"
    }
    datadog = {
      source  = "DataDog/datadog"
      version = "= 3.32.0"
    }
  }
}
```

**2. Centralized version management**
- You want a single place to manage all provider versions
- You're implementing organization-wide standards
- You need to track and audit provider version usage

## Best practices

### 1. Use semantic versioning constraints wisely

```hcl
# Good: Allows patch updates but prevents breaking changes
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.31"  # Allows 5.31.x but not 5.32.x
    }
  }
}

# Avoid: Too restrictive for modules
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"  # Only use exact pinning in compositions
    }
  }
}
```

### 2. Implement a hybrid approach

Combine both strategies for maximum flexibility and control:

```hcl
# Module defines minimum requirements
# modules/database/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}

# Composition enforces exact versions
# environments/production/versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"
    }
  }
}
```

### 3. Document version dependencies

Always document why specific versions are required:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      # Version 5.20+ required for vpc_endpoint_route_table_association
      # Version < 6.0 to avoid breaking changes in EKS module
      version = "~> 5.20, < 6.0"
    }
  }
}
```

### 4. Establish an update process

Create a systematic approach to provider updates:

1. **Development Environment**: Test with latest provider versions
2. **Staging Environment**: Use specific versions that passed dev testing
3. **Production Environment**: Use exact versions proven in staging

```hcl
# dev/versions.tf - Allow latest within major version
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# staging/versions.tf - Pin to tested version
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"
    }
  }
}

# production/versions.tf - Same as staging after validation
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.31.0"
    }
  }
}
```

