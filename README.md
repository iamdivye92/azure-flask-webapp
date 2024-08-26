## Deploying a Flask API to Azure Web App Using GitHub Actions
This project deploys a simple Flask API to an Azure Web App using GitHub Actions. It automates the build and deployment process, ensuring that updates are automatically pushed to Azure whenever changes are made to the code. The workflow simplifies continuous integration and deployment for the Flask application.


# Prerequisites

- **Azure Subscription**: An active Azure subscription to create and manage resources.
- **Azure CLI**: Installed and configured on your local machine.


## Workflow Steps

### **Step 1: Create a Basic Flask API**

Start by setting up a simple Flask API. 

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Azure!'

if __name__ == '__main__':
    app.run(debug=True)
```
Additionally, create a `requirements.txt` file to list your project's dependencies. Add Flask to the `requirements.txt` file:
```
    flask
```


 

### **Step 2: Install Azure CLI**

Download and install the Azure CLI from the [official Microsoft website](https://learn.microsoft.com/en-us/cli/azure/). Follow the installation instructions for your operating system. After installation, open your command line or terminal and log in to your Azure account using:


### **Step 4: Create a Resource Group**

Use the Azure CLI to create a new resource group. This group will contain all the resources for your Azure Web App. Replace :

```bash
az group create --name <YourResourceGroupName> --location <YourRegion>
```    
 



### **Step 5: Create an App Service Plan**

Use the Azure CLI to create an App Service Plan, which defines the region and pricing tier for your Azure Web App. Replace `<YourAppServicePlanName>` with your desired plan name and `<YourResourceGroupName>` with your resource group name. 

```bash
az appservice plan create --name <YourAppServicePlanName> --resource-group <YourResourceGroupName> --sku F1 --is-linux --location <YourRegion>
```
For Example 

```bash
az appservice plan create --name MyAppServicePlan --resource-group demo --sku F1 --is-linux --location westeurope
```
**Explanation of the Command:**

-   `--name MyAppServicePlan`: Specifies the name of the App Service Plan.
-   `--resource-group demo`: Defines the resource group in which the App Service Plan will be created.
-   `--sku F1`: Sets the pricing tier to Free (F1). You can change this to other tiers based on your needs.
-   `--is-linux`: Indicates that the App Service Plan will use a Linux environment.
-   `--location westeurope`: Specifies the Azure region where the App Service Plan will be deployed.

## STEP 6 Create a Web App

Use the Azure CLI to create a Web App within the App Service Plan. Replace `<YourWebAppName>`, `<YourResourceGroupName>`, `<YourAppServicePlanName>`, and `<YourRegion>` with your chosen values:

```bash
az webapp create --name <YourWebAppName> --resource-group <YourResourceGroupName> --plan <YourAppServicePlanName> --runtime "PYTHON|3.9" --location <YourRegion>
```
For Example 
```az webapp create --name flask-testing-env --resource-group demo --plan MyAppServicePlan --runtime "PYTHON|3.9" --location westeurope ```


**Explanation of the Command:**

-   `--name flask-testing-env`: The name of your Web App.
-   `--resource-group demo`: The resource group in which the Web App will be created.
-   `--plan MyAppServicePlan`: The App Service Plan that the Web App will use.
-   `--runtime "PYTHON|3.9"`: Specifies that the Web App will use Python version 3.9.
-   `--location westeurope`: The Azure region where the Web App will be deployed.

### **Step 7: Retrieve Deployment Credentials**

Retrieve the deployment credentials for your Web App, which will be used to configure GitHub Actions for automatic deployment. Use the Azure CLI to list the publishing profiles of your Web App:
```bash
az webapp deployment list-publishing-profiles --name <YourWebAppName> --resource-group <YourResourceGroupName> --xml
```
For Example : 
```az webapp deployment list-publishing-profiles --name flask-testing-env --resource-group demo --xml ```

**Explanation of the Command:**

-   `--name flask-testing-env`: The name of your Web App.
-   `--resource-group demo`: The resource group where your Web App is located.
-   `--xml`: Outputs the deployment credentials in XML format.




### **Step 8: Configure GitHub Repository with Deployment Credentials**

To automate the deployment of your Flask application using GitHub Actions, you need to add the deployment credentials from Azure to your GitHub repository as secrets.

1.  **Navigate to Your GitHub Repository:** Go to your GitHub repository where your Flask application code is hosted.
    
2.  **Open Repository Settings:** Click on the "Settings" tab of your repository.
    
3.  **Add Secrets:**
    
    -   Under the "Security" section, click on "Secrets and variables" and then select "Actions".
    -   Click the "New repository secret" button to add a new secret.
4.  **Add the Following Secrets:**
    
    -   **AZURE_WEBAPP_PUBLISH_PROFILE**: Paste the entire XML content of your Azure Web App publishing profile obtained from the previous step.
    -   **AZURE_WEBAPP_NAME**: Add the name of your Azure Web App, for example, `flask-testing-env`.

Adding these secrets will allow GitHub Actions to access your Azure Web App credentials securely, enabling continuous deployment of your application.

### **Step 9: Create a GitHub Actions Workflow File**

Now, create a GitHub Actions workflow file to automate the deployment of your Flask application to Azure. Follow these steps to set up the workflow:

1.  **Create Workflow Directory:** Inside your repository, create a directory named `.github/workflows`.
    
2.  **Create Workflow File:** In the `.github/workflows` directory, create a new file named `deploy.yml`.
    
3.  **Define the Workflow:** Add the following content to the `deploy.yml` file:

```bash

name: Deploy Flask App to Azure

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```      


Here’s the explanation using Markdown syntax:

----------

### Explanation of GitHub Actions Workflow

1.  **Workflow Name**:  
    Specifies the workflow's name as "Deploy Flask App to Azure," indicating its purpose.
    
2.  **Trigger Event**:  
    The workflow is triggered by a `push` event to the `main` branch of the repository.
    
3.  **Job Definition (`build-and-deploy`)**:  
    Defines a job that runs on an `ubuntu-latest` runner.
    
4.  **Steps**:
    
    -   **Checkout Code**: Clones the repository to the runner using `actions/checkout@v2`.
    -   **Set Up Python**: Installs Python 3.9 on the runner with `actions/setup-python@v2`.
    -   **Install Dependencies**: Upgrades `pip` and installs dependencies from `requirements.txt`.
    -   **Deploy to Azure Web App**: Deploys the Flask app to Azure using `azure/webapps-deploy@v2` with Azure Web App name and publish profile from GitHub secrets.

This workflow automates the deployment of a Flask app to Azure upon each push to the main branch.

----------

### Step Nine
To access the URL of your deployed Azure Web App from the Azure CLI without specifying the app name and resource group, you can use the following command:
```bash
az webapp show --name <YourWebAppName> --resource-group <YourResourceGroup> --query defaultHostName --output tsv
```
For Example :
```
az webapp show --name flask-testing-env --resource-group demo --query defaultHostName --output tsv
```
Explanation of Command :
-   `az webapp show`: This command retrieves information about the specified Azure Web App.
-   `--name flask-testing-env`: Specifies the name of your web app (`flask-testing-env`).
-   `--resource-group demo`: Specifies the resource group (`demo`) that contains your web app.
-   `--query defaultHostName`: Extracts the default host name (URL) of the web app from the retrieved information.
-   `--output tsv`: Outputs the result in plain text format (tab-separated values), making it easier to copy or use.

After running this command, you will get the URL of your deployed web app, such as `flask-testing-env.azurewebsites.net`. You can open this URL in a web browser to access your Flask
