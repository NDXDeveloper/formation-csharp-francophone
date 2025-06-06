# 21.2. DevOps pour projets C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Le DevOps est une approche qui combine le développement (Dev) et les opérations (Ops) pour améliorer la collaboration et automatiser le cycle de vie des applications. Dans cette section, nous allons explorer comment implémenter des pratiques DevOps pour vos projets C#.

## 21.2.1. CI/CD avec GitHub Actions

GitHub Actions est une plateforme d'automatisation intégrée à GitHub qui permet de créer des workflows de CI/CD directement dans votre repository.

### Qu'est-ce que la CI/CD ?

- **CI (Continuous Integration)** : Intégration automatique et tests des changements de code
- **CD (Continuous Deployment)** : Déploiement automatique des applications

### Création d'un workflow GitHub Actions de base

Créez un fichier `.github/workflows/dotnet.yml` dans votre repository :

```yaml
name: .NET

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore

    - name: Test
      run: dotnet test --no-build --verbosity normal
```

### Workflow avancé avec tests et analyse de code

```yaml
name: .NET Build and Test

on:
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main" ]

env:
  DOTNET_VERSION: '8.0.x'

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET ${{ env.DOTNET_VERSION }}
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.nuget/packages
        key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json') }}
        restore-keys: |
          ${{ runner.os }}-nuget-

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Run unit tests
      run: dotnet test --no-build --configuration Release --logger trx --results-directory "TestResults"

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results-${{ matrix.os }}
        path: TestResults/*.trx

  code-quality:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: SonarScanner for .NET
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: |
        dotnet tool install --global dotnet-sonarscanner
        dotnet sonarscanner begin /k:"my-project" /o:"my-org" /d:sonar.login="${SONAR_TOKEN}" /d:sonar.host.url="https://sonarcloud.io"
        dotnet build
        dotnet sonarscanner end /d:sonar.login="${SONAR_TOKEN}"
```

### Workflow de déploiement

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        default: 'staging'
        type: environment

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: ${{ github.event.inputs.environment || 'production' }}
      url: ${{ steps.deploy.outputs.webapp-url }}

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x

    - name: Build
      run: |
        dotnet restore
        dotnet build --configuration Release
        dotnet publish -c Release -o ./publish

    - name: Deploy to Azure WebApp
      id: deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'my-webapp'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

## 21.2.2. Azure DevOps

Azure DevOps est une suite complète d'outils DevOps proposée par Microsoft, particulièrement adaptée aux projets .NET.

### Composants d'Azure DevOps

1. **Azure Repos** : Gestion du code source (Git ou TFVC)
2. **Azure Pipelines** : CI/CD
3. **Azure Boards** : Gestion des projets et des tâches
4. **Azure Test Plans** : Tests manuels et automatisés
5. **Azure Artifacts** : Gestion des packages

### Pipeline Azure DevOps YAML

Créez un fichier `azure-pipelines.yml` dans votre repository :

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: Build
    displayName: 'Build job'
    steps:
    - task: UseDotNet@2
      displayName: 'Use .NET 8 SDK'
      inputs:
        packageType: 'sdk'
        version: '8.0.x'

    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'

    - task: DotNetCoreCLI@2
      displayName: 'Build solution'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration) --no-restore'

    - task: DotNetCoreCLI@2
      displayName: 'Run unit tests'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage" --results-directory $(Agent.TempDirectory)/TestResults'

    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/TestResults/**/coverage.cobertura.xml'

    - task: DotNetCoreCLI@2
      displayName: 'Publish application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish build artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: Deploy
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: Deploy
    displayName: 'Deploy to Azure App Service'
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure App Service'
            inputs:
              azureSubscription: 'Azure-Connection'
              appType: 'webApp'
              appName: 'my-webapp-staging'
              package: $(Pipeline.Workspace)/drop/**/*.zip
```

### Pipeline multi-stages avec validation manuelle

```yaml
stages:
- stage: Build
  displayName: 'Build and Test'
  # ... (comme précédemment)

- stage: DeployToStaging
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployStaging
    environment: 'staging'
    # ... (déploiement vers staging)

