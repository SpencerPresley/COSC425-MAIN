## Project Architecture and Workflow Overview

This document outlines the architecture and automated workflow designed to dynamically generate and update content on a Next.js website, leveraging AWS services and Docker for backend processing.

### Dockerized Python Pipeline

- The Python pipeline, along with all its dependencies, is containerized within a Docker image. This ensures consistency across different environments including development, testing, and production.

### Scheduled and Manual Python Pipeline Execution

- **Scheduled Execution** -> Dependent on WoS API: Amazon EventBridge is configured to trigger an AWS Lambda function at specific intervals, typically once a month. This Lambda function executes the Docker container, running the Python pipeline.
- **Manual Execution** -> Admin uploads new WoS data: An API Gateway endpoint is established for manual triggering of the pipeline. Upon receiving a request, this endpoint directly invokes the Lambda function, facilitating on-demand updates.

### Data Processing and Site Update Trigger

- When triggered, the Lambda function executes the pipeline within the Docker container. The pipeline processes data, generates a new JSON file, and uploads this file to an Amazon S3 bucket.
- Subsequently, a request is made to the hosted Next.js site on Vercel to trigger an update, ensuring the site content reflects the most recent data.

### CI/CD Automation

- **Docker Image Updates**: A CI/CD pipeline (GitHub Actions) automates the build and update process for the Docker image. After a commit, the pipeline builds a new Docker image and pushes it to Amazon ECR. The AWS Lambda function is configured to use the latest image, keeping the pipeline current.
- **Next.js Site Updates**: The Next.js site is linked to the front end repository, with Vercel managing automatic deployments. Commits to the repository prompt Vercel to rebuild and redeploy the site. Though these actions do not directly initiate the Python pipeline, the site may refetch the JSON from S3 to update static pages accordingly.

### Static Site Generation Strategy

- Features like Incremental Static Regeneration (ISR) from Next.js are utilized to dynamically refresh the site's content based on the latest JSON data from S3, allows for the updating of content without needing to redo the entire build process via a script or manually.