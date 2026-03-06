# AWS Deployment with OIDC Authentication

This reusable workflow provides secure, keyless AWS deployments using OpenID Connect (OIDC) authentication. No long-lived AWS credentials needed!

## 🔐 Why OIDC?

Traditional AWS deployments require storing IAM access keys as GitHub secrets. OIDC eliminates this security risk by using short-lived tokens that GitHub automatically generates for each workflow run.

**Benefits:**
- ✅ No long-lived credentials to rotate or leak
- ✅ Fine-grained IAM permissions per repository
- ✅ Automatic token expiration after workflow completion
- ✅ AWS CloudTrail shows GitHub context (repo, branch, actor)
- ✅ Industry best practice (2024+)

## 🚀 Supported Deployment Targets

This workflow supports multiple AWS services:

- **Amazon ECS**: Deploy containerized applications
- **AWS Lambda**: Deploy serverless functions
- **Amazon S3 + CloudFront**: Deploy static sites/SPAs
- **Amazon ECR**: Build and push container images

## 📋 Prerequisites

### 1. Configure AWS IAM Identity Provider

Set up GitHub OIDC in your AWS account (one-time setup):

```bash
# Using AWS CLI
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

### 2. Create IAM Role

Create an IAM role that GitHub can assume:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

**Security Tip**: Use `StringLike` conditions to restrict by branch/environment:
```json
"token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main"
```

### 3. Attach IAM Permissions

Attach policies to the role based on your deployment target:

**ECS Deployment:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```

**Lambda Deployment:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:UpdateFunctionCode",
        "lambda:UpdateFunctionConfiguration",
        "lambda:GetFunction"
      ],
      "Resource": "arn:aws:lambda:us-east-1:YOUR_ACCOUNT_ID:function:YOUR_FUNCTION"
    }
  ]
}
```

**S3 + CloudFront Deployment:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET",
        "arn:aws:s3:::YOUR_BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DISTRIBUTION_ID"
    }
  ]
}
```

## 📖 Usage Examples

### Deploy Container to ECS

```yaml
name: Deploy Production
on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: ghndrx/github-actions-library/reusable-workflows/deploy-aws-oidc.yml@main
    with:
      aws-region: us-east-1
      environment: production
      ecr-repository: my-app
      ecs-cluster: production-cluster
      ecs-service: my-app-service
      run-smoke-tests: true
      smoke-test-command: 'curl -f https://api.example.com/health'
    secrets:
      aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
      slack-webhook: ${{ secrets.SLACK_WEBHOOK }}
```

### Deploy Lambda Function

```yaml
name: Deploy Lambda
on:
  release:
    types: [published]

jobs:
  deploy:
    uses: ghndrx/github-actions-library/reusable-workflows/deploy-aws-oidc.yml@main
    with:
      aws-region: us-west-2
      environment: production
      lambda-function: my-function
      working-directory: lambda/
    secrets:
      aws-role-arn: ${{ secrets.AWS_LAMBDA_ROLE_ARN }}
```

### Deploy Static Site to S3 + CloudFront

```yaml
name: Deploy Frontend
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  deploy:
    needs: build
    uses: ghndrx/github-actions-library/reusable-workflows/deploy-aws-oidc.yml@main
    with:
      aws-region: us-east-1
      environment: production
      s3-bucket: my-website-bucket
      cloudfront-distribution: E1234567890ABC
    secrets:
      aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
```

### Multi-Environment Deployment Matrix

```yaml
name: Deploy All Environments
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options:
          - dev
          - staging
          - production

jobs:
  deploy:
    uses: ghndrx/github-actions-library/reusable-workflows/deploy-aws-oidc.yml@main
    with:
      aws-region: us-east-1
      environment: ${{ inputs.environment }}
      ecr-repository: my-app
      ecs-cluster: ${{ inputs.environment }}-cluster
      ecs-service: my-app-service
    secrets:
      aws-role-arn: ${{ secrets[format('AWS_ROLE_ARN_{0}', inputs.environment)] }}
```

## 🎯 Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `aws-region` | ✅ | - | AWS region |
| `environment` | ✅ | - | Deployment environment (dev/staging/production) |
| `working-directory` | ❌ | `.` | Working directory for deployment scripts |
| `ecr-repository` | ❌ | - | ECR repository name (for container deployments) |
| `ecs-cluster` | ❌ | - | ECS cluster name |
| `ecs-service` | ❌ | - | ECS service name |
| `lambda-function` | ❌ | - | Lambda function name |
| `s3-bucket` | ❌ | - | S3 bucket for static sites |
| `cloudfront-distribution` | ❌ | - | CloudFront distribution ID |
| `run-smoke-tests` | ❌ | `false` | Run smoke tests after deployment |
| `smoke-test-command` | ❌ | `npm run test:smoke` | Smoke test command |
| `deployment-timeout` | ❌ | `300` | Deployment timeout (seconds) |

## 🔑 Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `aws-role-arn` | ✅ | AWS IAM Role ARN for OIDC authentication |
| `slack-webhook` | ❌ | Slack webhook for deployment notifications |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `deployment-status` | Deployment status (success/failure) |
| `deployment-timestamp` | Deployment timestamp (ISO 8601) |
| `image-tag` | Container image tag (if applicable) |

## 🛡️ Security Best Practices

1. **Use environment protection rules**: Configure required reviewers for production deployments in GitHub
2. **Restrict IAM role assumption**: Use `StringLike` conditions in trust policy to limit by branch
3. **Principle of least privilege**: Grant only necessary permissions to the IAM role
4. **Enable AWS CloudTrail**: Monitor all API calls with GitHub context
5. **Use GitHub environments**: Separate secrets per environment (dev/staging/prod)

## 🔍 Troubleshooting

### "Not authorized to perform sts:AssumeRoleWithWebIdentity"

**Cause**: Trust policy doesn't allow your repository to assume the role.

**Fix**: Check the `Condition` block in your IAM role's trust policy:
```json
"StringLike": {
  "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
}
```

### "Error: Credentials could not be loaded"

**Cause**: OIDC provider not configured in AWS account.

**Fix**: Run the AWS CLI command from Prerequisites step 1.

### ECS Deployment Hangs

**Cause**: New task definition failed health checks or can't pull image.

**Fix**: 
- Check ECS task logs in CloudWatch
- Verify ECR permissions in IAM policy
- Check VPC/security group configuration

## 📚 References

- [AWS OIDC GitHub Guide](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [GitHub Actions Security Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [AWS IAM Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)

---

**Need help?** Open an issue or check the [documentation](../README.md).
