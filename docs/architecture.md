# Architecture

This document details the architecture of the Stellar Network Chat Demo, explaining how the different components interact to create a cohesive application.

## System Architecture

The application follows a modern blockchain architecture pattern with clear separation of concerns:

```mermaid
flowchart LR
    subgraph Frontend ["Frontend (Astro + Svelte)"]
        UI[User Interface]
        Store[State Management]
    end
    
    subgraph Backend ["Blockchain (Stellar)"]
        SC[Smart Contract]
        Events[Event Storage]
    end
    
    subgraph Services ["Services"]
        PK[Passkey Kit]
        LT[Launchtube]
        ZB[Zettablock]
    end
    
    UI -->|User Input| Store
    Store -->|Contract Call| SC
    SC -->|Emit Events| Events
    Events -->|Poll| Store
    Store -->|Update| UI
    
    PK -->|Authentication| Store
    LT -->|Transaction Processing| SC
    ZB -->|Indexing| Events
    
    classDef frontend fill:#f9f9ff,stroke:#333,stroke-width:1px
    classDef backend fill:#f0f0ff,stroke:#333,stroke-width:1px
    classDef services fill:#fffff0,stroke:#333,stroke-width:1px
    
    class Frontend frontend
    class Backend backend
    class Services services
```

## Technical Stack

| Layer | Technology | Description |
|-------|------------|-------------|
| **Blockchain** | Stellar (Soroban) | Smart contract platform |
| **Frontend Framework** | Astro | Web framework for content-focused websites |
| **UI Library** | Svelte | Reactive UI component framework |
| **Authentication** | PasskeyKit | WebAuthn-based authentication library |
| **Transaction Management** | Launchtube | Stellar transaction submission service |
| **Event Indexing** | Zettablock | Blockchain data indexing service |
| **Styling** | Tailwind CSS | Utility-first CSS framework |

## Directory Structure

```mermaid
graph TD
    A[Project Root] --> B[src/]
    A --> C[contracts/]
    A --> D[public/]
    A --> E[docs/]
    
    B --> F[components/]
    B --> G[utils/]
    B --> H[pages/]
    B --> I[layouts/]
    B --> J[store/]
    
    C --> K[chat-demo/]
    
    F --> L[Welcome.svelte]
    
    G --> M[rpc.ts]
    G --> N[chat.ts]
    G --> O[passkey-kit.ts]
    G --> P[zettablocks.ts]
    G --> Q[base.ts]
    
    J --> R[contractId.ts]
    J --> S[keyId.ts]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style K fill:#bbf,stroke:#333,stroke-width:2px
    style L fill:#dfd,stroke:#333,stroke-width:2px
    style M fill:#dfd,stroke:#333,stroke-width:2px
```

## Component Architecture

### Frontend Components

```mermaid
componentDiagram
    component Frontend {
        component Components {
            [Welcome.svelte]
        }
        component Utils {
            [chat.ts]
            [passkey-kit.ts]
            [rpc.ts]
            [zettablocks.ts]
        }
        component Store {
            [contractId.ts]
            [keyId.ts]
        }
    }
    
    [Welcome.svelte] --> [chat.ts]
    [Welcome.svelte] --> [passkey-kit.ts]
    [Welcome.svelte] --> [rpc.ts]
    [Welcome.svelte] --> [contractId.ts]
    [Welcome.svelte] --> [keyId.ts]
    [Welcome.svelte] --> [zettablocks.ts]
```

### Data Flow

The sequence of operations when sending and receiving messages:

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant PasskeyKit
    participant Launchtube
    participant Contract
    participant Blockchain
    
    User->>Frontend: Type message
    Frontend->>PasskeyKit: Request signature
    PasskeyKit->>User: Biometric prompt
    User->>PasskeyKit: Authenticate
    PasskeyKit->>Frontend: Return signed transaction
    Frontend->>Launchtube: Submit signed transaction
    Launchtube->>Contract: Execute transaction
    Contract->>Blockchain: Emit event with message
    
    loop Every 12 seconds
        Frontend->>Blockchain: Poll for new events
        Blockchain->>Frontend: Return events
        Frontend->>User: Display new messages
    end
```

## Key Design Decisions

### Event-Based Data Storage

Rather than storing message data directly in contract storage, we use events for data persistence. This approach:

- Reduces storage costs
- Simplifies contract logic
- Enables easy event indexing and querying
- Provides an immutable history of messages

### Authentication with Passkeys

We use PasskeyKit for authentication because:

- It's more secure than password-based authentication
- It provides a familiar user experience
- It can handle transaction signing
- It's compatible with modern browsers and devices

### Polling for Events

The application polls for new events every 12 seconds because:

- It provides near real-time updates
- It's more efficient than WebSocket connections for this use case
- It's simple to implement
- It's compatible with all browsers

### XDR Format for Event Data

Stellar uses XDR (External Data Representation) format for structured data:

- Binary format for efficient data transfer
- Platform-independent serialization
- Strongly typed data structures
- Consistent across different programming languages

## Integration Points

| Integration | Purpose | Integration Method |
|-------------|---------|-------------------|
| Stellar RPC | Contract interaction | HTTP/JSON-RPC |
| PasskeyKit | Authentication | JavaScript SDK |
| Launchtube | Transaction management | HTTP API |
| Zettablock | Event indexing | HTTP API |
``` 