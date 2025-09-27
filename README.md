# BuildFlow Construction management

## Tender Management
```mermaid
flowchart LR
    A(Receive RFT)
    B(Create project)
    C(Create sub packages)
    D(Process sub package quotes)
    E(Prepare RFT quote)
    F(Submit quote)
    G(Tender awarded)
    H(Value engineering)
    I(Finalise costing)
    J(Contract)

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
```

## Sub Contractor Packaging
### Sub contractor pre qualification
Assumption: Contractor prequalification (PQQ) - We will build this into contractor onboarding and will have a review step included in the flow to check the contractor pre-qualification is still good.

### Sub Package Creation
```mermaid
---
config:
  layout: elk
  look: handDrawn
---
flowchart LR
  A((    )) --> B[Create Package]
  B --> C[Select BoQ items]
  C --> D[Generate package BoQ]
  D --> E[Select documents]
  E --> F[Review / Approve Package]
  F --> G((O))
```
### Sub-contractor invitation to tender
```mermaid
---
config:
  layout: elk
  look: handDrawn
---
flowchart LR

  A((    )) --> B[Review Instructions]
  B --> C[Pick suitable subbies]
  C --> C1{PPQ}
  C1 -->|No| C2[Review sub contrator]
  C2 --> D[Attach package docs]
  C1 -->|Yes| D[Attach package docs]
  D --> E[Attach contracts]
  E --> F[Send email]
  F -->G((O))
```
