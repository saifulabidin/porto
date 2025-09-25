Frontend Architecture & Guidelines Document: Next.js
Version: 1.0
Date: September 26, 2025
Purpose: To provide a comprehensive blueprint for the Next.js frontend application. This document details the project structure, component architecture, state management strategy, and development conventions to ensure a consistent, scalable, and maintainable codebase.

CHAPTER 1: Core Architecture & Philosophy
The frontend is built with Next.js (App Router), chosen for its powerful features like Server-Side Rendering (SSR), Server Components, and a file-based routing system that simplifies development. Our philosophy is centered around a modern, component-driven architecture.

Component-Based Architecture: The UI is composed of small, reusable, and independent components. We follow a practical approach inspired by Atomic Design to keep components organized and focused on a single responsibility.

Server-First Mentality: We leverage Next.js Server Components by default for optimal performance, fetching data on the server whenever possible. Client Components ("use client") are used only when necessary for interactivity and state management.

Styling: We exclusively use Tailwind CSS for styling. It is implemented through the shadcn/ui component library, which provides unstyled, accessible components that can be customized to fit our Cyberpunk theme. This ensures consistency and rapid UI development.

Internationalization (i18n): The application is fully internationalized from the ground up using a dynamic route structure (/[lang]).

CHAPTER 2: In-Depth Directory Structure
The project follows a structured and scalable folder system based on Next.js App Router conventions.

/cyberpunk-portfolio-web/
├── app/
│   ├── (admin)/              # Route group for protected admin pages
│   │   └── admin/
│   │       ├── layout.tsx
│   │       └── page.tsx      # Admin dashboard
│   ├── (public)/             # Route group for public-facing pages
│   │   ├── [lang]/           # Dynamic route for i18n
│   │   │   ├── about/
│   │   │   │   └── page.tsx
│   │   │   ├── projects/
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx    # Root layout for public pages
│   │   │   └── page.tsx      # Homepage
│   │   └── layout.tsx
│   ├── api/                  # Next.js API Routes (e.g., for chatbot proxy)
│   └── layout.tsx            # Root application layout
├── components/
│   ├── ui/                   # Unmodified components from shadcn/ui (e.g., Button, Card)
│   ├── shared/               # Custom, complex components shared across pages (e.g., Header, Footer, Chatbot)
│   └── features/             # Components specific to a feature or page (e.g., ProjectCard, ProjectGallery)
├── lib/
│   ├── api.ts                # Centralized functions for fetching data from the Golang backend.
│   ├── utils.ts              # Utility functions (e.g., cn for classnames).
│   └── validators.ts         # Zod schemas for form validation.
├── hooks/                    # Custom React hooks (e.g., useLocalStorage).
├── public/
│   └── fonts/                # Self-hosted font files.
├── styles/
│   └── globals.css           # Global styles and Tailwind base layers.
├── i18n/
│   ├── dictionaries/         # JSON files for translations (en.json, ja.json, etc.).
│   │   ├── en.json
│   │   └── id.json
│   └── i18n-config.ts        # Configuration for supported locales.
├── .env.local                # Local environment variables.
├── tailwind.config.ts        # Tailwind CSS configuration.
└── next.config.js            # Next.js configuration.

CHAPTER 3: Component Design & State Management
3.1. Component Strategy
Server Components First: Components are Server Components by default. They handle data fetching and are rendered on the server.

Client Components ("use client"): Use this directive only for components that require interactivity (e.g., onClick, onChange) or state/lifecycle hooks (e.g., useState, useEffect). Keep them as small as possible ("leaf" components).

Composition: Pass Server Components as children to Client Components to limit the amount of client-side JavaScript.

3.2. Data Fetching
Data is fetched from the Golang API using a centralized module.

Server-Side: Use fetch directly within Server Components. Next.js automatically handles caching and memoization.

Client-Side: For dynamic data that changes frequently without a page reload (e.g., live search), use a library like SWR or TanStack Query.

Example Data Fetching in a Server Component (/app/(public)/[lang]/projects/page.tsx):

import { getProjects } from '@/lib/api';
import ProjectGallery from '@/components/features/ProjectGallery';

export default async function ProjectsPage({ params: { lang } }) {
    // Fetches data on the server at build time or request time
    const projects = await getProjects(lang);

    return (
        <main>
            <h1>My Projects</h1>
            <ProjectGallery projects={projects} />
        </main>
    );
}

3.3. State Management
The state management strategy is tiered to use the simplest effective solution.

Local State: Use useState and useReducer for state confined to a single component.

Shared State (Component Tree): Use React Context API for state that needs to be shared between a few nested components (e.g., theme, language).

Global State: For complex global state (e.g., admin authentication session), a lightweight library like Zustand is preferred over Context to avoid performance issues with re-renders.

CHAPTER 4: Frontend-Backend Communication Flow
All API interactions are managed through the /lib/api.ts module to ensure consistency and a single source of truth for API endpoints.

sequenceDiagram
    participant User
    participant PageComponent as Next.js Page (Server)
    participant ApiClient as /lib/api.ts
    participant GoBackend as Golang API @ VPS

    User->>PageComponent: Requests '/projects' page
    PageComponent->>ApiClient: Calls getProjects('en')
    ApiClient->>GoBackend: fetch('[https://api.yourdomain.com/api/public/projects?lang=en](https://api.yourdomain.com/api/public/projects?lang=en)')
    GoBackend-->>ApiClient: Returns JSON data of projects
    ApiClient-->>PageComponent: Returns parsed data
    PageComponent-->>User: Renders HTML with project data

CHAPTER 5: Internationalization (i18n) Flow
The i18n system is built around the dynamic [lang] segment in the URL.

Routing: A middleware (middleware.ts) will inspect the incoming request's path to determine the locale. If no locale is present, it redirects to the default locale (e.g., /en).

Dictionaries: Translation strings are stored in JSON files within /i18n/dictionaries/.

Loading Translations: A helper function will load the appropriate dictionary on the server based on the lang parameter and make it available to components, either through props or a Context.

Example Translation Usage:

// /lib/i18n.ts
import 'server-only';
const dictionaries = {
  en: () => import('@/i18n/dictionaries/en.json').then((module) => module.default),
  id: () => import('@/i18n/dictionaries/id.json').then((module) => module.default),
};

export const getDictionary = async (locale) => dictionaries[locale]();

// In a component
import { getDictionary } from '@/lib/i18n';

export default async function HomePage({ params: { lang } }) {
  const dict = await getDictionary(lang);
  return <h1>{dict.hero.title}</h1>; // Renders translated title
}
