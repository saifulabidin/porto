Application Flow Document: Cyberpunk Portfolio
Version: 1.0
Date: September 26, 2025
Related Document: Technical Design Document & Project Manual (Version: Final)

CHAPTER 1: Introduction
1.1. Purpose
This document illustrates the user journeys and data flows within the Cyberpunk Portfolio application. It serves as a visual guide to understand how different components of the system interact, from a user's click on the frontend to the data processing in the backend. This complements the Technical Design Document by focusing on the "how" of the application's processes.

CHAPTER 2: Public User & Visitor Flow
This section describes the typical journey of a public visitor (e.g., a recruiter, potential client).

2.1. Main Navigation Flow
This flow shows a user exploring the main content of the portfolio.

graph TD
    A[Start: User lands on Homepage] --> B{Sees Hero Section & Featured Projects};
    B --> C[Clicks on "Projects" in Navigation];
    C --> D[Views Project Gallery Page];
    D --> E{Filters projects by category, e.g., "backend"};
    E --> F[Clicks on a specific project];
    F --> G[Views Project Detail Page];
    G --> H[Clicks on Live Demo link];
    H --> I[Opens new tab with the project demo];

2.2. Multi-Language Selection Flow
This flow demonstrates how a user changes the website's language.

graph TD
    A[User is viewing a page in English] --> B[Clicks on Language Switcher];
    B --> C{Selects "Japanese"};
    C --> D[Next.js router navigates to the same page with '/ja' prefix];
    D --> E[Frontend re-fetches dynamic content for Japanese];
    E --> F[UI text (static) is replaced with Japanese translations];
    F --> G[Page is now displayed entirely in Japanese];

2.3. Chatbot Interaction Flow
This flow illustrates the process of a user asking the AI chatbot a question.

graph TD
    A[User clicks the floating chatbot icon] --> B[Chat window opens];
    B --> C[User types a question: "What tech stack was used in Project X?"];
    C --> D[Frontend sends the question to the Backend API];
    D --> E[Backend forwards the query to Google Gemini API];
    E --> F[Gemini API processes the question and returns an answer];
    F --> G[Backend receives the answer and sends it back to the Frontend];
    G --> H[The answer is displayed in the chat window];

CHAPTER 3: Admin User Flow
This section describes the journey of Saiful Abidin managing the portfolio content.

3.1. Admin Authentication Flow
This flow covers the login process for the admin panel.

graph TD
    A[Admin navigates to '/admin/login'] --> B[Enters email and password];
    B --> C[Clicks "Login" button];
    C --> D[Frontend sends credentials to POST /api/admin/login];
    D --> E{Backend verifies credentials against the database};
    subgraph "If Successful"
        E -- Success --> F[Backend generates a JWT];
        F --> G[Backend returns the JWT to the Frontend];
        G --> H[Frontend stores JWT securely (e.g., in an httpOnly cookie)];
        H --> I[Redirects to the Admin Dashboard '/admin'];
    end
    subgraph "If Failed"
        E -- Failure --> J[Backend returns an error (e.g., 401 Unauthorized)];
        J --> K[Frontend displays an "Invalid credentials" message];
    end

3.2. Content Management Flow (Create New Project)
This flow shows the process of adding a new project to the portfolio.

graph TD
    A[Admin is on the Dashboard] --> B[Navigates to "Projects" section];
    B --> C[Clicks "Add New Project" button];
    C --> D[Fills out the project form with multilingual content];
    D --> E[Uploads gallery images];
    E --> F[Clicks "Save"];
    F --> G[Frontend sends a POST request with project data to /api/admin/projects];
    G --> H[Backend validates the data];
    H --> I[Backend saves the new project and its i18n content to the PostgreSQL database];
    I --> J[Backend returns a success message];
    J --> K[Frontend shows a success notification and redirects to the project list];

CHAPTER 4: System Data Flow
This section provides a high-level overview of how data moves through the system for a typical public request.

4.1. API Data Request Flow (Viewing Projects)
sequenceDiagram
    participant User
    participant Frontend (Next.js @ Vercel)
    participant Backend (Golang @ VPS)
    participant Database (PostgreSQL @ VPS)

    User->>Frontend (Next.js @ Vercel): Requests '/projects' page
    Frontend (Next.js @ Vercel)->>Backend (Golang @ VPS): GET /api/public/projects?lang=en
    Backend (Golang @ VPS)->>Database (PostgreSQL @ VPS): SELECT * FROM projects JOIN i18n_content ...
    Database (PostgreSQL @ VPS)-->>Backend (Golang @ VPS): Returns project records
    Backend (Golang @ VPS)-->>Frontend (Next.js @ Vercel): Returns JSON data of projects
    Frontend (Next.js @ Vercel)-->>User: Renders the page with project data
