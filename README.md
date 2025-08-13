# Hashicat AWS Project

Hashicat is a Terraform project that deploys a simple "Meow World" web application on AWS. It is designed for use in HashiCorp workshops to demonstrate Terraform's capabilities, including infrastructure provisioning, integration with Sentinel for policy as code, and basic application deployment.

The project provisions a complete, self-contained environment including a VPC, subnet, security group, and an EC2 instance that serves the web application.

## Prerequisites

Before you begin, ensure you have the following installed and configured:

*   **Terraform**: Version 1.0 or later.
*   **AWS Account**: An active AWS account with credentials configured for Terraform to use. This typically means having your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables set.
*   **Git**: To clone this repository.

## Usage

1.  **Clone the repository:**
    ```sh
    git clone <repository-url>
    cd hashicat-aws
    ```

2.  **Configure your deployment:**
    Create a `terraform.tfvars` file by copying the example file:
    ```sh
    cp terraform.tfvars.example terraform.tfvars
    ```
    Edit `terraform.tfvars` to provide a value for the `prefix` variable. This prefix is used to name your resources to avoid conflicts with other deployments. You can also customize other variables in this file as needed.

    ```hcl
    # terraform.tfvars
    prefix = "my-hashicat-app"
    ```

3.  **Initialize Terraform:**
    Run `terraform init` to download the necessary providers.
    ```sh
    terraform init
    ```

4.  **Plan and Apply:**
    Run `terraform plan` to review the resources that will be created.
    ```sh
    terraform plan
    ```
    If the plan is acceptable, apply the configuration to deploy the infrastructure.
    ```sh
    terraform apply
    ```
    Terraform will prompt you for confirmation before proceeding. Type `yes` to approve.

After the apply is complete, Terraform will output the URL and IP address of your new web application.

## Infrastructure

This project creates the following AWS resources:

*   `aws_vpc`: A dedicated Virtual Private Cloud (VPC) for the application.
*   `aws_subnet`: A public subnet within the VPC.
*   `aws_internet_gateway`: To provide internet access to the subnet.
*   `aws_route_table`: To route traffic from the subnet to the internet gateway.
*   `aws_security_group`: A firewall to control traffic to the EC2 instance, allowing HTTP, HTTPS, and SSH access.
*   `aws_instance`: An EC2 instance (default: `t2.micro`) running Ubuntu 22.04.
*   `aws_eip`: An Elastic IP address to provide a static public IP for the EC2 instance.
*   `aws_key_pair`: An SSH key pair for accessing the instance. The private key is generated and saved as `<prefix>-ssh-key.pem`.
*   `null_resource`: A resource to run provisioners that install Apache and deploy the web application on the EC2 instance.

## Inputs

The following input variables can be configured in your `terraform.tfvars` file:

| Name            | Description                                                                                             | Type   | Default       | Required |
| --------------- | ------------------------------------------------------------------------------------------------------- | ------ | ------------- | :------: |
| `prefix`        | A unique prefix included in the name of most resources.                                                 | `string` | -             |   Yes    |
| `region`        | The AWS region where the resources will be created.                                                     | `string` | `us-east-1`   |    No    |
| `address_space` | The CIDR block for the VPC.                                                                             | `string` | `10.0.0.0/16` |    No    |
| `subnet_prefix` | The CIDR block for the subnet.                                                                          | `string` | `10.0.10.0/24`|    No    |
| `instance_type` | The EC2 instance type to use.                                                                           | `string` | `t2.micro`    |    No    |
| `admin_username`| Administrator user name for mysql. (Note: MySQL is not installed in this configuration).                | `string` | `hashicorp`   |    No    |
| `height`        | The height of the placeholder image in pixels.                                                          | `string` | `400`         |    No    |
| `width`         | The width of the placeholder image in pixels.                                                           | `string` | `600`         |    No    |
| `placeholder`   | The image-as-a-service URL to use for the cat image.                                                    | `string` | `placebeard.it` |    No    |

## Outputs

| Name         | Description                                        |
| ------------ | -------------------------------------------------- |
| `catapp_url` | The public DNS URL of the deployed web application.  |
| `catapp_ip`  | The public IP address of the deployed web application. |

## Sentinel Policies

This repository includes [Sentinel](https://www.hashicorp.com/sentinel) policies for policy-as-code, located in the `policies/` directory. These policies are intended to be used with Terraform Cloud or Terraform Enterprise.

*   **`enforce-mandatory-tags`**: Requires all EC2 instances to have `Environment` and `Department` tags. The `Environment` tag must have a value of `dev`, `qa`, or `prod`.
*   **`restrict-deployment-cost`**: Ensures that the estimated monthly cost increase for a deployment is less than $100.

These policies are configured with a `soft-mandatory` enforcement level, which means violations will produce a warning but will not stop a Terraform run.

## Exercises

The `exercises/` directory contains additional Terraform files intended for use in guided workshops. These are not part of the main deployment.

## License

This project is licensed under the [MIT License](LICENSE).
