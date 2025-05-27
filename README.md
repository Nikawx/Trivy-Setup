Trivy Setup on Azure DevOps Pipeline
=======================================

This guide explains how to set up **Trivy** in an Azure DevOps Pipeline to scan Docker images, filesystems, and Terraform configurations for vulnerabilities and misconfigurations.

* * * * *

First Steps
--------------

1.  **Create a directory** named `.azure-pipelines` in the root of your repository.

2.  **Add the `trivy.yml` pipeline file** to that directory.

> You can find a reference example here: [Trivy Setup GitHub Repo](https://github.com/Nikawx/Trivy-Setup/tree/main)

* * * * *

Creating the Pipeline in Azure DevOps
----------------------------------------

1.  Go to **Pipelines** in your Azure DevOps project.

2.  Click **"New pipeline"**.

3.  Select **Azure Repos Git (YAML)**.

4.  Choose your project and repository.

5.  Select:

    -   **Existing Azure Pipelines YAML file**

    -   Branch: `main` (or your desired branch)

    -   Path: `/.azure-pipelines/trivy.yml`

6.  Click **"Run"** to create the pipeline.

* * * * *

`trivy.yml` Pipeline Steps
-----------------------------

### Build Docker Image

yaml

CopierModifier

`- script: |
    echo "🐳 Building vulnerable Docker image"
    docker build -t vuln-app ./app # Change directory if needed
  displayName: 'Build Docker image'`

### 🔍 Trivy Scan: Docker Image

yaml

CopierModifier

`- script: |
    echo " Scanning Docker image with Trivy (SARIF)"
    mkdir -p $(reportDir)
    trivy image --ignore-unfixed --severity HIGH,CRITICAL,MEDIUM --format sarif --output $(reportDir)/trivy-image.sarif vuln-app
  displayName: 'Trivy Scan: Docker Image'`

Scans the Docker image `vuln-app` built from the `./app` directory for **HIGH**, **CRITICAL**, and **MEDIUM** vulnerabilities.

* * * * *

### Trivy Scan: Filesystem

yaml

CopierModifier

`- script: |
    echo " Scanning filesystem with Trivy (SARIF)"
    trivy fs --ignore-unfixed --severity HIGH,CRITICAL,MEDIUM --format sarif --output $(reportDir)/trivy-fs.sarif ./app
  displayName: 'Trivy Scan: Filesystem'`

Scans all files in `./app` for vulnerabilities. The results are saved in SARIF format.

* * * * *

### ☁️ Trivy Scan: Terraform (Infrastructure as Code)

yaml

CopierModifier

`- script: |
    echo " Scanning Terraform configuration with Trivy (SARIF)"
    trivy config --severity HIGH,CRITICAL,MEDIUM --format sarif --output $(reportDir)/trivy-config.sarif ./infra
  displayName: 'Trivy Scan: Terraform (IaC)'`

Scans Terraform files located in `./infra` for misconfigurations and outputs the result in SARIF format.

* * * * *

 Reporting
------------

SARIF reports are automatically converted to **HTML reports** and made accessible under the **"Extensions"** tab for each pipeline run/commit.

* * * * *

 Ignoring Vulnerabilities
---------------------------

To ignore specific CVEs or types of findings:

1.  Add a `.trivyignore` file in the **root of your repository**.

2.  List CVEs or ignore rules in the file.

Example:

CopierModifier

`CVE-2021-12345
CVE-2022-67890`

> Trivy will skip the listed vulnerabilities during scans.
