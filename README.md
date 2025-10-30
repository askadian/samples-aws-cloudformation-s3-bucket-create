# S3 CloudFormation + GitHub Actions

This repository stores an AWS CloudFormation template that provisions an S3 bucket and two GitHub Actions workflows to create (deploy) and delete the CloudFormation stack manually.

Important notes:
- Workflows are designed to use OIDC role assumption (recommended). See "Configure OIDC & Role" below.
- If you prefer static credentials, you can set AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as repository secrets (not recommended).
- The CloudFormation template is stored at `s3-bucket-template.yaml` (provided as-is).

Quick run (manual):
1. Configure the repository secrets or OIDC role (see below).
2. In the GitHub UI open "Actions" → select "Deploy S3 stack" or "Delete S3 stack" → Run workflow → supply `stack_name` (example: `my-s3-stack`) and `region` (default `us-east-1`) → Run workflow.

Files of interest:
- s3-bucket-template.yaml — CloudFormation template (creates an S3 bucket + bucket policy)
- .github/workflows/create-stack.yml — workflow to deploy the CloudFormation stack (workflow_dispatch)
- .github/workflows/delete-stack.yml — workflow to empty the bucket and delete the CloudFormation stack (workflow_dispatch)

Configure OIDC & Role (recommended):
1. Create an IAM role in the target AWS account with a trust relationship that allows GitHub Actions OIDC provider to assume the role from this repository (or organization). See:
   - https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-your-identity-provider
   - https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html
2. Give the role permissions to manage CloudFormation stacks and S3 operations, for example:
   - `cloudformation:*` (deploy/delete) for the target stack
   - `s3:*` on the bucket or the ability to empty objects before deletion
   - (Optional) `kms:Decrypt` / KMS permissions if using a CMK
3. Add the role ARN as the repository secret `AWS_ROLE_ARN`.

Alternatively (less secure) set these repository secrets:
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_REGION (optional; workflows default to `us-east-1`)

Notes about deletion:
- CloudFormation stack deletion will fail if the S3 bucket is not empty. The delete workflow attempts to compute the bucket name from the stack name and AWS account ID, empty the bucket, and then delete the stack.
- If you use a different bucket naming pattern or external resources, adapt the delete job accordingly.