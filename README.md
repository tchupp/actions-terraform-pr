# GitHub Actions: Terraform PR

GitHub Action for adding `terraform plan` output as a PR comment.

**[Shut up and show me the copy/paste](#examples)**

## Usage

This action can be used as follows:
```yaml
      - uses: tchupp/terraform-pr@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          default-branch: "main"
          path: "terraform"
```

### Assumptions

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

### Inputs

#### github-token

**Required**. This can just be set to `${{ secrets.GITHUB_TOKEN }}`

#### default-branch

**Optional**. Set this to the default branch for your repo, ex `main` or `master`.  
This will default to `main` if not specified.

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
      - '.github/workflows/**'

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
          default-branch: "main"

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'schedule')
        run: |
          terraform apply -auto-approve
```

### Simple Setup - directory

This setup assumes you have your terraform files in a subdirectory in your repo, and you only have a single workspace:
```bash
$ tree
.
├── app
│   ├── index.ts
└── terraform
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
      - '.github/workflows/**'

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
          default-branch: "main"
          path: "terraform"

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'schedule')
        run: |
          cd terraform
          terraform apply -auto-approve
```
