```mermaid
flowchart TD
    subgraph INPUT
        A[Fundus Image<br/>256 × 256 × 3]
    end

    subgraph PREPROCESSING
        B[Resize, Normalize]
    end

    subgraph ENCODER["ENCODER (Contracting Path)"]
        E1[DoubleConv<br/>3→64<br/>256×256]
        E2[DownBlock<br/>64→128<br/>128×128]
        E3[DownBlock<br/>128→256<br/>64×64]
        E4[DownBlock<br/>256→512<br/>32×32]
        E5[Bottleneck<br/>512→1024<br/>16×16]
    end

    subgraph CLASSIFIER["CLASSIFICATION BRANCH"]
        C1[AdaptiveAvgPool<br/>16×16→1×1]
        C2[Fully Connected Layers]
        C3[Sigmoid]
        C4[Classification<br/>Probability]
    end

    subgraph DECODER["DECODER (Expanding Path)"]
        D1[UpBlock<br/>1024→512<br/>32×32]
        D2[UpBlock<br/>512→256<br/>64×64]
        D3[UpBlock<br/>256→128<br/>128×128]
        D4[UpBlock<br/>128→64<br/>256×256]
    end

    subgraph HEADS["SEGMENTATION HEADS"]
        H1[Cup Head<br/>Conv 64→1<br/>+Sigmoid]
        H2[Disc Head<br/>Conv 64→1<br/>+Sigmoid]
    end

    subgraph OUTPUTS["OUTPUT MASKS"]
        O1[Cup Mask<br/>256×256×1]
        O2[Disc Mask<br/>256×256×1]
    end

    subgraph CDR["CDR CALCULATION"]
        CDR1[Cup Area = Σ cup_mask]
        CDR2[Disc Area = Σ disc_mask]
        CDR3["CDR = √Cup_Area / √Disc_Area"]
    end

    subgraph CDRCLASS["CDR-BASED CLASSIFICATION"]
        CDRC1["CDR_prob = σ(10 × (CDR - 0.5))"]
    end

    subgraph ENSEMBLE["ENSEMBLE PREDICTION"]
        ENS1["Final = 0.6×Classifier + 0.4×CDR_prob"]
    end

    subgraph DECISION["FINAL DECISION"]
        DEC1{Final > 0.5?}
        DEC2[GLAUCOMA]
        DEC3[NORMAL]
    end

    A --> B --> E1
    E1 --> E2 --> E3 --> E4 --> E5
    
    E5 --> C1 --> C2 --> C3 --> C4 
    
    E5 --> D1
    E4 -.->|Skip| D1
    D1 --> D2
    E3 -.->|Skip| D2
    D2 --> D3
    E2 -.->|Skip| D3
    D3 --> D4
    E1 -.->|Skip| D4
    
    D4 --> H1 --> O1
    D4 --> H2 --> O2
    
    O1 --> CDR1
    O2 --> CDR2
    CDR1 --> CDR3
    CDR2 --> CDR3
    
    CDR3 --> CDRC1
    
    C4 --> ENS1
    CDRC1 --> ENS1
    
    ENS1 --> DEC1
    DEC1 -->|Yes| DEC2
    DEC1 -->|No| DEC3