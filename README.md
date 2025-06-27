# Clinical AI Chatbot - Architecture Diagram

## System Architecture Overview

```mermaid
graph TB
    %% User Interface Layer
    subgraph "User Interface Layer"
        UI[Web Interface<br/>Flask + HTML/CSS/JS]
        API[Flask API<br/>/chat endpoint]
    end

    %% Core Workflow Engine
    subgraph "LangGraph Workflow Engine"
        START([Start])
        GUARD[Input Guardrail<br/>Content Safety Check]
        ROUTER[Router Agent<br/>Query Classification]
        LIT_AGENT[Literature Agent<br/>Case Studies & Research]
        DIAG_AGENT[Diagnostics Agent<br/>WHO Guidelines]
        GREET_AGENT[Greetings Agent<br/>General Conversation]
        END([End])
    end

    %% Knowledge Base Layer
    subgraph "Knowledge Base Layer"
        LIT_VECTOR[Literature Vector Store<br/>FAISS Index<br/>Medical Cases]
        DIAG_VECTOR[Diagnostics Vector Store<br/>FAISS Index<br/>WHO Guidelines]
        LIT_RETRIEVER[Literature Retriever<br/>Contextual Compression]
        DIAG_RETRIEVER[Diagnostics Retriever<br/>Contextual Compression]
    end

    %% LLM Layer
    subgraph "LLM Layer"
        GPT[GPT-3.5-turbo<br/>Response Generation]
        EMBED[OpenAI Embeddings<br/>text-embedding-3-large]
    end

    %% Data Processing Layer
    subgraph "Data Processing Layer"
        PDF_PROC[PDF Processing<br/>PyMuPDF + pdfplumber]
        MARKDOWN[Markdown Conversion<br/>Text + Tables + Links]
        VECTOR_CREATE[Vector Store Creation<br/>FAISS Indexing]
    end

    %% Raw Data Sources
    subgraph "Data Sources"
        PDFS[Raw PDFs<br/>Medical Literature<br/>WHO Guidelines]
    end

    %% User Flow
    UI --> API
    API --> START
    
    %% Workflow Flow
    START --> GUARD
    GUARD --> ROUTER
    ROUTER --> LIT_AGENT
    ROUTER --> DIAG_AGENT
    ROUTER --> GREET_AGENT
    LIT_AGENT --> END
    DIAG_AGENT --> END
    GREET_AGENT --> END

    %% Agent to Knowledge Base Connections
    LIT_AGENT --> LIT_RETRIEVER
    DIAG_AGENT --> DIAG_RETRIEVER
    LIT_RETRIEVER --> LIT_VECTOR
    DIAG_RETRIEVER --> DIAG_VECTOR

    %% Knowledge Base to LLM
    LIT_VECTOR --> EMBED
    DIAG_VECTOR --> EMBED
    LIT_AGENT --> GPT
    DIAG_AGENT --> GPT
    GREET_AGENT --> GPT

    %% Data Processing Flow
    PDFS --> PDF_PROC
    PDF_PROC --> MARKDOWN
    MARKDOWN --> VECTOR_CREATE
    VECTOR_CREATE --> LIT_VECTOR
    VECTOR_CREATE --> DIAG_VECTOR

    %% Styling
    classDef userLayer fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef workflowLayer fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef knowledgeLayer fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef llmLayer fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef dataLayer fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef sourceLayer fill:#f1f8e9,stroke:#33691e,stroke-width:2px

    class UI,API userLayer
    class START,GUARD,ROUTER,LIT_AGENT,DIAG_AGENT,GREET_AGENT,END workflowLayer
    class LIT_VECTOR,DIAG_VECTOR,LIT_RETRIEVER,DIAG_RETRIEVER knowledgeLayer
    class GPT,EMBED llmLayer
    class PDF_PROC,MARKDOWN,VECTOR_CREATE dataLayer
    class PDFS sourceLayer
```

## Detailed Component Architecture

