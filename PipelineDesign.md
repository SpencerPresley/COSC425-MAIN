# Project Architecture Design Document

This document outlines the architecture and workflow for a dynamic content generation and deployment system using AWS services, Docker, and a Next.js application hosted on Vercel. The system automates the generation of JSON data via a Python pipeline, updates the data on an Amazon S3 bucket, and dynamically updates content on a statically generated Next.js website.

## Sequence of Operations

1. **Python Pipeline Execution**: Triggered both on a schedule and on-demand.
2. **Data Storage in Amazon S3**: Generated JSON data is stored.
3. **Dynamic Site Update on Vercel**: Next.js site updates based on the new JSON data.
4. **Security and Authentication**: Ensure secure operations across all components.
5. **CI/CD Integration**: Automate deployment and updates.

---

### 1. Python Pipeline Execution

- **Docker Containerization**: The Python pipeline, including all dependencies, is containerized using Docker. This ensures consistency across development, testing, and production environments.
- **AWS Lambda Execution**: The Docker container is hosted on Amazon Elastic Container Registry (ECR) and executed by AWS Lambda. Lambda supports container images, enabling complex applications with heavy dependencies.
- **Scheduled Triggers via Amazon EventBridge**: Regular execution of the Python pipeline is scheduled using Amazon EventBridge, ensuring data is refreshed at predefined intervals.
- **On-Demand Execution via API Gateway**: An API Gateway endpoint allows for manual triggering of the pipeline. This is ideal for scenarios where an administrator needs to refresh the data outside the regular schedule.

### 2. Data Storage in Amazon S3

- **S3 Bucket Configuration**: Output JSON data from the Python pipeline is stored in a designated S3 bucket. This bucket is configured for public read access to serve as a data endpoint for the Next.js application.
- **IAM Roles and Policies**: Proper IAM roles and policies are established to allow the Lambda function write access to the S3 bucket, ensuring secure and scoped access.

### 3. Dynamic Site Update on Vercel

- **Incremental Static Regeneration (ISR)**: Utilizes Next.js's ISR feature for dynamic content updates without needing to rebuild and redeploy the entire site.
- **On-Demand Revalidation**: An API route within the Next.js application (/api/revalidate) allows for on-demand revalidation of specific pages, triggered by the Python pipeline after updating the JSON in S3.
- **Secure Endpoint**: The revalidation endpoint is secured with a secret token to prevent unauthorized access.

### 4. Security and Authentication

- **Endpoint Security**: API Gateway and the Next.js revalidation endpoint use secret tokens for authentication, ensuring that only authorized requests trigger the pipeline and content revalidation.
- **S3 Bucket Policy**: The S3 bucket is secured to prevent unauthorized access or modification of the data, while still allowing public read access for the JSON data.

### 5. CI/CD Integration

- **Automated Docker Builds**: CI/CD pipelines (e.g., GitHub Actions) are set up to automatically build and push updated Docker images to ECR whenever the Python codebase changes.
- **Automated Lambda Deployment**: The CI/CD pipeline includes steps to update the AWS Lambda function with the latest Docker image from ECR.
- **Vercel Integration**: The Next.js site is connected to its Git repository. Commits to the main branch trigger automatic rebuilds and deployments on Vercel, leveraging Vercel's native GitHub integration for seamless CI/CD.

---

## Logging / Monitoring - Something to consider in the future.

- **AWS Lambda and S3**: Use AWS CloudWatch for basic logging of Lambda function executions and S3 access. This setup automatically captures logs without additional configuration, providing insight into the performance and any errors of your Python pipeline and data storage.

- **Docker Logs**: Have Docker container log important events and errors to stdout/stderr, AWS Lambda will capture this and send it to CloudWatch by default.

- **Next.js on Vercel**: Utilize Vercel's dashboard for logs related to the Next.js application's deployment and runtime errors. Vercel automatically captures and displays these logs.

- **Monitoring**: Set up basic alerts in CloudWatch for error monitoring and Lambda execution errors. Will keep us notified of critical errors. 

