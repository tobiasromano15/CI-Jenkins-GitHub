Report: Analysis of Jenkins Pipeline Workflow
Overview
This Jenkins pipeline automates the build, test, and deployment process for the Customers API application hosted on a GitHub repository. It incorporates parallelized testing, software composition analysis (SCA), deployment to a local environment, and post-deployment API testing.

Breakdown of Pipeline Stages
1. Agent and Environment Configuration
Agent: The pipeline executes on any available agent (agent any).
Environment Variables:
APP_VERSION: Dynamically generates the application version using the build ID (1.0.$BUILD_ID).
SNYK_TOKEN: Retrieves the API token for Snyk, a security scanning tool, using Jenkins credentials management.
2. Stage: Checkout
Purpose: Clones the main branch of the Customers API repository from GitHub.
Details:
Repository URL: https://github.com/Veverita-Engineering/Customers-API.
Disables polling and changelog generation for the checkout.
3. Stage: Build
Purpose: Compiles and packages the application using Maven.
Steps:
Verifies the Maven Wrapper (./mvnw) version.
Executes clean package with the following flags:
-DskipTests: Skips tests during the build.
-Drevision=$APP_VERSION: Embeds the dynamically created application version in the build process.
Post Action:
Archives the resulting JAR files located in the target/ directory and generates a fingerprint for traceability.
4. Stage: Parallel Stage
Purpose: Executes unit tests and software composition analysis (SCA) in parallel to reduce build time.
Fail-Fast Mechanism: Ensures the pipeline stops parallel execution immediately if one task fails.
4.1. Sub-Stage: Test
Steps:
Runs unit tests using Maven (./mvnw test).
Post Action:
Always publishes JUnit test reports from target/surefire-reports/*.xml.
4.2. Sub-Stage: SCA (Software Composition Analysis)
Steps:
Executes ./mvnw snyk:test to perform dependency vulnerability scans using Snyk.
Checks the exit code:
If non-zero, marks the build as unstable, indicating security vulnerabilities were detected.
5. Stage: Deploy
Purpose: Deploys the packaged application to a local environment via SSH.
Steps:
Uses an SSH private key (labuser-ssh-key) from Jenkins credentials for secure authentication.
Executes the following actions:
Transfers the JAR file to the target server (localhost on port 2201).
Stops any running instance of the application (pkill -9 java).
Starts the new application instance using java -jar.
Environment Details:
Application runs on port 8081 with temporary files stored in /home/labsuser/.tmp.
6. Stage: API Tests
Purpose: Validates the deployed application using API tests.
Steps:
Uses Node.js (installed as node-lts) for API testing.
Retries the test execution up to 3 times to account for transient issues.
Runs Newman (a CLI tool for Postman) to execute API test scripts:
CustomersAPI.json: The test collection.
baseUrl and version: Environment variables passed to the test, pointing to the deployed application and its version.



Summary of Key Features
Dynamic Application Versioning: Embeds version information in the build, ensuring traceability.
Parallelization: Runs unit tests and security scans simultaneously to optimize performance.
Security Scanning: Integrates Snyk to detect vulnerabilities in dependencies.
Automated Deployment: Ensures zero-downtime deployment to a local environment.
Post-Deployment Testing: Validates functionality and stability of the deployed API with automated tests.
Observations
Efficiency: The pipeline leverages parallel execution and retries for robust and quick feedback.
Traceability: Archived artifacts and test reports provide a detailed audit trail for builds.
Security: The inclusion of SCA ensures dependency vulnerabilities are caught early in the pipeline.
Automation: Automates key development lifecycle stages, from code checkout to deployment and testing.
This pipeline demonstrates a comprehensive DevOps approach, ensuring high-quality application builds while maintaining security and efficiency.