```mermaid
graph LR
    %% Router Agent Details
    subgraph "Router Agent"
        ROUTER_INPUT[User Query]
        ROUTER_LLM[GPT-3.5-turbo<br/>Structured Output]
        ROUTER_DECISION{Classification<br/>Decision}
        ROUTER_OUTPUT[Route to Agent]
    end

    %% Literature Agent Details
    subgraph "Literature Agent"
        LIT_INPUT[Literature Query]
        LIT_CHAIN[Literature Chain]
        LIT_PROMPT[Literature Template]
        LIT_OUTPUT[Literature Response]
    end

    %% Diagnostics Agent Details
    subgraph "Diagnostics Agent"
        DIAG_INPUT[Diagnostics Query]
        DIAG_CHAIN[Diagnostics Chain]
        DIAG_PROMPT[Diagnostics Template]
        DIAG_OUTPUT[Diagnostics Response]
    end

    %% Vector Store Details
    subgraph "Vector Store System"
        QUERY[Query Text]
        EMBEDDING[Text Embedding]
        SIMILARITY[Similarity Search]
        CONTEXT[Retrieved Context]
        COMPRESSION[Context Compression]
    end

    %% Chain Components
    subgraph "Chain Architecture"
        RETRIEVER[Retriever]
        PROMPT[Prompt Template]
        LLM[Language Model]
        PARSER[Output Parser]
    end

    %% Connections
    ROUTER_INPUT --> ROUTER_LLM
    ROUTER_LLM --> ROUTER_DECISION
    ROUTER_DECISION --> ROUTER_OUTPUT

    LIT_INPUT --> LIT_CHAIN
    LIT_CHAIN --> LIT_PROMPT
    LIT_PROMPT --> LIT_OUTPUT

    DIAG_INPUT --> DIAG_CHAIN
    DIAG_CHAIN --> DIAG_PROMPT
    DIAG_PROMPT --> DIAG_OUTPUT

    QUERY --> EMBEDDING
    EMBEDDING --> SIMILARITY
    SIMILARITY --> CONTEXT
    CONTEXT --> COMPRESSION

    RETRIEVER --> PROMPT
    PROMPT --> LLM
    LLM --> PARSER

    %% Styling
    classDef agentStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef vectorStyle fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef chainStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px

    class ROUTER_INPUT,ROUTER_LLM,ROUTER_DECISION,ROUTER_OUTPUT,LIT_INPUT,LIT_CHAIN,LIT_PROMPT,LIT_OUTPUT,DIAG_INPUT,DIAG_CHAIN,DIAG_PROMPT,DIAG_OUTPUT agentStyle
    class QUERY,EMBEDDING,SIMILARITY,CONTEXT,COMPRESSION vectorStyle
    class RETRIEVER,PROMPT,LLM,PARSER chainStyle
```

## Data Flow Architecture

```mermaid
sequenceDiagram
    participant User
    participant WebUI
    participant FlaskAPI
    participant LangGraph
    participant Router
    participant LiteratureAgent
    participant DiagnosticsAgent
    participant VectorStore
    participant LLM
    participant KnowledgeBase

    User->>WebUI: Type medical question
    WebUI->>FlaskAPI: POST /chat {message}
    FlaskAPI->>LangGraph: invoke(workflow)
    
    LangGraph->>LangGraph: Input Guardrail Check
    LangGraph->>Router: Route query
    
    alt Literature Query
        Router->>LiteratureAgent: Process literature query
        LiteratureAgent->>VectorStore: Search literature index
        VectorStore->>KnowledgeBase: Retrieve relevant documents
        KnowledgeBase-->>VectorStore: Return context
        VectorStore-->>LiteratureAgent: Return compressed context
        LiteratureAgent->>LLM: Generate response with context
        LLM-->>LiteratureAgent: Return literature response
        LiteratureAgent-->>LangGraph: Return response
    else Diagnostics Query
        Router->>DiagnosticsAgent: Process diagnostics query
        DiagnosticsAgent->>VectorStore: Search diagnostics index
        VectorStore->>KnowledgeBase: Retrieve relevant documents
        KnowledgeBase-->>VectorStore: Return context
        VectorStore-->>DiagnosticsAgent: Return compressed context
        DiagnosticsAgent->>LLM: Generate response with context
        LLM-->>DiagnosticsAgent: Return diagnostics response
        DiagnosticsAgent-->>LangGraph: Return response
    end
    
    LangGraph-->>FlaskAPI: Return final response
    FlaskAPI-->>WebUI: JSON response
    WebUI-->>User: Display response
```

## Knowledge Base Architecture

