# Legacy Pipeline Reconnaissance

## Directory Structure

```
legacy/tools/pipelines/
├── run-extractor.yaml
├── run-publisher-with-env.yaml
└── run-publisher.yaml
```

## Variables and Tasks Analysis

| File | Variables | Tasks | Touches subs? |
|------|-----------|--------|---------------|
| `.pre-commit-config.yaml` | None | None | NO |
| `legacy/configuration.extractor.yaml` | None | None | NO |
| `legacy/configuration.prod.yaml` | `{#tokenName#}`, `{#test#}`, `{#aiToolkitAudience#}`, `{#cognitiveCacheAudience#}`, `{#contentServerApiUrl#}`, `{#devaAudience#}`, `{#apimIdentityId#}` | None | NO |
| `legacy/tools/pipelines/run-extractor.yaml` | `$(SERVICE_CONNECTION_NAME)`, `$(apiops_release_version)`, `$(Agent.TempDirectory)`, `$(Agent.OS)`, `$(Agent.OSArchitecture)`, `$(EXTRACTOR_FILE_PATH)`, `$(AZURE_BEARER_TOKEN)`, `$(AZURE_CLIENT_ID)`, `$(AZURE_CLIENT_SECRET)`, `$(AZURE_TENANT_ID)`, `$(AZURE_SUBSCRIPTION_ID)`, `${{ parameters.RESOURCE_GROUP_NAME }}`, `${{ parameters.APIM_INSTANCE_NAME }}`, `$(Build.ArtifactStagingDirectory)`, `${{ parameters.API_MANAGEMENT_SERVICE_OUTPUT_FOLDER_PATH }}`, `${{ parameters.API_SPECIFICATION_FORMAT }}`, `${{ parameters.CONFIGURATION_YAML_PATH }}`, `$(Build.SourceBranchName)`, `$(Pipeline.Workspace)`, `$(System.TeamFoundationCollectionUri)`, `$(System.TeamProject)`, `${{ parameters.TARGET_BRANCH_NAME }}`, `$(Build.BuildId)`, `${{ parameters.APIM_REPOSITORY_NAME }}`, `$(System.AccessToken)` | `AzureCLI@2`, `PowerShell@2`, `NodeTool@0`, `PublishTestResults@2`, `PublishPipelineArtifact@1`, `Bash@3`, `DownloadPipelineArtifact@2` | YES |
| `legacy/tools/pipelines/run-publisher.yaml` | `${{ parameters.API_MANAGEMENT_SERVICE_OUTPUT_FOLDER_PATH }}`, `$(RESOURCE_GROUP_NAME)`, `$(APIM_NAME)`, `${{ parameters.COMMIT_ID }}`, `$(RESOURCE_GROUP_NAME_Prod)`, `$(Build.SourcesDirectory)` | `template` | NO |
| `legacy/tools/pipelines/run-publisher-with-env.yaml` | `${{ parameters.CONFIGURATION_YAML_PATH }}`, `${{ parameters.API_MANAGEMENT_SERVICE_NAME }}`, `$(SERVICE_CONNECTION_NAME)`, `$(apiops_release_version)`, `$(Agent.TempDirectory)`, `$(Agent.OS)`, `$(Agent.OSArchitecture)`, `$(PUBLISHER_FILE_PATH)`, `$(AZURE_BEARER_TOKEN)`, `$(AZURE_CLIENT_ID)`, `$(AZURE_CLIENT_SECRET)`, `$(AZURE_TENANT_ID)`, `$(AZURE_SUBSCRIPTION_ID)`, `${{ parameters.RESOURCE_GROUP_NAME }}`, `$(Build.SourcesDirectory)`, `${{ parameters.API_MANAGEMENT_SERVICE_OUTPUT_FOLDER_PATH }}`, `${{ parameters.API_MANAGEMENT_SERVICE_NAME }}`, `$(Build.SourceVersion)`, `${{ parameters.CONFIGURATION_YAML_PATH }}` | `AzureCLI@2`, `qetza.replacetokens.replacetokens-task.replacetokens@3`, `PowerShell@2` | YES |

## APIOps Tool References

### APIOps Extractor

* **`legacy/tools/pipelines/run-extractor.yaml`**
```yaml
- task: PowerShell@2
  displayName: Fetch extractor
  inputs:
    targetType: "inline"
    script: |
      Write-Information "Downloading release..."
      $uri = "https://github.com/Azure/apiops/releases/download/$(apiops_release_version)/$releaseFileName"
      $downloadFilePath = Join-Path "$(Agent.TempDirectory)" $releaseFileName
      Invoke-WebRequest -Uri "$uri" -OutFile "$downloadFilePath"

      Write-Information "Extracting release..."
      $executableFolderPath = Join-Path "$(Agent.TempDirectory)" "extractor"
      Expand-Archive -Path "$downloadFilePath" -DestinationPath "$executableFolderPath"
      $executableFilePath = Join-Path "$executableFolderPath" $executableFileName
```

```yaml
- task: PowerShell@2
  displayName: Run extractor
  inputs:
    targetType: "inline"
    script: |
      & "$(EXTRACTOR_FILE_PATH)"                
      if ($LASTEXITCODE -ne 0) { throw "Running extractor failed."}
```

### APIOps Publisher

* **`legacy/tools/pipelines/run-publisher-with-env.yaml`**
```yaml
- task: PowerShell@2
  displayName: Fetch publisher
  inputs:
    targetType: "inline"
    script: |
      Write-Information "Downloading release..."
      $uri = "https://github.com/Azure/apiops/releases/download/$(apiops_release_version)/$releaseFileName"
      $downloadFilePath = Join-Path "$(Agent.TempDirectory)" $releaseFileName
      Invoke-WebRequest -Uri "$uri" -OutFile "$downloadFilePath"

      Write-Information "Extracting release..."
      $executableFolderPath = Join-Path "$(Agent.TempDirectory)" "publisher"
      Expand-Archive -Path "$downloadFilePath" -DestinationPath "$executableFolderPath"
      $executableFilePath = Join-Path "$executableFolderPath" $executableFileName
```

```yaml
- task: PowerShell@2
  displayName: Run publisher for ${{ parameters.ENVIRONMENT}} environment
  inputs:
    targetType: "inline"
    script: |
      & "$(PUBLISHER_FILE_PATH)"                
      if ($LASTEXITCODE -ne 0) { throw "Running publisher failed."}
```

## Risk Summary

Extractor & publisher exist; variables include $(AZURE_SUBSCRIPTION_ID); no deploySubscriptions flag -> must set to false in Actions.
