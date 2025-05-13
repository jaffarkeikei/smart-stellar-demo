# Documentation for Stellar Network Demo: Passkey-powered Onchain Chat

Welcome to the documentation for the Stellar Network Chat Demo. This documentation provides comprehensive information about the project's architecture, components, and workflows.

## Table of Contents

1. [Project Overview](./project-overview.md)
2. [Architecture](./architecture.md)
3. [Smart Contract](./smart-contract.md)
4. [Frontend](./frontend.md)
5. [Authentication](./authentication.md)
6. [Transaction Flow](./transaction-flow.md)
7. [Event Handling](./event-handling.md)
8. [Development Guide](./development-guide.md)

## Quick Start

To get started with the project:

1. Clone the repository
2. Install dependencies: `pnpm install`
3. Start the development server: `pnpm run dev`

## Key Features

- Passkey-powered authentication using WebAuthn
- Onchain chat messages stored as Soroban contract events
- Real-time message polling and display
- Transaction management via Launchtube
- Event indexing with Zettablock

## Technology Stack

- **Blockchain**: Stellar Network (Soroban smart contracts)
- **Frontend**: Astro + Svelte
- **Authentication**: Passkey Kit
- **Transaction Management**: Launchtube
- **Indexing**: Zettablock 