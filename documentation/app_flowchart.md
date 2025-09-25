Application Flowchart Document: Cyberpunk Portfolio
Version: 1.0
Date: September 26, 2025
Purpose: To provide detailed visual flowcharts of user journeys, admin processes, and system data flows for the Cyberpunk Portfolio project.

CHAPTER 1: Public User Journey Flowchart
This flowchart illustrates a comprehensive journey of a potential client or recruiter exploring the portfolio from start to finish.

graph TD
    subgraph "Exploration Phase"
        A[Start: Visit Homepage] --> B[Scrolls through Hero & Featured Projects];
        B --> C{Decides to see more projects};
        C --> D[Navigates to Projects Page];
        D --> E[Filters projects by "Frontend"];
        E --> F[Selects "Project Alpha"];
    end

    subgraph "Evaluation Phase"
        F --> G[Reads Project Alpha's details, tech stack, and role];
        G --> H{Has a specific question};
        H --> I[Opens AI Chatbot];
        I --> J[Asks: "What was the main challenge in Project Alpha?"];
        J --> K[Receives AI-generated answer];
    end

    subgraph "Contact Phase"
        K --> L[Closes Chatbot and navigates to "About Me" page];
        L --> M[Reads bio and experience timeline];
        M --> N[Clicks on Contact/Email link];
        N --> O[Opens email client to send a message];
        O --> P[End of Journey];
    end

CHAPTER 2: Admin Content Management Lifecycle
This flowchart details the complete lifecycle of creating and publishing a new blog post by the admin.

graph TD
    subgraph "Authentication"
        A[Start: Admin navigates to /admin/login] --> B[Submits Credentials];
        B --> C{Auth Check};
        C -- Invalid --> B;
        C -- Valid --> D[Redirects to Admin Dashboard];
    end

    subgraph "Content Creation"
        D --> E[Navigates to "Blog Management"];
        E --> F[Clicks "Create New Post"];
        F --> G[Writes content in Markdown Editor];
        G --> H[Fills out Title, Tags, and other metadata for all languages];
        H --> I[Clicks "Save Draft"];
    end
    
    subgraph "Database Interaction"
        I --> J[API Request: POST /api/admin/posts];
        J --> K[Backend validates data];
        K -- Invalid --> H;
        K -- Valid --> L[Backend saves post to DB with 'draft' status];
        L --> M[API Response: Success];
    end

    subgraph "Publishing & Verification"
        M --> N[Admin reviews the draft in the list];
        N --> O[Clicks "Publish"];
        O --> P[API Request: PUT /api/admin/posts/:id/publish];
        P --> Q[Backend updates post status to 'published'];
        Q --> R[Admin opens the public blog page in a new tab];
        R --> S{Verifies the new post is visible};
        S --> T[Logs out from admin panel];
        T --> U[End];
    end

CHAPTER 3: System & Data Flowcharts
3.1. Detailed API Request Lifecycle (Sequence Diagram)
This diagram shows the detailed, step-by-step process of an authenticated API request from the admin panel.

sequenceDiagram
    participant Frontend
    participant Backend (Gin)
    participant AuthMiddleware
    participant Handler
    participant Service
    participant Repository
    participant Database

    Frontend->>Backend (Gin): PUT /api/admin/projects/123 (with JWT)
    Backend (Gin)->>AuthMiddleware: Passes request
    AuthMiddleware->>AuthMiddleware: Verifies JWT signature and expiry
    AuthMiddleware-->>Backend (Gin): JWT is valid, forwards request
    Backend (Gin)->>Handler: Calls UpdateProject handler
    Handler->>Handler: Binds and validates JSON payload
    Handler->>Service: Calls UpdateProject service with data
    Service->>Repository: Calls UpdateProject repository function
    Repository->>Database: Executes SQL UPDATE query on 'projects' table
    Database-->>Repository: Returns success/failure
    Repository-->>Service: Returns updated project model
    Service-->>Handler: Returns result
    Handler-->>Backend (Gin): Prepares success JSON response
    Backend (Gin)-->>Frontend: Sends 200 OK with updated project data

3.2. Deployment CI/CD Flowchart
This flowchart visualizes the automated deployment process for both frontend and backend.

graph TD
    A[Developer pushes code to a feature branch on GitHub] --> B[Creates a Pull Request to 'develop' branch];
    
    subgraph "CI Pipeline (GitHub Actions)"
        B --> C[Action Triggered: Linting & Static Analysis];
        C --> D[Run Unit & Integration Tests];
    end

    D -- Tests Fail --> E[PR Blocked, notify developer];
    D -- Tests Pass --> F[Code is approved and merged into 'develop'];

    subgraph "Frontend Deployment (Vercel)"
        F --> G[Vercel detects push to 'develop'];
        G --> H[Starts a new preview deployment];
        H --> I[Builds Next.js app];
        I --> J[Deployment successful, preview URL is available];
    end

    subgraph "Backend Deployment (Manual/CI on Main)"
        K[Developer merges 'develop' into 'main'] --> L[CI/CD Action for backend is triggered];
        L --> M[Agent securely SSH into VPS];
        M --> N[Pulls latest 'main' branch];
        N --> O[Runs 'docker-compose build' to create new image];
        O --> P[Runs 'docker-compose up -d' to restart backend with new image];
        P --> Q[Deployment complete];
    end