```mermaid
graph TB
    %% Raw Data Sources
    subgraph "Raw Data Sources"
        WHO_PDF[WHO Guidelines PDF<br/>guideline-170-en.pdf]
        CASE_PDFS[Medical Case PDFs<br/>Case_2.pdf, Case_4.pdf, etc.]
    end

    %% Processing Pipeline
    subgraph "Data Processing Pipeline"
        PDF_EXTRACT[PDF Text Extraction<br/>PyMuPDF]
        TABLE_EXTRACT[Table Extraction<br/>pdfplumber]
        LINK_EXTRACT[Link Extraction<br/>Fitz]
        MARKDOWN_CONV[Markdown Conversion<br/>Custom Utils]
        TEXT_CLEAN[Text Cleaning<br/>Formatting Utils]
    end

    %% Vector Store Creation
    subgraph "Vector Store Creation"
        CHUNK[Text Chunking]
        EMBED[OpenAI Embeddings<br/>text-embedding-3-large]
        FAISS_CREATE[FAISS Index Creation]
        INDEX_SAVE[Save to Local Storage]
    end

    %% Knowledge Bases
    subgraph "Knowledge Bases"
        DIAG_KB[Diagnostics Knowledge Base<br/>data/diagnos_db/]
        LIT_KB[Literature Knowledge Base<br/>data/faiss_index/]
    end

    %% Retrieval System
    subgraph "Retrieval System"
        QUERY_EMBED[Query Embedding]
        SIMILARITY_SEARCH[Similarity Search]
        CONTEXT_COMPRESS[Context Compression<br/>LLMChainFilter]
        RELEVANT_DOCS[Relevant Documents]
    end

    %% Data Flow
    WHO_PDF --> PDF_EXTRACT
    CASE_PDFS --> PDF_EXTRACT
    
    PDF_EXTRACT --> TABLE_EXTRACT
    PDF_EXTRACT --> LINK_EXTRACT
    PDF_EXTRACT --> MARKDOWN_CONV
    
    TABLE_EXTRACT --> MARKDOWN_CONV
    LINK_EXTRACT --> MARKDOWN_CONV
    MARKDOWN_CONV --> TEXT_CLEAN
    
    TEXT_CLEAN --> CHUNK
    CHUNK --> EMBED
    EMBED --> FAISS_CREATE
    FAISS_CREATE --> INDEX_SAVE
    
    INDEX_SAVE --> DIAG_KB
    INDEX_SAVE --> LIT_KB
    
    DIAG_KB --> QUERY_EMBED
    LIT_KB --> QUERY_EMBED
    
    QUERY_EMBED --> SIMILARITY_SEARCH
    SIMILARITY_SEARCH --> CONTEXT_COMPRESS
    CONTEXT_COMPRESS --> RELEVANT_DOCS

    %% Styling
    classDef sourceStyle fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef processStyle fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef vectorStyle fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef retrievalStyle fill:#e1f5fe,stroke:#01579b,stroke-width:2px

    class WHO_PDF,CASE_PDFS sourceStyle
    class PDF_EXTRACT,TABLE_EXTRACT,LINK_EXTRACT,MARKDOWN_CONV,TEXT_CLEAN,CHUNK,EMBED,FAISS_CREATE,INDEX_SAVE processStyle
    class DIAG_KB,LIT_KB vectorStyle
    class QUERY_EMBED,SIMILARITY_SEARCH,CONTEXT_COMPRESS,RELEVANT_DOCS retrievalStyle
```

## Security and Safety Architecture

