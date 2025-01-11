# Terraform CI/CD with GitHub Actions: Globally Autoscaling Web Application

This repository demonstrates how to build a CI/CD pipeline using GitHub Actions to automate the deployment of a globally autoscaling web application on Google Cloud Platform (GCP) using Terraform.

**Project Structure:**

The project deploys a simple "Interstellar" themed web application across two regions (`us-central1` and `europe-west2`) using:

*   **Managed Instance Groups (MIGs):** For autoscaling and resilience.
*   **Global HTTPS Load Balancer:** For traffic distribution and SSL termination.
*   **Cloud NAT:** For providing internet access to instances without public IPs.
*   **Google Cloud Storage:** To store the static website files.
*   **Cloud DNS:** To manage the DNS records.

**CI/CD Pipeline:**

The GitHub Actions workflow automates the following:

*   **`terraform init`:** Initializes the Terraform working directory.
*   **`terraform plan`:** Generates a Terraform plan on pull requests. The plan is also added as a comment to the pull request.
*   **`terraform apply`:** Applies the Terraform plan automatically when changes are merged into the `main` branch.

## Prerequisites

Before you begin, make sure you have the following:

1.  **Google Cloud Account:** A Google Cloud account with billing enabled.
2.  **Google Cloud Project:** Create a project in the Google Cloud Console. Note down your Project ID.
3.  **Service Account:**
    *   Create a service account in your Google Cloud project.
    *   Grant the following roles to your service account:
        *   `roles/compute.networkAdmin`
        *   `roles/compute.instanceAdmin.v1`
        *   `roles/compute.securityAdmin`
        *   `roles/storage.admin`
        *   `roles/dns.admin`
        *   `roles/iam.serviceAccountKeyAdmin`
        *   `roles/iam.serviceAccountUser`
    *   Download the service account key as a JSON file and store it securely. You'll need to add it as a GitHub secret.

4.  **Enable APIs:**
    *   Make sure the following APIs are enabled for your project:
        *   Compute Engine API
        *   Cloud DNS API
        *   Cloud Resource Manager API
        *   Identity and Access Management (IAM) API
        *   Service Usage API

5.  **gcloud CLI:** Install and configure the Google Cloud CLI on your local machine. ([https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install))
6.  **Terraform:** Install Terraform (v1.6.6 or later) on your local machine. ([https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/install-cli](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/install-cli))
7.  **GitHub Account:** A GitHub account to fork and clone this repository.
8.  **Cloud DNS:**
    *   Create a public managed DNS zone in Cloud DNS for your domain (e.g., `google-cloud-pocs.dev`).

## Getting Started

1.  **Fork the Repository:**
    *   Fork this repository (`https://github.com/SamPriyadarshi/terraform-ci-cd`) to your own GitHub account.

2.  **Clone the Repository:**

    ```bash
    git clone https://github.com/SamPriyadarshi/terraform-ci-cd.git
    cd terraform-ci-cd
    ```

3.  **Create a Google Cloud Storage Bucket for Terraform State:**

    *   Create a GCS bucket to store your Terraform state remotely. You can use the `gcloud` CLI:

    ```bash
    gsutil mb -b on -l us-central1 gs://<your-unique-bucket-name>
    gsutil versioning set on gs://<your-unique-bucket-name>
    ```

    *   Replace `<your-unique-bucket-name>` with a globally unique name for your bucket (e.g., `your-project-name-tfstate`).

4.  **Update Terraform Backend Configuration:**

    *   Modify the `main.tf` file and update the `backend "gcs"` block with your bucket name:

    ```terraform
    terraform {
      backend "gcs" {
        bucket = "your-unique-bucket-name"  # Replace with your bucket name
        prefix = "terraform/state"
      }
      # ... rest of the configuration ...
    }
    ```

5.  **Create `terraform.tfvars`:**

    *   Create a file named `terraform.tfvars` in the root directory of the project.
    *   Populate it with the required variables. Here's an example:

    ```
    project_id              = "your-gcp-project-id"  # Replace with your Project ID
    bucket_name             = "your-website-bucket-name"  # Replace with a unique bucket name for website files
    dns_zone_name           = "google-cloud-pocs-dev" # Replace with your DNS zone name
    dns_managed_zone_name = "google-cloud-pocs-dev"  # Replace with your DNS managed zone name
    ```

6.  **Add GitHub Secrets:**

    *   In your forked GitHub repository, go to "Settings" -> "Secrets and variables" -> "Actions".
    *   Create the following repository secrets:
        *   **`GCP_CREDENTIALS`:** Paste the entire contents of your service account JSON key file.

7.  **Update the `startup_script.tpl`:**

    *   Modify the `startup_script.tpl` file inside `instance_template` module to update the bucket name. Replace `your-website-bucket-name` with the name of the bucket you created in step 5.

8.  **Push Changes to your Fork:**
    *   Commit and push the changes you made to your forked repository:
    ```bash
    git add .
    git commit -m "Configure Terraform backend and variables"
    git push origin main
    ```

## Workflow Overview

The repository contains a GitHub Actions workflow (`.github/workflows/terraform.yml`) that automates the Terraform deployment process:

*   **On Pull Request:**
    *   `terraform init`
    *   `terraform plan` (saved as an artifact)
    *   Adds a comment to the PR with the plan summary.
*   **On Push to `main`:**
    *   `terraform init`
    *   `terraform apply` (using the saved plan from the pull request, if applicable)

## Triggering the Workflow

1.  **Create a Feature Branch:**
    *   Create a new branch from `main` for your changes:

        ```bash
        git checkout -b my-feature-branch
        ```

2.  **Make Changes:**
    *   Make your desired changes to the Terraform code (e.g., modify website content, change instance types, etc.).

3.  **Commit and Push:**
    *   Commit your changes and push them to your feature branch:

        ```bash
        git add .
        git commit -m "Your commit message"
        git push origin my-feature-branch
        ```

4.  **Create a Pull Request:**
    *   Create a pull request on GitHub from your feature branch to the `main` branch.

5.  **Review Terraform Plan:**
    *   The GitHub Actions workflow will automatically run `terraform init` and `terraform plan`.
    *   Review the plan in the workflow logs and in the PR comment added by the workflow.

6.  **Merge Pull Request:**
    *   Once the plan is approved, merge the pull request into `main`.

7.  **Terraform Apply:**
    *   The GitHub Actions workflow will automatically run `terraform apply` to deploy the changes to your Google Cloud project.

## Accessing the Web Application

After the workflow has successfully completed:

*   You can find the external IP address of the load balancer in the workflow logs or in the Google Cloud Console (Compute Engine -> Load balancing).
*   You can also access the application using the DNS name configured in Cloud DNS (e.g., `galactic-empire.google-cloud-pocs.dev`).

## Cleaning Up

To delete the resources created by Terraform, run the following command **locally**:

```bash
terraform destroy