- stage: DeployToProduction
  displayName: 'Deploy to Production'
  dependsOn: DeployToStaging
  condition: succeeded()
  jobs:
  - deployment: DeployProduction
    environment: 'production'
    strategy:
      runOnce:
        preDeploy:
          steps:
          - task: ManualValidation@0
            displayName: 'Manual validation for production deployment'
            inputs:
              instructions: 'Please validate the staging environment before deploying to production'
              onTimeout: 'reject'
        deploy:
          steps:
          # ... (déploiement vers production)
```

## 21.2.3. Tests automatisés en pipeline

L'automatisation des tests est essentielle pour assurer la qualité du code et la stabilité des déploiements.

### Structure des tests

```csharp
// Tests unitaires
public class CustomerServiceTests
{
    private readonly Mock<ICustomerRepository> _repositoryMock;
    private readonly CustomerService _service;

    public CustomerServiceTests()
    {
        _repositoryMock = new Mock<ICustomerRepository>();
        _service = new CustomerService(_repositoryMock.Object);
    }

    [Fact]
    public async Task CreateCustomer_ShouldReturnNewCustomerId()
    {
        // Arrange
        var customer = new Customer { FirstName = "John", LastName = "Doe" };
        _repositoryMock.Setup(r => r.AddAsync(It.IsAny<Customer>()))
                      .ReturnsAsync(1);

        // Act
        var result = await _service.CreateCustomerAsync(customer);

        // Assert
        Assert.True(result.IsSuccess);
        Assert.Equal(1, result.Value);
    }
}

// Tests d'intégration
public class CustomerControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public CustomerControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Get_ReturnsSuccessStatusCode()
    {
        // Act
        var response = await _client.GetAsync("/api/customers");

        // Assert
        response.EnsureSuccessStatusCode();
    }
}
```

### Configuration des tests dans GitHub Actions

```yaml
name: Test Suite

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      sql-server:
        image: mcr.microsoft.com/mssql/server:2019-latest
        env:
          SA_PASSWORD: YourPassword123!
          ACCEPT_EULA: Y
        ports:
          - 1433:1433
        options: >-
          --health-cmd="/opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P YourPassword123! -Q 'SELECT 1'"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore

    - name: Run unit tests
      run: dotnet test --no-build --filter Category=Unit --logger trx --results-directory "TestResults"

    - name: Run integration tests
      env:
        ConnectionStrings__DefaultConnection: "Server=localhost,1433;Database=TestDb;User Id=sa;Password=YourPassword123!;TrustServerCertificate=true"
      run: dotnet test --no-build --filter Category=Integration --logger trx --results-directory "TestResults"

    - name: Run acceptance tests
      run: |
        # Démarrer l'application en arrière-plan
        dotnet run --project src/MyApp.Api &
        sleep 10
        # Exécuter les tests d'acceptation
        dotnet test test/MyApp.AcceptanceTests --logger trx --results-directory "TestResults"

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: TestResults/*.trx

    - name: Generate coverage report
      run: |
        dotnet tool install -g dotnet-reportgenerator-globaltool
        reportgenerator -reports:TestResults/**/coverage.cobertura.xml -targetdir:CoverageReport -reporttypes:Html

    - name: Upload coverage report
      uses: actions/upload-artifact@v3
      with:
        name: coverage-report
        path: CoverageReport/
```

## 21.2.4. Déploiement continu

Le déploiement continu automatise la mise en production des applications après validation des tests.

### Stratégies de déploiement

#### 1. Rolling Deployment

```yaml
# azure-pipelines.yml
stages:
- stage: Deploy
  jobs:
  - deployment: DeployAzure
    environment: 'production'
    strategy:
      rolling:
        maxConcurrent: 2
        preDeploy:
          steps:
          - script: echo "Pre-deploy validation"
        deploy:
          steps:
          - script: echo "Deploy the application"
        postDeploy:
          steps:
          - script: echo "Post-deploy validation"
        on:
          failure:
            steps:
            - script: echo "Rollback deployment"
```

#### 2. Blue-Green Deployment

```yaml
# Exemple avec deux slots Azure App Service
- stage: BlueGreenDeploy
  jobs:
  - job: DeployToStaging
    steps:
    - task: AzureWebApp@1
      inputs:
        appName: 'myapp'
        resourceGroupName: 'myapp-rg'
        slotName: 'staging'

  - job: ValidationTests
    dependsOn: DeployToStaging
    steps:
    - script: |
        # Exécuter des tests de validation sur le slot staging
        curl https://myapp-staging.azurewebsites.net/health

  - job: SwapSlots
    dependsOn: ValidationTests
    steps:
    - task: AzureAppServiceManage@0
      inputs:
        action: 'Swap'
        appName: 'myapp'
        resourceGroupName: 'myapp-rg'
        sourceSlot: 'staging'