```mermaid
graph TB
    %% Input Layer
    subgraph "Input Layer"
        USER_INPUT[User Input]
    end

    %% Guardrail System
    subgraph "Guardrail System"
        CONTENT_CHECK[Content Safety Check]
        PII_DETECT[PII Detection]
        HARM_CHECK[Harmful Content Check]
        MEDICAL_CHECK[Medical Relevance Check]
        INJECTION_CHECK[Prompt Injection Check]
    end

    %% Decision Points
    subgraph "Decision Points"
        SAFE{Is Safe?}
        MEDICAL{Is Medical?}
        APPROPRIATE{Is Appropriate?}
    end

    %% Actions
    subgraph "Actions"
        ALLOW[Allow Processing]
        BLOCK[Block with Warning]
        REDIRECT[Redirect to Human]
    end

    %% Safety Features
    subgraph "Safety Features"
        DISCLAIMER[Medical Disclaimer]
        PROFESSIONAL[Professional Consultation Advice]
        SOURCE_ATTRIBUTION[Source Attribution]
        LIMITATION[Limitation Disclosure]
    end

    %% Flow
    USER_INPUT --> CONTENT_CHECK
    CONTENT_CHECK --> PII_DETECT
    PII_DETECT --> HARM_CHECK
    HARM_CHECK --> MEDICAL_CHECK
    MEDICAL_CHECK --> INJECTION_CHECK
    
    INJECTION_CHECK --> SAFE
    SAFE -->|No| BLOCK
    SAFE -->|Yes| MEDICAL
    MEDICAL -->|No| REDIRECT
    MEDICAL -->|Yes| APPROPRIATE
    APPROPRIATE -->|Yes| ALLOW
    APPROPRIATE -->|No| REDIRECT
    
    ALLOW --> DISCLAIMER
    ALLOW --> PROFESSIONAL
    ALLOW --> SOURCE_ATTRIBUTION
    ALLOW --> LIMITATION

    %% Styling
    classDef inputStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef guardrailStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef decisionStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef actionStyle fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef safetyStyle fill:#fce4ec,stroke:#c2185b,stroke-width:2px

    class USER_INPUT inputStyle
    class CONTENT_CHECK,PII_DETECT,HARM_CHECK,MEDICAL_CHECK,INJECTION_CHECK guardrailStyle
    class SAFE,MEDICAL,APPROPRIATE decisionStyle
    class ALLOW,BLOCK,REDIRECT actionStyle
    class DISCLAIMER,PROFESSIONAL,SOURCE_ATTRIBUTION,LIMITATION safetyStyle
```

## Deployment Architecture

```mermaid
graph TB
    %% Client Layer
    subgraph "Client Layer"
        BROWSER[Web Browser<br/>Chrome, Firefox, Safari]
        MOBILE[Mobile Browser]
    end

    %% Load Balancer
    subgraph "Load Balancer"
        LB[Load Balancer<br/>Nginx/AWS ALB]
    end

    %% Application Layer
    subgraph "Application Layer"
        FLASK_APP[Flask Application<br/>app2.py]
        LANGGRAPH[LangGraph Workflow<br/>main.py]
    end

    %% Service Layer
    subgraph "Service Layer"
        OPENAI_API[OpenAI API<br/>GPT-3.5-turbo<br/>text-embedding-3-large]
    end

    %% Data Layer
    subgraph "Data Layer"
        FAISS_INDEX[FAISS Vector Store<br/>Local Storage]
        STATIC_FILES[Static Files<br/>Templates, CSS, JS]
    end

    %% Infrastructure
    subgraph "Infrastructure"
        SERVER[Server<br/>Linux/Windows]
        DOCKER[Docker Container]
        CLOUD[Cloud Platform<br/>AWS/GCP/Azure]
    end

    %% Connections
    BROWSER --> LB
    MOBILE --> LB
    LB --> FLASK_APP
    FLASK_APP --> LANGGRAPH
    LANGGRAPH --> OPENAI_API
    LANGGRAPH --> FAISS_INDEX
    FLASK_APP --> STATIC_FILES
    
    FLASK_APP --> SERVER
    SERVER --> DOCKER
    DOCKER --> CLOUD

    %% Styling
    classDef clientStyle fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef loadBalancerStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef appStyle fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef serviceStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef dataStyle fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef infraStyle fill:#e0f2f1,stroke:#004d40,stroke-width:2px

    class BROWSER,MOBILE clientStyle
    class LB loadBalancerStyle
    class FLASK_APP,LANGGRAPH appStyle
    class OPENAI_API serviceStyle
    class FAISS_INDEX,STATIC_FILES dataStyle
    class SERVER,DOCKER,CLOUD infraStyle
```

## Key Architecture Features

### 1. **Modular Design**
- Separated concerns with distinct agents for different medical domains
- Pluggable architecture allows easy addition of new knowledge domains
- Independent vector stores for different types of medical information

### 2. **Scalability**
- LangGraph's stateful workflow management
- FAISS vector search for efficient retrieval
- Flask-based API for horizontal scaling
- Docker containerization support

### 3. **Safety & Compliance**
- Comprehensive input validation and content filtering
- Medical disclaimers and professional consultation recommendations
- Source attribution and limitation disclosures
- PII detection and protection

### 4. **Performance**
- Contextual compression for relevant information retrieval
- Cached vector embeddings for fast similarity search
- Asynchronous processing capabilities
- Optimized prompt templates for efficient LLM usage

### 5. **User Experience**
- Real-time chat interface with audio feedback
- Text-to-speech for accessibility
- Responsive design for multiple devices
- Clear medical disclaimers and professional guidance 
