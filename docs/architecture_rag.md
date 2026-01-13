# RAG-enabled architecture overview

The diagram below shows the RAG-enabled deployment flow for this repository. It focuses on the deployed Azure resources and the runtime data flow when Azure AI Search is enabled.

```mermaid
flowchart LR
    user[User / Client] -->|HTTP| ca[Azure Container App\napi_and_frontend]

    subgraph Azure[Azure Resource Group]
        ca -->|Managed Identity\n(AZURE_CLIENT_ID)| ai_foundry[Azure AI Foundry\n(AI Services Account)]
        ca -->|Managed Identity\n(AZURE_CLIENT_ID)| ai_project[Azure AI Foundry Project]
        ca -->|Managed Identity\n(AZURE_CLIENT_ID)| search[Azure AI Search]

        ca -->|Logs/Traces (optional)| appinsights[Application Insights]
        appinsights --> log_analytics[Log Analytics Workspace]

        ca --> acr[Container Registry]
        ca --> env[Container Apps Environment]
        env --> log_analytics

        ai_project --> ai_foundry
    end

    subgraph RAG_Flow[Runtime RAG flow]
        ca -->|Embeddings + Index ops| search
        ca -->|Retrieve context| search
        ca -->|Prompt + context| ai_foundry
    end

    note["RAG is enabled when USE_AZURE_AI_SEARCH_SERVICE=true" ] --- search
```

## Notes
- Managed Identity is used by the Container App to access Azure AI Foundry (project endpoint and model inference) and Azure AI Search.
- Application Insights + Log Analytics are used when tracing is enabled.
- The Container Apps Environment provides the runtime boundary and log configuration for Container Apps.