```

#### 3. Canary Deployment avec Kubernetes

```yaml
# k8s-canary-deployment.yml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 60s}
      - setWeight: 40
      - pause: {duration: 60s}
      - setWeight: 60
      - pause: {duration: 60s}
      - setWeight: 80
      - pause: {duration: 60s}
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
```

### Pipeline de déploiement avec validation

```yaml
# GitHub Actions avec validations
name: Deploy to Production

on:
  workflow_run:
    workflows: ["Build and Test"]
    branches: [main]
    types:
      - completed

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    environment:
      name: production
      url: https://myapp.com

    steps:
    - uses: actions/checkout@v3

    - name: Download artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-artifacts

    - name: Deploy to Production
      id: deploy
      run: |
        # Script de déploiement
        ./deploy.sh production

    - name: Smoke Tests
      run: |
        # Tests de fumée post-déploiement
        curl -f https://myapp.com/health || exit 1

    - name: Notify Slack
      if: success()
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        fields: repo,message,commit,author,action,eventName,ref,workflow
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## 21.2.5. Infrastructure as Code

L'Infrastructure as Code (IaC) permet de gérer et provisionner l'infrastructure via du code déclaratif.

### ARM Templates (Azure Resource Manager)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "webAppName": {
      "type": "string",
      "metadata": {
        "description": "Name of the web app"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "appServicePlanName": "[concat(parameters('webAppName'), '-asp')]",
    "sqlServerName": "[concat(parameters('webAppName'), '-sql')]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2021-02-01",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "S1",
        "tier": "Standard"
      },
      "properties": {
        "name": "[variables('appServicePlanName')]"
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2021-02-01",
      "name": "[parameters('webAppName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]",
        "siteConfig": {
          "netFrameworkVersion": "v8.0",
          "appSettings": [
            {
              "name": "WEBSITE_RUN_FROM_PACKAGE",
              "value": "1"
            }
          ]
        }
      }
    }
  ]
}
```

### Bicep (langage simplifié pour Azure)

```bicep
param webAppName string
param location string = resourceGroup().location
param environment string = 'dev'

var appServicePlanName = '${webAppName}-${environment}-asp'
var sqlServerName = '${webAppName}-${environment}-sql'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: environment == 'prod' ? 'P1v3' : 'B1'
    tier: environment == 'prod' ? 'Premium' : 'Basic'
  }
  properties: {
    reserved: true
  }
}

resource webApp 'Microsoft.Web/sites@2022-03-01' = {
  name: webAppName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: 'DOTNETCORE|8.0'
      appSettings: [
        {
          name: 'ASPNETCORE_ENVIRONMENT'
          value: environment
        }
        {
          name: 'ConnectionStrings__DefaultConnection'
          value: 'Server=${sqlServer.properties.fullyQualifiedDomainName};Database=myapp;User ID=${sqlServerAdminLogin};Password=${sqlServerAdminPassword};'
        }
      ]
    }
  }
}

resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
  name: sqlServerName
  location: location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: 'Password123!'
  }
}

resource sqlDatabase 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
  parent: sqlServer
  name: 'myapp'
  location: location
  sku: {
    name: environment == 'prod' ? 'S1' : 'Basic'
  }
}

output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
```

### Terraform pour multi-cloud

```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Variables
variable "environment" {
  description = "The deployment environment"
  type        = string
  default     = "dev"
}

variable "app_name" {
  description = "The name of the application"
  type        = string
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "${var.app_name}-${var.environment}-rg"
  location = "eastus"
}

# App Service Plan
resource "azurerm_service_plan" "main" {
  name                = "${var.app_name}-${var.environment}-plan"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = var.environment == "prod" ? "P1v3" : "B1"
}

# Web App
resource "azurerm_linux_web_app" "main" {
  name                = "${var.app_name}-${var.environment}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  service_plan_id     = azurerm_service_plan.main.id

  site_config {
    always_on                = var.environment == "prod" ? true : false
    application_stack {
      dotnet_version = "8.0"
    }
  }

  app_settings = {
    "ASPNETCORE_ENVIRONMENT"                = var.environment
    "WEBSITE_RUN_FROM_PACKAGE"              = "1"
    "ConnectionStrings__DefaultConnection"  = azurerm_mssql_database.main.connection_string
  }
}

