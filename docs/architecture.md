# Architecture Diagrams

## Overall System Flow

```mermaid
flowchart LR
    A[User Uploads Document] --> B[Document Processor]
    B --> C[Chunk & Embed]
    C --> D[Qdrant Vector Store]
    D --> E[Ready for Questions]

    F[User Asks Question] --> G[Query Mode Selection]
    G --> H{Which Mode?}

    H -->|Baseline| I[Retrieve All Context]
    H -->|Semantic| J[Retrieve + Highlight Relevant]
    H -->|Full| K[Retrieve + Highlight + HHEM Validate]

    I --> L[Generate Answer]
    J --> L
    K --> L

    L --> M[Return Answer + Metrics]
```

## Document Processing Pipeline

```mermaid
flowchart TD
    A[File Upload] --> B{File Type?}
    B -->|PDF| C[PyPDFLoader]
    B -->|TXT| D[TextLoader]
    B -->|MD| E[UnstructuredMarkdownLoader]
    B -->|JSON| F[Custom JSON Handler]

    C --> G[LangChain Document]
    D --> G
    E --> G
    F --> G

    G --> H[Text Splitter]
    H --> I[Chunks]

    I --> J[OpenAI Embeddings]
    J --> K[Vector Points]
    K --> L[Qdrant Storage]
```

## Query Mode Comparison

```mermaid
flowchart LR
    subgraph Baseline
        A1[Search] --> A2[All Context] --> A3[Generate Answer]
    end

    subgraph Semantic
        B1[Search] --> B2[Highlight Relevant] --> B3[Less Context] --> B4[Generate Answer]
    end

    subgraph Full
        C1[Search] --> C2[Highlight Relevant] --> C3[Less Context] --> C4[Generate Answer]
        C4 --> C5[HHEM Validate] --> C6[Answer + Score]
    end

    style A2 fill:#f9f,stroke:#333,stroke-width:2px
    style B3 fill:#9f9,stroke:#333,stroke-width:2px
    style C5 fill:#99f,stroke:#333,stroke-width:2px
```

## Service Layer Architecture

```mermaid
flowchart TB
    subgraph API Layer
        A[FastAPI Endpoints]
    end

    subgraph Services Layer
        B[RAGEngine]
        C[DocumentProcessor]
        D[VectorStore]
        E[SemanticHighlighter]
        F[HHEMValidator]
    end

    subgraph External Services
        G[Qdrant]
        H[OpenAI API]
        I[HuggingFace Models]
    end

    A --> B
    B --> C
    B --> D
    B --> E
    B --> F
    D --> G
    D --> H
    E --> I
    F --> I
```

## Semantic Highlighting Process

```mermaid
flowchart LR
    A[User Question] --> B[Retrieve Top K Chunks]
    B --> C[For Each Chunk]

    C --> D[Split into Sentences]
    D --> E[Score Relevance]
    E --> F{Score > Threshold?}

    F -->|Yes| G[Keep Sentence]
    F -->|No| H[Drop Sentence]

    G --> I[Join Kept Sentences]
    H --> I

    I --> J[Pruned Context]
```

## HHEM Validation Process

```mermaid
flowchart LR
    A[Context + Answer] --> B[HHEM Model]
    B --> C[Calculate Score 0-1]

    C --> D{Score >= Threshold?}

    D -->|Yes| E[Reliable Answer ✓]
    D -->|No| F[Possible Hallucination ⚠️]

    E --> G[Return Answer]
    F --> H[Return Answer + Warning]
```

## Cost Comparison

```mermaid
flowchart LR
    subgraph Baseline Cost
        A1[2500 Tokens] --> A2[$0.000375]
    end

    subgraph Semantic Cost
        B1[1200 Tokens] --> B2[$0.000180]
    end

    subgraph Full Cost
        C1[1200 Tokens] --> C2[$0.000180]
    end

    A2 -.->|52% Savings| B2
    A2 -.->|52% Savings| C2

    style A1 fill:#f99,stroke:#333
    style B1 fill:#9f9,stroke:#333
    style C1 fill:#9f9,stroke:#333
```
