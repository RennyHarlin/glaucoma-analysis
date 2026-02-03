```mermaid
flowchart LR
    subgraph ENCODER["ENCODER"]
        direction TB
        I["Input<br/>256×256×3"]
        E1["DoubleConv<br/>256×256×64"]
        E2["DownBlock<br/>128×128×128"]
        E3["DownBlock<br/>64×64×256"]
        E4["DownBlock<br/>32×32×512"]
        E5["Bottleneck<br/>16×16×1024"]
    end

    subgraph BOTTLENECK["BOTTLENECK + CLASSIFIER"]
        direction TB
        BN["16×16×1024"]
        CLS["Classifier Branch<br/>↓<br/>AdaptiveAvgPool(1)<br/>↓<br/>Fully Connected Layers<br/>↓<br/>Sigmoid<br/>↓<br/> Classification Prob"]
    end

    subgraph DECODER["DECODER"]
        direction TB
        D1["Up<br/>32×32×256"]
        D2["Up<br/>64×64×128"]
        D3["Up<br/>128×128×64"]
        D4["Up<br/>256×256×64"]
    end

    subgraph OUTPUT["OUTPUTS"]
        direction TB
        CUP["Cup Mask<br/>256×256×1"]
        DISC["Disc Mask<br/>256×256×1"]
    end

    I --> E1 --> E2 --> E3 --> E4 --> E5 --> BN
    BN --> CLS
    BN --> D1 --> D2 --> D3 --> D4
    
    E4 -.->|"Skip Connection"| D1
    E3 -.->|"Skip Connection"| D2
    E2 -.->|"Skip Connection"| D3
    E1 -.->|"Skip Connection"| D4
    
    D4 --> CUP
    D4 --> DISC