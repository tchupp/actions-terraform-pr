# GitHub Actions: Terraform PR

GitHub Action for adding `terraform plan` output as a PR comment

## Usage

### Simple Setup

This setup assumes you have your terraform files at the top level of your repo like so:
```bash
$ tree
.
├── main.tf
└── versions.tf

0 directories, 2 files
```
$ cat .github/workflows/terraform.yml

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
        run: terraform apply -auto-approve
```
