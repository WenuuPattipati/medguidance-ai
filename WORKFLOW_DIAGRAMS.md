# System Architecture & Workflows

## 🏥 Doctor Mode Architecture

A high-precision clinical research workflow dealing with complex medical queries.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#E3F2FD', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#F3E5F5'}}}%%
graph LR
    subgraph Frontend [User Interface]
        User([👤 Clinician]) -->|Query| UI[Doctor Mode UI]
        UI -->|POST /api/chat| API[Next.js API Route]
    end

    subgraph Orchestrator [Phoenix Tracing & Orchestration]
        direction TB
        API -->|Start Span| ChatSpan(Chat Span)
        
        subgraph EvidenceEngine [Evidence Engine]
            ChatSpan -->|Call Tool| EvidenceSpan(Tool Span: Gather Evidence)
            EvidenceSpan -->|PICO Extraction| PICO[Query Analysis]
            PICO -->|Parallel| Sources{Source Selection}
            
            Sources -->|Search| PM[PubMed]
            Sources -->|Search| CL[Cochrane]
            Sources -->|Search| GL[Guidelines]
            Sources -->|Search| CT[ClinicalTrials]
            Sources -->|Search| DM[DailyMed]
            
            PM & CL & GL & CT & DM -->|Raw Results| Reranker[BGE Cross-Encoder]
            Reranker -->|Top 8| Context[Context Window]
        end
        
        subgraph ReasoningEngine [Reasoning & Synthesis]
            ChatSpan -->|Call LLM| LLMSpan(LLM Span: GPT-4o)
            Context -->|Injected| LLMSpan
            LLMSpan -->|Stream| Output[Structured Response]
        end
        
        Output -->|Data Stream| API
    end

    API -->|SSE Stream| UI
    UI -->|Render| Tabs[Clinical Tabs]

    classDef phoenix fill:#fff3e0,stroke:#f57c00,stroke-width:2px,stroke-dasharray: 5 5;
    classDef ai fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef tools fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    
    class ChatSpan,EvidenceSpan,LLMSpan phoenix;
    class Reranker,LLMSpan ai;
    class PM,CL,GL,CT,DM tools;
```

## 🧠 General Mode Architecture

A simplified, educational workflow optimized for speed and accessibility.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#F3E5F5', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#E3F2FD'}}}%%
graph LR
    subgraph Frontend [User Interface]
        User([👤 General User]) -->|Query| UI[General Mode UI]
        UI -->|POST /api/chat| API[Next.js API Route]
    end

    subgraph Orchestrator [Phoenix Tracing & Orchestration]
        direction TB
        API -->|Start Span| ChatSpan(Chat Span)
        
        subgraph DataGathering [Data Gathering]
            ChatSpan -->|Parallel Call| EvSpan(Tool: Evidence)
            ChatSpan -->|Parallel Call| ImgSpan(Tool: Images)
            
            EvSpan -->|Search| KB[MedlinePlus & CDC]
            EvSpan -->|Search and Ranking| BGE[BGE Reranker]
            
            ImgSpan -->|Fetch| OpenI[Open-i Images]
        end
        
        subgraph Synthesis [Response Generation]
            ChatSpan -->|Call LLM| LLMSpan(LLM Span: GPT-4o-mini)
            KB -->|Context| LLMSpan
            OpenI -->|Visuals| LLMSpan
            
            LLMSpan -->|Stream| Output[Simple Response]
        end
        
        Output -->|Data Stream| API
    end

    API -->|SSE Stream| UI
    UI -->|Render| View[Consumer View]

    classDef phoenix fill:#fff3e0,stroke:#f57c00,stroke-width:2px,stroke-dasharray: 5 5;
    classDef ai fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef tools fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    
    class ChatSpan,EvSpan,ImgSpan,LLMSpan phoenix;
    class BGE,LLMSpan ai;
    class KB,OpenI tools;
```
