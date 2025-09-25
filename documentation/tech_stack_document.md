Technology Stack Summary Document
Version: 1.0
Date: September 26, 2025
Purpose: To provide a consolidated, at-a-glance reference for all technologies, frameworks, libraries, and platforms used in the Cyberpunk Portfolio project.

1. Overview
This project utilizes a modern, decoupled architecture with a high-performance backend and a dynamic, server-rendered frontend.

Frontend: A Next.js application hosted on Vercel for optimal performance and SEO.

Backend: A Golang API containerized with Docker and hosted on a dedicated VPS for full control and scalability.

Database: A PostgreSQL database co-located with the backend on the VPS to ensure low-latency data access.

2. Frontend Stack
Category

Technology / Library

Rationale

Framework

Next.js 14+ (App Router)

Server Components, SSR, file-based routing for performance and DX.

Language

TypeScript

Type safety, improved code quality, and maintainability.

Styling

Tailwind CSS

Utility-first CSS for rapid and consistent UI development.

UI Components

shadcn/ui

Accessible, unstyled components built on Radix UI and Tailwind.

Icons

Lucide React

Consistent, lightweight, and customizable SVG icons.

State Management

Zustand (Global), React Context (Shared)

Lightweight global state management and standard React state sharing.

Data Fetching (Client)

SWR or TanStack Query

For client-side data fetching, caching, and revalidation.

Form Validation

Zod

Schema-based validation for type-safe forms.

Internationalization (i18n)

next-intl or Custom Implementation

Dynamic routing (/[lang]) and JSON dictionaries for multi-language support.

3. Backend Stack
Category

Technology / Library

Rationale

Language

Golang (Go)

High performance, concurrency, and efficiency for a fast API.

Web Framework

Gin or Echo

Lightweight, high-performance frameworks with robust routing and middleware.

Database

PostgreSQL

Powerful, reliable, and feature-rich open-source relational database.

ORM

GORM

Mature and widely-used ORM for Go, simplifying database interactions.

Authentication

JWT (JSON Web Tokens)

Standard for creating stateless authentication sessions.

Password Hashing

Bcrypt

Industry-standard algorithm for securely hashing passwords.

Validation

go-validator

For validating incoming data structures against defined rules.

4. Infrastructure & DevOps Stack
Category

Technology / Platform

Rationale

Frontend Hosting

Vercel

Seamless Git integration, automatic deployments, and global CDN for Next.js.

Backend Hosting

VPS (e.g., DigitalOcean, Linode)

Full control over the server environment, cost-effective for stable workloads.

Database Hosting

VPS (Co-located with Backend)

Minimizes network latency between the API and the database.

Containerization

Docker & Docker Compose

Ensures consistent development and production environments, simplifies deployment.

Web Server / Reverse Proxy

Nginx

Manages incoming traffic, handles SSL termination, and serves as a reverse proxy.

5. Tooling, Standards, & Security
Category

Tool / Standard

Purpose

API Documentation

OpenAPI (Swagger)

Defines a clear contract for the API, enabling auto-generated documentation.

Code Formatting

Prettier (Frontend), gofmt (Backend)

Enforces a consistent code style across the project.

Vulnerability Scanning

npm audit (Frontend), govulncheck (Backend)

Proactively identifies and helps mitigate security vulnerabilities in dependencies.

Version Control

Git (with Git Flow or similar)

Manages source code history and facilitates collaborative development.

