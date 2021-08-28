# GitHub Actions: Terraform PR

GitHub Action for adding `terraform plan` output as a PR comment.

**[Shut up and show me the copy/paste](#examples)**

## Usage

This action can be used as follows:
```yaml
      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          apply-branch: "main"
```

### Assumptions

The most common cause for issues involves a mismatch in expectations.  
Please read below to make sure your setup aligns with the assumptions made by this action.

#### Setup

This action expects that the repo has been checked out and terraform has been set up with the correct version.  
The following snippet from an example job, [which can be found below](#simple-setup).
```yaml
...
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: ~1.0.0
          cli_config_credentials_token: ${{ secrets.TF_TOKEN }}
...
```

#### Terraform Workspaces

Some teams choose to have multiple sets of terraform configuration in one git repository.  
Let's say your repo looked like this:
```bash
$ tree
.
├── aws-route53
│   ├── main.tf
│   └── versions.tf
└── aws-vpc
    ├── main.tf
    └── versions.tf

2 directories, 4 files
```

If you had `development`, `staging`, and `production` workspaces for both sets of config files, you would have 6 workspaces total.

This action expects that your terraform workspace is uniquely named, at least inside your repository.
That means the workspace names would be:
- `aws-route53-development`
- `aws-route53-staging`
- `aws-route53-production`
- `aws-vpc-development`
- `aws-vpc-staging`
- `aws-vpc-production`

This naming scheme is common when using a backend such as Terraform Cloud.

### Inputs

#### github-token

**Required**. This can just be set to `${{ secrets.GITHUB_TOKEN }}`

#### apply-branch

**Optional**. Set this to the default branch for your repo, ex `main` or `master`.  
Leave blank to disable auto-apply.

#### path

**Optional**. Set this to the path to your terraform configuration files.  
This will default to the top level directory if not specified.

## Examples

### Simple Setup

This setup assumes you have your terraform files at the top level of your repo, and you only have a single workspace:
```bash
$ tree
.
├── main.tf
└── versions.tf

0 directories, 2 files
```

In your `.github/workflows/terraform.yml` file you would have:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'

jobs:
  terraform:
    name: "Terraform"
    runs-on: ubuntu-latest
    env:
      TF_WORKSPACE: "default"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: ~1.0.0
          cli_config_credentials_token: ${{ secrets.TF_TOKEN }}

      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          apply-branch: "main"

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: |
          terraform apply -auto-approve
```

### Single directory with a single workspace

This setup assumes you have your terraform files in a subdirectory in your repo, and you only have a single workspace:
```bash
$ tree
.
├── app
│   ├── index.ts
└── terraform
    ├── main.tf
    └── versions.tf

2 directories, 3 files
```

In your `.github/workflows/terraform.yml` file you would have:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'

jobs:
  terraform:
    name: "Terraform"
    runs-on: ubuntu-latest
    env:
      TF_WORKSPACE: "default"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: ~1.0.0
          cli_config_credentials_token: ${{ secrets.TF_TOKEN }}

      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          apply-branch: "main"
          path: "terraform"
```

### Single directory with multiple workspaces

This setup assumes you have your terraform files in a subdirectory in your repo, and you only have a single workspace:
```bash
$ tree
.
├── app
│   ├── index.ts
└── terraform
    ├── main.tf
    └── versions.tf

2 directories, 3 files
```

In your `.github/workflows/terraform.yml` file you would have:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'

jobs:
  terraform:
    name: "Terraform"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env:
          - development
          - staging
          - production
    env:
      TF_WORKSPACE: "${{ matrix.env }}"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: ~1.0.0
          cli_config_credentials_token: ${{ secrets.TF_TOKEN }}

      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          apply-branch: "main"
          path: "terraform"
```

### Manual Terraform Apply

In some situation, you want to control the `terraform apply` yourself.

In your `.github/workflows/terraform.yml` file you would have:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    paths:
      - '**.tf'
      - '.github/workflows/terraform.yml'

jobs:
  terraform:
    name: "Terraform"
    runs-on: ubuntu-latest
    env:
      TF_WORKSPACE: "default"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: ~1.0.0
          cli_config_credentials_token: ${{ secrets.TF_TOKEN }}

      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: |
          terraform apply -auto-approve
```