# SQL Server
resource "azurerm_mssql_server" "main" {
  name                         = "${var.app_name}-${var.environment}-sql"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = random_password.sql_admin_password.result

  minimum_tls_version = "1.2"
}

# SQL Database
resource "azurerm_mssql_database" "main" {
  name           = var.app_name
  server_id      = azurerm_mssql_server.main.id
  sku_name       = var.environment == "prod" ? "S1" : "Basic"
  max_size_gb    = var.environment == "prod" ? 100 : 2
}

# Outputs
output "web_app_url" {
  value = "https://${azurerm_linux_web_app.main.default_hostname}"
}
```

### Pulumi (IaC avec C#)

```csharp
using Pulumi;
using Pulumi.AzureNative.Resources;
using Pulumi.AzureNative.Web;
using Pulumi.AzureNative.Web.Inputs;

class Program
{
    static Task<int> Main() => Deployment.RunAsync<MyStack>();
}

class MyStack : Stack
{
    public MyStack()
    {
        var resourceGroup = new ResourceGroup("myapp-rg");

        var appServicePlan = new AppServicePlan("myapp-plan", new AppServicePlanArgs
        {
            ResourceGroupName = resourceGroup.Name,
            Kind = "Linux",
            Reserved = true,
            Sku = new SkuDescriptionArgs
            {
                Name = "B1",
                Tier = "Basic"
            }
        });

        var app = new WebApp("myapp", new WebAppArgs
        {
            ResourceGroupName = resourceGroup.Name,
            ServerFarmId = appServicePlan.Id,
            SiteConfig = new SiteConfigArgs
            {
                LinuxFxVersion = "DOTNETCORE|8.0",
                AppSettings = new[]
                {
                    new NameValuePairArgs
                    {
                        Name = "ASPNETCORE_ENVIRONMENT",
                        Value = "Production"
                    }
                }
            }
        });

        this.Url = Output.Format($"https://{app.DefaultHostName}");
    }

    [Output] public Output<string> Url { get; set; }
}
```

### Intégration IaC dans les pipelines

```yaml
# GitHub Actions avec Terraform
name: Infrastructure Deployment

on:
  push:
    branches: [main]
    paths:
      - 'infrastructure/**'

jobs:
  terraform:
    runs-on: ubuntu-latest

    env:
      ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      ARM_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}

    steps:
    - uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0

    - name: Terraform Init
      run: terraform init
      working-directory: ./infrastructure

    - name: Terraform Plan
      run: terraform plan -out=tfplan
      working-directory: ./infrastructure

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply tfplan
      working-directory: ./infrastructure
```

## Bonnes pratiques DevOps pour C#

1. **Configuration par environnement**
   - Utilisez des transformations de configuration
   - Stockez les secrets de manière sécurisée (Key Vault, GitHub Secrets)
   - Variabilisez les paramètres par environnement

2. **Versioning et tagging**
   - Utilisez SemVer pour vos versions
   - Taggez vos déploiements
   - Maintenez un changelog

3. **Monitoring et observabilité**
   - Implémentez des health checks
   - Utilisez Application Insights ou des solutions équivalentes
   - Configurez des alertes proactives

4. **Sécurité**
   - Scannez les vulnérabilités
   - Validez les permissions minimales
   - Auditez les accès aux secrets

5. **Documentation**
   - Documentez vos pipelines
   - Maintenez des runbooks pour les opérations
   - Documentez les procédures de rollback

## Conclusion

Le DevOps pour projets C# nécessite une combinaison d'outils, de pratiques et de culture. Les plateformes comme GitHub Actions et Azure DevOps facilitent grandement l'automatisation, tandis que l'Infrastructure as Code permet de gérer l'infrastructure de manière fiable et reproductible. L'adoption de ces pratiques améliore significativement la vitesse de développement, la qualité des applications et la fiabilité des déploiements.

⏭️ 21.3. [Tendances et évolutions futures de C#](/21-extensions-et-ecosysteme-csharp/21-3-tendances-et-evolutions-futures-de-csharp.md)
