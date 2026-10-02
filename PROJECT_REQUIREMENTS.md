# Project Requirements

## Project contents

This folder is a clone of the provided Movie Picture Pipeline starter repository. It contains the React frontend, Flask backend, Terraform infrastructure, and four GitHub Actions workflows:

- Frontend CI: pull requests to `main` that change `starter/frontend/**`, plus manual runs.
- Frontend CD: pushes to `main` that change `starter/frontend/**`, plus manual runs.
- Backend CI: pull requests to `main` that change `starter/backend/**`, plus manual runs.
- Backend CD: pushes to `main` that change `starter/backend/**`, plus manual runs.

CI runs lint and tests in parallel, then builds the Docker image only after both succeed. CD does the same checks, pushes an image tagged with the triggering commit SHA to ECR, and deploys that image with Kustomize to EKS.

## Required to run CI

No AWS resources or credentials are needed for the pull-request CI workflows. GitHub Actions needs permission to read the repository and the repository needs its default branch named `main`.

## Required to run deployments

1. Create a GitHub repository under your account or organization, set its default branch to `main`, and push this project there. The current `origin` remote points to the public Udacity starter repository; replace it with your new repository before pushing.
2. Have an AWS account. Use `us-east-1`: the supplied Terraform provider, availability zones, and VPC endpoint definitions are currently hard-coded to this region. Creating EKS and its worker nodes can incur charges.
3. From `setup/terraform`, install/use Terraform 1.3.9, initialize Terraform, and apply the provided configuration. Record its `frontend_ecr`, `backend_ecr`, and `cluster_name` outputs.
4. The supplied Terraform configuration creates an EKS access entry for `github-action-user` and associates the cluster admin access policy. With this configuration, `setup/init.sh` is not required; it is retained for the legacy `aws-auth` mapping workflow or clusters that do not use the Terraform-managed access entry.
5. Add these GitHub repository Actions secrets:
   - `AWS_ACCESS_KEY_ID`: access key for `github-action-user`.
   - `AWS_SECRET_ACCESS_KEY`: matching secret access key.
6. Add these GitHub repository Actions variables:
   - `AWS_REGION`: set to `us-east-1` for the supplied Terraform configuration.
   - `EKS_CLUSTER_NAME`: Terraform `cluster_name` output.
   - `FRONTEND_ECR_REPOSITORY`: full Terraform `frontend_ecr` repository URL.
   - `BACKEND_ECR_REPOSITORY`: full Terraform `backend_ecr` repository URL.
   - `BACKEND_API_URL`: public backend LoadBalancer URL, without `/movies`; required for frontend CD.

The AWS user needs permission to authenticate to ECR, push images to both repositories, describe the EKS cluster, and deploy the Kubernetes resources. The supplied Terraform policy grants the workflow user ECR/EKS/EC2 permissions and the EKS access policy grants cluster access. Store access keys only as GitHub Actions secrets; never commit them to the repository.

Frontend CI uses `REACT_APP_MOVIE_API_URL=http://localhost:5000`, matching the rubric and starter notes. Frontend CD reads the `BACKEND_API_URL` repository variable and fails early if it is missing, preventing a deployment that cannot reach the backend. Deploy the backend first, obtain its public address from `kubectl get service backend`, set `BACKEND_API_URL` to `http://<external-address>`, and then deploy the frontend.

## Local tools

For local app work: Docker, Node.js version from `starter/frontend/.nvmrc`, Python 3.10, and Pipenv. For infrastructure/deploy setup: AWS CLI, Terraform 1.3.9, `kubectl`, and Kustomize. These tools are installed on GitHub-hosted runners by the workflows where needed; AWS resources and GitHub settings are still user-provided.

## Teardown

After deployment verification, run `terraform destroy` from `setup/terraform` to avoid ongoing AWS charges. Review the plan carefully before approving the destroy operation.