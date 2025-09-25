Backend Architecture & Structure Document: Golang API
Version: 1.0
Date: September 26, 2025
Purpose: To provide a detailed blueprint for the Golang backend application's structure, coding patterns, and internal data flow. This document serves as a guide for developers to ensure code consistency and maintainability.

CHAPTER 1: Core Architecture Philosophy
The backend will be built following the principles of Layered Architecture, which is a pragmatic implementation of Clean Architecture. This approach separates concerns into distinct layers, ensuring that the application is modular, testable, and maintainable.

Dependency Rule: Dependencies flow inwards. The handler layer knows about the service layer, and the service layer knows about the repository layer. The repository layer is the only one that interacts directly with the database.

Decoupling: Interfaces are used extensively to decouple layers, allowing for easier testing (mocking) and swapping of implementations in the future.

Single Responsibility: Each component (handler, service, repository) has a single, well-defined responsibility.

CHAPTER 2: In-Depth Directory Structure
This section expands on the structure defined in the main Technical Design Document.

/cyberpunk-api/
├── cmd/
│   └── server/
│       └── main.go           # Entry point: server setup, dependency injection, routing.
├── internal/
│   ├── config/               # Handles loading config from .env files (Viper).
│   ├── database/             # Manages the PostgreSQL connection pool.
│   ├── handler/              # HTTP handlers (controllers). Translates HTTP requests to service calls.
│   │   ├── admin_handler.go
│   │   └── public_handler.go
│   ├── middleware/           # HTTP middleware (e.g., JWT auth, logging, CORS).
│   │   └── auth_middleware.go
│   ├── model/                # Data structures for the application (e.g., Project, Skill).
│   │   ├── project.go
│   │   └── i18n.go
│   ├── repository/           # Data access layer. Handles all database operations (CRUD).
│   │   └── project_repository.go
│   └── service/              # Business logic layer. Orchestrates data from repositories.
│       └── project_service.go
├── pkg/
│   ├── jwt/                  # Reusable package for generating and validating JWTs.
│   └── utils/                # General utility functions (e.g., password hashing, response formatting).
├── .env                      # Environment variables.
├── go.mod                    # Go module dependencies.
└── Dockerfile                # Docker build instructions.

CHAPTER 3: Layer Interaction & Code Examples
This chapter illustrates how the layers work together using the "Project" entity as an example.

3.1. Model (/internal/model/project.go)
Defines the data structure.

package model

import "time"

type Project struct {
    ID             int       `json:"id" gorm:"primaryKey"`
    TitleID        int       `json:"-"`
    Title          I18nContent `json:"title" gorm:"foreignKey:TitleID"`
    DescriptionID  int       `json:"-"`
    Description    I18nContent `json:"description" gorm:"foreignKey:DescriptionID"`
    TechStack      []string  `json:"tech_stack" gorm:"type:text[]"`
    ProjectURL     string    `json:"project_url"`
    SourceCodeURL  string    `json:"source_code_url"`
    IsFeatured     bool      `json:"is_featured"`
    CreatedAt      time.Time `json:"created_at"`
}

3.2. Repository (/internal/repository/project_repository.go)
Handles direct database interaction. It implements an interface for decoupling.

package repository

import (
    "gorm.io/gorm"
    "your_project/internal/model"
)

type ProjectRepository interface {
    FindAll() ([]model.Project, error)
    Create(project model.Project) (model.Project, error)
}

type projectRepository struct {
    db *gorm.DB
}

func NewProjectRepository(db *gorm.DB) ProjectRepository {
    return &projectRepository{db: db}
}

func (r *projectRepository) FindAll() ([]model.Project, error) {
    var projects []model.Project
    // Preload loads the related i18n content
    err := r.db.Preload("Title").Preload("Description").Find(&projects).Error
    return projects, err
}

func (r *projectRepository) Create(project model.Project) (model.Project, error) {
    err := r.db.Create(&project).Error
    return project, err
}

3.3. Service (/internal/service/project_service.go)
Contains the business logic. It depends on the repository interface.

package service

import (
    "your_project/internal/model"
    "your_project/internal/repository"
)

type ProjectService interface {
    GetAllProjects() ([]model.Project, error)
    CreateProject(project model.Project) (model.Project, error)
}

type projectService struct {
    repo repository.ProjectRepository
}

func NewProjectService(repo repository.ProjectRepository) ProjectService {
    return &projectService{repo: repo}
}

func (s *projectService) GetAllProjects() ([]model.Project, error) {
    // Business logic could be added here, e.g., filtering or sorting.
    return s.repo.FindAll()
}

func (s *projectService) CreateProject(project model.Project) (model.Project, error) {
    // Business logic before creating, e.g., validation.
    return s.repo.Create(project)
}

3.4. Handler (/internal/handler/public_handler.go)
Manages HTTP requests and responses. It depends on the service interface.

package handler

import (
    "net/http"
    "[github.com/gin-gonic/gin](https://github.com/gin-gonic/gin)"
    "your_project/internal/service"
)

type PublicHandler struct {
    projectService service.ProjectService
}

func NewPublicHandler(projectService service.ProjectService) *PublicHandler {
    return &PublicHandler{projectService: projectService}
}

func (h *PublicHandler) GetProjects(c *gin.Context) {
    projects, err := h.projectService.GetAllProjects()
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to fetch projects"})
        return
    }
    c.JSON(http.StatusOK, projects)
}

3.5. Main Application (/cmd/server/main.go)
This is where everything is wired together (Dependency Injection).

package main

import (
    "[github.com/gin-gonic/gin](https://github.com/gin-gonic/gin)"
    "your_project/internal/database"
    "your_project/internal/handler"
    "your_project/internal/repository"
    "your_project/internal/service"
)

func main() {
    // Initialize Database
    db := database.InitDB()

    // Initialize Layers (Dependency Injection)
    projectRepo := repository.NewProjectRepository(db)
    projectSvc := service.NewProjectService(projectRepo)
    publicHandler := handler.NewPublicHandler(projectSvc)

    // Setup Gin Router
    router := gin.Default()

    // Public Routes
    publicRoutes := router.Group("/api/public")
    {
        publicRoutes.GET("/projects", publicHandler.GetProjects)
    }

    // Start Server
    router.Run(":8080")
}

CHAPTER 4: Backend Internal Data Flow
This diagram illustrates how an incoming HTTP request is processed through the layers.

sequenceDiagram
    participant GinRouter as Gin Router
    participant ProjectHandler as Handler
    participant ProjectService as Service
    participant ProjectRepository as Repository
    participant GORM
    participant Database

    GinRouter->>ProjectHandler: Forwards GET /projects request
    ProjectHandler->>ProjectService: Calls GetAllProjects()
    ProjectService->>ProjectRepository: Calls FindAll()
    ProjectRepository->>GORM: Executes db.Preload(...).Find(...)
    GORM->>Database: Generates and executes SQL query
    Database-->>GORM: Returns database rows
    GORM-->>ProjectRepository: Maps rows to []model.Project structs
    ProjectRepository-->>ProjectService: Returns slice of projects
    ProjectService-->>ProjectHandler: Returns slice of projects
    ProjectHandler-->>GinRouter: Sends JSON response with status 200
