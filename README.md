graph TB
    User((User))

    subgraph Frontend["React Frontend (Vite)"]
        UI[Dashboard UI]
        State[State Management]
        APIClient[API Client]
    end

    subgraph BackendCore["Backend Core"]
        Gateway[FastAPI Gateway\nCORS + Pydantic Validators]
        Orchestrator[Orchestrator Engine\nasyncio.gather]

        subgraph CommitteeAgents["Committee of Specialized Agents (Parallel)"]
            QuantAgent[Financial\nQuant Agent]
            NLPAgent[NLP\nTranscript Agent]
            NewsAgent[News &\nMacro Agent]
            CompAgent[Competitor\nPeer Agent]
        end

        subgraph MLLayer["Machine Learning"]
            yFinance[yFinance\nFetcher]
            XGBoost[XGBoost\nQuantile Model\nP05 / P50 / P95]
            SHAP[SHAP\nExplainer]
        end

        subgraph IntelligenceLayer["LLM Intelligence"]
            Groq[Groq LLM\nLlama 3 70B]
        end

        Ensembler[CIO Ensembler\nWeighted Confidence Aggregator]
        AuditManager[Audit Manager]
        SQLite[(SQLite\nAudit Trail)]
    end

    subgraph DataSources["External Data Sources"]
        Finnhub[Finnhub API\nCompany Profile]
        AlphaV[Alpha Vantage\nHistorical Data]
        FRED[FRED API\nMacro Indicators]
        NewsAPI[NewsAPI\nArticles]
        FeatureStore[Feature Store\nEngineered Vectors]
    end

    %% User → Frontend
    User -->|Search Ticker + Date| UI
    UI --> State
    State --> APIClient

    %% Frontend → Backend
    APIClient -->|POST /predict| Gateway

    %% Gateway → Orchestrator
    Gateway --> Orchestrator

    %% Orchestrator → Data Sources
    Orchestrator --> Finnhub
    Orchestrator --> AlphaV
    Orchestrator --> FRED
    Orchestrator --> NewsAPI
    Finnhub -->|Historical Data| FeatureStore
    AlphaV -->|Historical Data| FeatureStore
    FRED -->|Macro Data| FeatureStore
    NewsAPI -->|News Articles| FeatureStore

    %% Orchestrator → Agents (Parallel)
    FeatureStore -->|Features + JSON| QuantAgent
    FeatureStore -->|Transcript Text| NLPAgent
    FeatureStore -->|Macro Metrics| NewsAgent
    FeatureStore -->|Benchmarking Data| CompAgent

    %% ML Pipeline
    QuantAgent --> yFinance
    yFinance -->|Historical Prices| XGBoost
    XGBoost -->|Forecasts| SHAP
    SHAP -->|SHAP Values + Forecast| Ensembler

    %% LLM Agents
    NLPAgent -->|Analyze Transcript| Groq
    NewsAgent -->|Analyze Macro News| Groq
    CompAgent -->|Benchmark Peers| Groq
    Groq -->|Sentiment + Insights| NLPAgent
    Groq -->|Macro Score| NewsAgent
    Groq -->|Competitive Position| CompAgent

    %% Agents → Ensembler
    QuantAgent -->|Results + Confidence 0.84| Ensembler
    NLPAgent -->|Results + Confidence 0.65| Ensembler
    NewsAgent -->|Results + Confidence 0.72| Ensembler
    CompAgent -->|Results + Confidence 0.58| Ensembler

    %% Ensembler → Audit → DB
    Ensembler -->|Final Signal + Recommendation| AuditManager
    AuditManager -->|Persist| SQLite

    %% Response back to Frontend
    AuditManager -->|Consolidated JSON Response| Gateway
    Gateway -->|Consolidated State| APIClient
    APIClient -->|Render Dashboard| UI
    UI -->|Display Analysis + PDF Report| User
