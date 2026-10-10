# CloudFormation Nested Stack Template - VPC with optionally Internet Gateway

<!-- Row 1: Status - Most Important -->
[![Release](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)&nbsp;[![GitHub Action](https://img.shields.io/badge/GitHub-Action-blue?logo=github)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-vpc/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/47f336f3716a7d89eba6109c564fd442/raw/cfn-nested-aws-vpc.json)](https://gist.github.com/subhamay-bhattacharyya/47f336f3716a7d89eba6109c564fd442)

This repository contains a nested CloudFormation template that creates an Amazon VPC with DNS support and hostnames enabled, plus an optional Internet Gateway attached to the VPC.

## Overview

This is a **nested stack template** designed to be invoked from a parent/root CloudFormation stack. The template is stored in this repository and should be uploaded to an S3 bucket so parent stacks can reference it via `TemplateURL`.

## Template Files

- **`cloudformation/template.yaml`** — Nested template that creates the VPC and the optional Internet Gateway
- **`cloudformation/parameters.json`** — Sample parameter values
- **`cloudformation/stack-config.json`** — Stack name and template/parameter file locations

## Resources Created

| Resource | Type | Created When | Description |
| ----------- | ------ | --------- | ------------- |
| `VPC` | `AWS::EC2::VPC` | Always | VPC using the `VPCCidrBlock` CIDR, tagged `<VpcNameTag>-vpc` |
| `InternetGateway` | `AWS::EC2::InternetGateway` | `EnableInternetGateway` is `true` | Internet Gateway tagged `<VpcNameTag>-vpc-igw` |
| `AttachGateway` | `AWS::EC2::VPCGatewayAttachment` | `EnableInternetGateway` is `true` | Attaches the Internet Gateway to the VPC |

## Template Features

- ✅ VPC with configurable CIDR block
- ✅ DNS support and DNS hostnames enabled by default (toggle with `EnableDnsHostnames`)
- ✅ Optional Internet Gateway, created and attached only when enabled
- ✅ Name and environment tagging on all resources
- ✅ Input validation for the name prefix and CIDR format

## Parameters

| Parameter | Type | Default | Allowed Values / Pattern | Description |
| ----------- | ------ | --------- | ------------- | ------------- |
| `EnvironmentName` | String | `devl` | `devl`, `test`, `prod` | Environment name, applied as the `Environment` tag |
| `VpcNameTag` | String | — (required) | Starts with a letter; letters, digits and hyphens only | Name prefix for the VPC and related resources |
| `VPCCidrBlock` | String | `10.0.0.0/16` | IPv4 CIDR format (e.g., `10.0.0.0/16`) | CIDR block for the VPC |
| `EnableDnsHostnames` | String | `true` | `true`, `false` | Enables DNS support and DNS hostnames in the VPC |
| `EnableInternetGateway` | String | `true` | `true`, `false` | Creates an Internet Gateway and attaches it to the VPC |

**Note:** `VPCCidrBlock` is checked for CIDR format only. Choose a range that does not overlap with networks you plan to peer or connect to.

## Outputs

- `VPCId` — VPC ID
- `VPCCidr` — VPC CIDR block
- `InternetGatewayId` — Internet Gateway ID (returned only when `EnableInternetGateway` is `true`)

## Usage

### 1. Upload the Template to S3

```bash
aws s3 cp cloudformation/template.yaml s3://your-cfn-bucket/cloudformation/template.yaml
```

### 2. Reference from a Parent Stack

In your parent/root CloudFormation template:

```yaml
VpcNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/cloudformation/template.yaml
    Parameters:
      EnvironmentName: !Ref EnvironmentName
      VpcNameTag: !Ref VpcNameTag
      VPCCidrBlock: 10.0.0.0/16
      EnableDnsHostnames: 'true'
      EnableInternetGateway: 'true'
    Tags:
      - Key: Environment
        Value: !Ref EnvironmentName

Outputs:
  VpcId:
    Value: !GetAtt VpcNestedStack.Outputs.VPCId
  VpcCidr:
    Value: !GetAtt VpcNestedStack.Outputs.VPCCidr
```

### 3. Deploy Using AWS CLI

#### Option A: VPC with Internet Gateway (default)

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name cfn-nested-vpc-stack \
  --parameter-overrides \
    EnvironmentName=devl \
    VpcNameTag=myproject \
    VPCCidrBlock=10.0.0.0/16 \
    EnableDnsHostnames=true \
    EnableInternetGateway=true \
  --region us-east-1
```

#### Option B: Isolated VPC (no Internet Gateway)

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name cfn-nested-vpc-isolated \
  --parameter-overrides \
    EnvironmentName=devl \
    VpcNameTag=myproject-isolated \
    VPCCidrBlock=10.1.0.0/16 \
    EnableInternetGateway=false \
  --region us-east-1
```

#### Validate the Template

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
```

## Resource Naming Convention

| Resource | Name / Tag |
| ----------- | ------------- |
| VPC | `{VpcNameTag}-vpc` |
| Internet Gateway | `{VpcNameTag}-vpc-igw` |
| All resources | `Environment` tag set from `EnvironmentName` |

Example: `VpcNameTag=myproject` in `devl` produces a VPC named `myproject-vpc` and an Internet Gateway named `myproject-vpc-igw`.

## Best Practices Implemented

- ✅ DNS support and DNS hostnames enabled by default
- ✅ Internet Gateway is optional, for isolated VPC deployments
- ✅ Consistent `Name` and `Environment` tagging
- ✅ Parameter validation for name prefix, environment and CIDR format
- ✅ Outputs for cross-stack references from parent stacks

## License

MIT
