You can run and manage **Azure Pipelines** directly from VS Code using the **Azure Pipelines extension** and the **YAML pipeline files** in your repo. Here’s a step-by-step guide:

---

## 🔧 Prerequisites

1. **Install VS Code** (latest version).
2. **Install Azure Pipelines extension**

   * Go to Extensions → Search for `Azure Pipelines` → Install.
3. **Install Azure CLI** (optional but recommended).

   ```bash
   az login
   ```
4. **GitHub/Azure DevOps repo** with your project code.

---

## 🚀 Steps to Run Azure Pipeline from VS Code

### 1. Create or Edit Your Pipeline YAML

* In VS Code, create a file named `azure-pipelines.yml` at the root of your repo.
* Example:

  ```yaml
  trigger:
    - main

  pool:
    vmImage: 'ubuntu-latest'

  steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
      displayName: 'Install Node.js'

    - script: |
        npm install
        npm run build
      displayName: 'Build app'

    - script: npm test
      displayName: 'Run tests'
  ```

---

### 2. Connect VS Code to Azure DevOps

* Open the **Azure Pipelines extension** in VS Code.
* Sign in with your Azure DevOps account.
* Select your **organization** and **project**.
* Link your local repo → remote Azure DevOps repo.

---

### 3. Run Pipeline from VS Code

* In VS Code Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P` on Mac):

  * Search: **Azure Pipelines: Run Pipeline**
  * Choose your pipeline (it will list all pipelines in the connected project).
  * Select the **branch** to run on.
* You’ll see logs directly inside VS Code or open in the Azure DevOps portal.

---

### 4. Monitor and Debug

* The **Azure Pipelines extension** shows:

  * Running pipelines
  * Build logs
  * Status (success/failure)
* You can click on a pipeline to view full logs in the Azure DevOps portal.

---

✅ **Tip:** If you prefer CLI, you can also trigger pipelines from VS Code terminal using:

```bash
az pipelines run --name <PipelineName> --branch main
```

---
Awesome — here’s a plug-and-play VS Code setup to create, trigger, and watch **Azure Pipelines** right from your editor.

# 1) Quick install (one-time)

```bash
# Azure CLI (skip if installed)
# macOS: brew update && brew install azure-cli
# Linux: https://aka.ms/azcli

# Login
az login

# Add Azure DevOps CLI extension
az extension add --name azure-devops

# Set your defaults (EDIT org & project)
az devops configure --defaults organization=https://dev.azure.com/<your-org> project="<your-project>"

# (Optional) store a PAT for non-interactive use
az devops login
# paste your Azure DevOps PAT and hit Enter
```

# 2) VS Code workspace files

Create a `.vscode` folder in your repo with these three files.

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "ms-azure-devops.azure-pipelines",
    "ms-vscode.azure-account",
    "ms-azuretools.azure-dev"
  ]
}
```

### `.vscode/settings.json`

```json
{
  "azure-pipelines.yamlFileGlob": "**/azure-pipelines*.yml",
  "azure-pipelines.defaultBranch": "main",
  "files.exclude": {
    "**/.azdevops": true
  }
}
```

### `.vscode/tasks.json`

> Adds Command Palette tasks to **Run pipeline**, **Create pipeline**, and **Open last run**.

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Pipelines: Run (current branch)",
      "type": "shell",
      "command": "BRANCH=$(git rev-parse --abbrev-ref HEAD); az pipelines run --name \"${input:pipelineName}\" --branch \"$BRANCH\"",
      "problemMatcher": []
    },
    {
      "label": "Pipelines: Create from azure-pipelines.yml",
      "type": "shell",
      "command": "az pipelines create --name \"${input:pipelineName}\" --repository $(basename -s .git $(git config --get remote.origin.url)) --repository-type tfsgit --branch $(git rev-parse --abbrev-ref HEAD) --yml-path azure-pipelines.yml",
      "problemMatcher": []
    },
    {
      "label": "Pipelines: Open last run in browser",
      "type": "shell",
      "command": "RUN_ID=$(az pipelines runs list --top 1 --query '[0].id' -o tsv); az pipelines runs show --id $RUN_ID --output table; az pipelines runs show --id $RUN_ID --query '_links.web.href' -o tsv | xargs open",
      "problemMatcher": []
    }
  ],
  "inputs": [
    {
      "id": "pipelineName",
      "type": "promptString",
      "description": "Azure Pipeline name",
      "default": "ci-build"
    }
  ]
}
```

> Use via **Terminal → Run Task…** (or `⌘⇧P` → “Run Task”).

# 3) Starter `azure-pipelines.yml` templates

Pick one and save as `azure-pipelines.yml` at repo root.

### Node.js (build & test)

```yaml
trigger:
  - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'
    displayName: Install Node.js
  - script: |
      npm ci
      npm run build
      npm test --if-present
    displayName: Build & Test
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: "$(System.DefaultWorkingDirectory)"
      ArtifactName: "drop"
      publishLocation: "Container"
```

### Python (pytest)

```yaml
trigger:
  - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.11'
  - script: |
      python -m pip install --upgrade pip
      pip install -r requirements.txt
      pytest -q
    displayName: Install & Test
```

### Docker build & push (to ACR)

```yaml
trigger:
  - main

variables:
  imageName: myapp

pool:
  vmImage: ubuntu-latest

steps:
  - task: AzureCLI@2
    inputs:
      azureSubscription: "<service-connection-to-subscription>"
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        az acr login -n $ACR_NAME
        docker build -t $ACR_NAME.azurecr.io/${imageName}:$(Build.BuildId) .
        docker push $ACR_NAME.azurecr.io/${imageName}:$(Build.BuildId)
```

# 4) Typical flows in VS Code

### A) First-time pipeline creation

1. Commit `azure-pipelines.yml`.
2. **Run Task → Pipelines: Create from azure-pipelines.yml**
   (or Command Palette: `> Azure Pipelines: Create Pipeline`)
3. Choose your organization/project if prompted. Done.

### B) Trigger a run on your current branch

* **Run Task → Pipelines: Run (current branch)**
  or Command Palette: `> Azure Pipelines: Run Pipeline` and pick the pipeline.

### C) Watch logs quickly

* In the **Azure Pipelines** panel (Activity Bar) you’ll see runs and logs.
* Or **Run Task → Pipelines: Open last run in browser** to jump into DevOps UI.

# 5) Handy CLI one-liners (use VS Code Terminal)

```bash
# List pipelines
az pipelines list -o table

# Trigger a run on 'main'
az pipelines run --name "ci-build" --branch main

# Tail last run logs (open in browser via URL)
az pipelines runs show --id $(az pipelines runs list --top 1 -o tsv --query "[0].id") \
  --query "_links.web.href" -o tsv
```

# 6) Common gotchas (fast fixes)

* **Not authorized / 401**: ensure `az devops login` with a valid **PAT** that has Build & Read/Execute permissions.
* **Pipeline not found**: run the **Create** task once or verify the pipeline name.
* **Service connection errors** (for Azure resources, ACR, AKS): create a service connection in **Project Settings → Service connections**, then reference it in YAML (e.g., `azureSubscription`).
* **GitHub repo**: if your code is on GitHub, set repository type to `github` during `az pipelines create` or create the pipeline from the DevOps UI and authorize GitHub.

---

