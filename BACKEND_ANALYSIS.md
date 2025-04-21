# Codebase Analysis: nestar (Backend)

This document provides a high-level overview of the `nestar` backend project based on its configuration files, dependencies, and directory structure.

## Project Theme & Purpose

The backend provides the API and supporting services for the **Real Estate** frontend (`nestar-next`). It handles data persistence, business logic, real-time communication, and background tasks related to properties, users (members/agents), community interactions, and authentication.

It follows a monorepo structure, containing:
*   `nestar-api`: The main application serving the GraphQL API and WebSocket connections to the frontend.
*   `nestar-batch`: Likely handles scheduled jobs and background processing (e.g., data aggregation, notifications).

## Technologies Used

*   **Core Framework:** NestJS (v10)
*   **Language:** TypeScript
*   **Architecture:** Monorepo
*   **API Protocols:**
    *   GraphQL (`@nestjs/graphql`, `@apollo/server`)
    *   WebSockets (`@nestjs/websockets`)
    *   REST (Likely, via `@nestjs/platform-express`)
*   **Database:** MongoDB
*   **ORM/ODM:** Mongoose (`@nestjs/mongoose`)
*   **Authentication:** JWT (`@nestjs/jwt`), Bcrypt (`bcryptjs`)
*   **Real-time Communication:** Socket.IO / WS (`@nestjs/platform-socket.io`, `@nestjs/platform-ws`)
*   **Scheduling:** `@nestjs/schedule`
*   **Validation:** `class-validator`, `class-transformer`
*   **HTTP Client:** `@nestjs/axios`
*   **File Handling:** `graphql-upload`
*   **Key Libraries:** RxJS, Reflect Metadata
*   **Development & Tooling:**
    *   Package Manager: NPM (inferred from `package-lock.json`)
    *   Linting/Formatting: ESLint, Prettier
    *   Build System: NestJS CLI (`@nestjs/cli`), TypeScript (`tsc`)
    *   Testing: Jest, Supertest

## Core Modules (`nestar-api/src/components`)

*   `property`: Manages property listings and data.
*   `member`: Handles user accounts (buyers, sellers, agents).
*   `board-article`: Manages community forum/blog posts.
*   `comment`: Handles comments on articles or properties.
*   `like`: Manages 'like' interactions.
*   `follow`: Manages user/agent following relationships.
*   `view`: Tracks views (e.g., property views).
*   `auth`: Handles user authentication and authorization.

This analysis is based on `package.json`, `nest-cli.json`, `tsconfig.json`, and directory structures within `apps/`. 