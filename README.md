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

👉 Do you want me to also give you a **ready VS Code workspace setup** (with extensions, YAML template, and CLI commands) so you can instantly start running pipelines from your editor?
