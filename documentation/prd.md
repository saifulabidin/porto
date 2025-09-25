Technical Design Document & Project Manual: Cyberpunk Portfolio
Author: Saiful Abidin (sabidzpro)
Version: Final
Date: September 26, 2025

CHAPTER 1: Project Charter
1.1. Background & Vision
This project aims to build a digital portfolio for Saiful Abidin, a Full Stack Developer & Cyber Security Specialist. The main vision is to create an immersive digital experience with a Cyberpunk theme that not only showcases his work but also serves as a demonstration of advanced technical skills. This portfolio will function as an interactive, modern, and memorable digital business card for all visitors.

1.2. Project Objectives
Business Objectives:

Enhance personal branding as a technology expert.

Attract better career opportunities, freelance clients, or project collaborations.

Provide a platform for knowledge sharing through a blog.

Technical Objectives:

Implement a modern technology stack (Golang, Next.js, Docker, PostgreSQL).

Build and deploy a decoupled architecture between the frontend and backend.

Integrate AI services (Google Gemini) for innovative functionality.

1.3. Scope
In-Scope: All features defined in this document, from the public frontend, admin panel, and chatbot, to the deployment infrastructure on Vercel and a VPS.

Out-of-Scope: Authentication systems for public visitors (only admins can log in), e-commerce features, integration with third-party platforms other than Google Gemini, initial content creation (only the system itself).

1.4. Stakeholders
Project Owner: Saiful Abidin

Target Audience:

Recruiters and Technical Managers from tech companies.

Potential clients looking for a developer for their projects.

The developer community and fellow tech enthusiasts.

CHAPTER 2: System Architecture & Technical Design
2.1. Conceptual Architecture Diagram
The system uses a decoupled architecture:

A User accesses the Frontend (Next.js) hosted on Vercel.

The frontend communicates via a Public API (REST) with the Backend (Golang).

The Backend, running inside a Docker container on a VPS, manages all business logic.

The backend connects to the Database (PostgreSQL), which also runs in a Docker container on the same VPS.

An Admin accesses the Admin Panel (part of the Next.js app), which communicates via a Admin API (REST) protected by JWT authentication.

The Chatbot on the frontend calls a dedicated endpoint on the backend, which then communicates with the Google Gemini API.

2.2. Backend Architecture (Golang)
Framework: Gin Gonic (for fast routing and middleware).

Directory Structure (Recommended):

/cmd/server/main.go      # Application entry point
/internal/               # Project's private code
    /config/             # Configuration management (Viper)
    /database/           # DB connection & initialization
    /handler/            # HTTP handlers (controllers)
    /middleware/         # Middleware (Auth, CORS, Logging)
    /model/              # Data structs/entities
    /repository/         # Database access logic (CRUD)
    /service/            # Business logic
/pkg/                    # Code safe to share
    /utils/              # Utility functions (hashing, jwt)
go.mod

Key Libraries:

ORM/Database: GORM or sqlx for interacting with PostgreSQL.

Configuration: Viper for managing environment variables.

Authentication: jwt-go for JSON Web Token implementation.

2.3. Database Design (PostgreSQL)
Table Schema (SQL DDL):

-- Table to store multilingual content in JSON format
CREATE TABLE i18n_content (
    id SERIAL PRIMARY KEY,
    en TEXT,
    id_lang TEXT, -- "id" is a keyword, so use another name
    zh TEXT,
    ja TEXT,
    es TEXT
);

CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    title_id INT REFERENCES i18n_content(id),
    description_id INT REFERENCES i18n_content(id),
    role_id INT REFERENCES i18n_content(id),
    tech_stack TEXT[],
    project_url VARCHAR(255),
    source_code_url VARCHAR(255),
    gallery TEXT[],
    is_featured BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE skills (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    icon_svg_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) CHECK (category IN ('frontend', 'backend', 'devops', 'security')),
    mastery_level INT CHECK (mastery_level BETWEEN 1 AND 100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Other tables (experiences, certificates, blog_posts, testimonials) follow a similar pattern.

2.4. Frontend Architecture (Next.js)
Directory Structure:

/app/[lang]/                 # App Router based routing
    /page.js
    /projects/[slug]/page.js
/components/
    /features/               # Large components (HeroSection, ProjectList)
    /layout/                 # Header, Footer, MainLayout
    /ui/                     # Components from shadcn/ui
/lib/                        # Utility functions, API client
/hooks/                      # Custom hooks (e.g., useMediaQuery)
/styles/                     # Global CSS
/public/                     # Static assets (fonts, images)

State Management:

Server State: React Query or SWR for fetching, caching, and mutating data from the API.

Client State: Zustand or React Context for simple global state (e.g., theme, chatbot status).

API Client: Create a single axios instance in /lib/api.js configured with the baseURL from an environment variable. This instance will automatically handle adding authentication headers.

CHAPTER 3: Frontend Design System (Cyberpunk Theme)
This chapter defines the visual identity and interaction rules for all frontend elements. The goal is to create a cohesive, immersive experience that aligns with the Cyberpunk aesthetic.

3.1. Design Philosophy
High-tech, Low-life: Combining advanced interface elements (glowing effects, data streams) with a dystopian feel (glitches, dark colors, grids).

Functional & Informative: The design must clarify information, not obscure it. Every visual effect should have a purpose.

Immersive & Interactive: Users should feel like they are interacting with a terminal or system from the future.

3.2. Color Palette
This palette is designed for high contrast and clear readability in dark environments.

Backgrounds:

bg-primary: #0A0A0A (Onyx) - Main page background.

bg-secondary: #141414 (Eerie Black) - For surfaces like Cards or Modals.

Accents (Neon):

accent-blue: #00FFFF (Electric Blue) - For primary interactive elements and links.

accent-pink: #FF00FF (Cyber Pink) - For highlights, notifications, or critical elements.

accent-green: #39FF14 (Lime Green) - For success indicators or positive data.

Text:

text-primary: #E0E0E0 (Platinum) - For main text and headings.

text-secondary: #888888 (Gray) - For subheadings, descriptions, and helper text.

text-disabled: #444444 - For inactive elements.

3.3. Typography
Main Font (Headings & UI): Share Tech Mono or Fira Code (Monospaced) - Provides a technical, terminal-like feel. Imported from Google Fonts.

Body Font: Inter - A highly readable sans-serif for long paragraphs.

Hierarchy & Scale (Tailwind Classes):

H1: text-4xl or text-5xl, font-bold, tracking-wider, color text-primary.

H2: text-3xl, font-semibold, color text-primary.

H3: text-xl, font-medium, color text-secondary.

Paragraph: text-base, font-normal, leading-relaxed, color text-primary.

Link: Color accent-blue, underline-offset-4, with a color transition on hover.

3.4. Component System Customization
Custom styles for shadcn/ui components.

Button:

Primary Variant: accent-blue background, bg-primary text, a faint box-shadow with accent-blue for a glow effect. Sharp corners (rounded-none). The glow intensifies on hover.

Secondary/Outline Variant: 1px solid border, accent-blue border color, transparent background. On hover, the background fills with a low-opacity (10%) accent-blue.

Card:

bg-secondary background.

1px solid #333333 border. On hover, the border color transitions to accent-blue.

No box-shadow to maintain a flat, digital look.

Optional: Can have a very faint grid or circuit pattern background.

Input & Form:

No background (bg-transparent).

A 1px bottom border (border-b) with text-secondary color.

On focus, the bottom border changes color to accent-pink and has a glow effect.

Placeholder text uses text-secondary color.

3.5. Animation & Visual Effects
Glitch Effect: Used subtly on page transitions or when loading important elements.

Scan Line Effect: A static or gently animated overlay across the entire page to give a CRT monitor feel.

Glow Effect: Applied to interactive elements (links, buttons, active inputs) using box-shadow or filter: drop-shadow.

Typing Effect: Used on the hero section text or the chatbot to simulate text being typed.

Loading States: Replace standard spinners with ASCII art animations, "data streams," or "decryption effects."

3.6. Iconography
Library: lucide-react.

Style: Icons should be displayed in an outline style.

Color: Use text-primary color by default, changing to an accent color when inside an active interactive element.

CHAPTER 4: Detailed Feature Specifications
This section details every functionality to be built, from both the user-facing (frontend) and administrative (backend) sides.

4.1. Public Frontend Features
Home (Landing Page):

Hero Section: Displays name, title (Full Stack Developer & Cyber Security Specialist), and a short summary with a Typing Effect.

Featured Projects: Showcases 3-4 projects marked as is_featured.

Skills Summary: Visualization of key skills with a SkillsMatrix.

Testimonials: A scrollable section of client quotes.

Call to Action (CTA): A button directing to the contact page or email.

Projects Page:

Project List: A gallery of all projects with filters by technology category (frontend, backend, etc.).

Project Detail Page: A dedicated page for each project containing a full description, image/video gallery, tech stack, role, and links to the live demo and source code.

About Me Page:

Personal and professional biography.

Experience History Timeline (Timeline component).

List of Certifications.

Work setup (hardware & software used).

Blog Page:

A list of all articles with search and filter-by-category/tag functionality.

A detail page for each article with well-rendered Markdown content.

Interactive Chatbot (Gemini API):

A floating chatbot icon in the bottom-right corner.

A chat interface trained to answer questions about Saiful Abidin, his projects, and skills based on the website's content.

Multi-language Support:

A dropdown or switcher to select one of the 5 languages (Indonesian, English, Chinese, Japanese, Spanish).

Dynamic content from the API and static UI text will change according to the selected language.

Terminal Mode (Easter Egg):

Can be activated with a specific key combination (e.g., Ctrl+T).

Presents the website interface as a terminal where users can type commands (ls projects, cat about.txt, help) to navigate.

4.2. Admin Panel (CMS) Features
A secure interface for managing all dynamic website content.

Authentication: Login page with email and password, using JWT authentication.

Dashboard: The main page after login, displaying brief stats (number of projects, articles, etc.).

Project Management (CRUD): A form to add, view, edit, and delete project data according to the data model in Chapter 2.

Blog Management (CRUD): A Markdown-based text editor (e.g., react-markdown) for writing and managing articles.

Skills, Certificates, Experience, & Testimonials Management (CRUD): Simple forms for managing their respective data.

CHAPTER 5: Deployment Strategy & DevOps
5.1. Frontend Deployment (Vercel)
Trigger: Automatically on git push to the main or develop branch.

Process: Vercel will run next build, perform optimizations, and deploy the result to its global CDN.

Environment Variables: Configured via Vercel Project Settings (e.g., NEXT_PUBLIC_API_URL).

5.2. Backend Deployment (VPS & Docker)
Dockerfile (Multi-stage build):

# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /server ./cmd/server

# Stage 2: Final Image
FROM alpine:latest
WORKDIR /app
COPY --from=builder /server .
COPY .env .
EXPOSE 8080
CMD ["./server"]

Docker Compose (docker-compose.yml):

version: '3.8'
services:
  backend:
    build: .
    container_name: cyberpunk_api
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - DATABASE_URL=${DATABASE_URL}
  db:
    image: postgres:15-alpine
    container_name: cyberpunk_db
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
volumes:
  postgres_data:

Deployment Workflow:

Push code to the GitHub repository.

(Manually/Automated via CI/CD) SSH into the VPS.

Run git pull origin main.

Run docker-compose build.

Run docker-compose up -d to restart containers with the new image.

Reverse Proxy: Use Nginx or Caddy on the VPS to manage SSL (HTTPS) and forward traffic from port 443 to the backend container's port 8080.

CHAPTER 6: Development & Quality Protocols
6.1. Version Control Strategy (Git)
Model: GitFlow-lite.

main: Stable production code.

develop: Integration branch for upcoming release features.

feature/<feature-name>: Branches for new feature development.

Commit Rules: Use Conventional Commits (e.g., feat: add project CRUD endpoints, fix: correct validation logic).

6.2. Code Quality & Linting
Backend (Go): Use golangci-lint with a standard configuration.

Frontend (JS/TS): Use ESLint and Prettier, enforced with Git hooks (Husky).

6.3. Testing Strategy
Unit Tests:

Go: Use the built-in testing package to test services and repositories.

Next.js: Use Jest and React Testing Library to test components and hooks.

Integration Tests: Test the interaction between services and the database in the backend.

End-to-End (E2E): Manual testing according to the "Definition of Done."

APPENDIX A: Environment Variables (.env)
Backend (.env):

PORT=8080
GIN_MODE=release
DATABASE_URL="postgres://user:password@db:5432/dbname?sslmode=disable"
JWT_SECRET_KEY="a-very-strong-secret"
GEMINI_API_KEY="your-gemini-api-key"

Frontend (.env.local):

NEXT_PUBLIC_API_URL="[https://api.sabidzpro.is-a.dev](https://api.sabidzpro.is-a.dev)"

APPENDIX B: API Contract (OpenAPI/Swagger)
The API documentation will be created using the OpenAPI 3.0 specification. The backend will automatically serve this documentation using swaggo (a Go library) at the GET /swagger/index.html endpoint to facilitate frontend integration and testing.
