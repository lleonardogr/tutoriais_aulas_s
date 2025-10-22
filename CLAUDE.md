# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **JetBrains Writerside documentation project** for a computer science course covering two major topics: **Web Development** and **Database Management with SQL Server**. The repository contains structured course materials organized as two parallel curricula, each with 16 lessons progressing from fundamentals to advanced topics.

The documentation is built and deployed automatically to GitHub Pages via GitHub Actions whenever changes are pushed to the main branch.

## Technology Stack

- **Documentation Platform:** JetBrains Writerside
- **Content Format:** Markdown (.md) and XML (.topic) files
- **Build System:** Docker-based Writerside builder (version 241.18775)
- **Deployment:** GitHub Pages via GitHub Actions
- **Language:** Portuguese (Brazilian)

## Repository Structure

```
.
├── Writerside/
│   ├── topics/
│   │   ├── web/                    # Web development lessons (HTML, CSS, JS)
│   │   │   ├── Web-Starting-Page.topic
│   │   │   ├── Aula-0-História-da-Web.md
│   │   │   ├── Aula-1-Instalação-do-ambiente.md
│   │   │   └── ... (Aulas 2-16)
│   │   ├── database/               # Database lessons (SQL Server)
│   │   │   ├── Database-Starting-Page.topic
│   │   │   ├── Aula-DB-0-História-dos-Bancos-de-Dados.md
│   │   │   ├── Aula-DB-1-Instalação-SQL-Server.md
│   │   │   └── ... (Aulas DB-2 to DB-16)
│   │   ├── Main-Landing-Page.topic  # Main entry point
│   │   └── starter.md               # Hidden template
│   ├── images/                  # Images and multimedia assets
│   ├── cfg/                     # Build configuration files
│   ├── in.tree                  # Table of contents (hierarchical structure)
│   ├── writerside.cfg           # Main Writerside configuration
│   ├── redirection-rules.xml
│   ├── c.list                   # Categories
│   └── v.list                   # Variables
├── .github/workflows/
│   └── build-docs.yml           # CI/CD pipeline for building and deploying docs
├── CLAUDE.md                    # This file
└── LICENSE                      # MIT License
```

## Common Commands

### Building Documentation Locally

To build the documentation locally using Docker:

```bash
docker run --rm -v $(pwd):/opt/sources \
  jetbrains/writerside-builder:241.18775 \
  build --instance Writerside/in --artifact webHelpIN2-all.zip
```

### Testing Documentation

After building, test the documentation for broken links and validation:

```bash
# Download the Writerside checker and run validation
# (Typically done in CI, but can be run locally with the checker action)
```

### Viewing Documentation Locally

After building, unzip the artifact and serve it locally:

```bash
unzip -O UTF-8 webHelpIN2-all.zip -d docs-preview
# Serve the docs-preview directory with any HTTP server
python3 -m http.server 8000 -d docs-preview
```

## Content Structure

The course is organized into **two main topics**, each with 16 lessons:

### Topic 1: Web Development (`web/`)

Covers HTML, CSS, and JavaScript fundamentals (Aulas 0-16):

1. **HTML & CSS Fundamentals** (Lessons 0-8)
   - Web history and fundamentals
   - HTML structure and semantics
   - CSS basics, styling, layouts, Flexbox, and Grid
   - First static project

2. **JavaScript Fundamentals** (Lessons 9-12)
   - JavaScript introduction and syntax
   - DOM manipulation and events
   - Control flow and functions
   - Interactive components

3. **Advanced Topics** (Lessons 13-15)
   - Asynchronous programming and APIs
   - Debugging and developer tools
   - Best practices, optimization, responsive design

4. **Projects and Assessments** (Lesson 16)
   - Three progressive projects with specific requirements
   - Delivery dates and evaluation criteria

### Topic 2: Database Management (`database/`)

Covers SQL Server and relational databases (Aulas DB-0 to DB-16):

1. **Database Fundamentals** (Lessons DB-0 to DB-4)
   - History of databases and SQL
   - SQL Server installation and configuration
   - Introduction to SQL and basic syntax
   - Data modeling (conceptual, logical, physical)
   - SQL Server data types

2. **Data Manipulation** (Lessons DB-5 to DB-8)
   - SELECT queries and basic operations
   - INSERT, UPDATE, DELETE
   - WHERE clauses and operators
   - First practical database project

3. **Advanced Queries** (Lessons DB-9 to DB-12)
   - JOINs and relationships
   - Aggregate functions and GROUP BY
   - Subqueries
   - Views and indexes

4. **Advanced Topics** (Lessons DB-13 to DB-16)
   - Stored procedures
   - Triggers and transactions
   - Optimization and best practices
   - Database projects and assessments

## Editing Content

### Adding New Lessons

#### For Web Development Lessons:

1. Create a new markdown file in `Writerside/topics/web/` following the naming pattern: `Aula-XX-Topic-Name.md`
2. Add the topic to the table of contents in `Writerside/in.tree` under the web section:
   ```xml
   <toc-element topic="web/Aula-XX-Topic-Name.md"/>
   ```
3. Update the web starting page (`web/Web-Starting-Page.topic`) if the lesson should be featured

#### For Database Lessons:

1. Create a new markdown file in `Writerside/topics/database/` following the naming pattern: `Aula-DB-XX-Topic-Name.md`
2. Add the topic to the table of contents in `Writerside/in.tree` under the database section:
   ```xml
   <toc-element topic="database/Aula-DB-XX-Topic-Name.md"/>
   ```
3. Update the database starting page (`database/Database-Starting-Page.topic`) if the lesson should be featured

### Content Guidelines

- **Language:** All content is in Portuguese (Brazil)
- **Naming Convention:**
  - Web lessons: `Aula-X-Title.md`
  - Database lessons: `Aula-DB-X-Title.md`
- **Folder Organization:** Web content in `web/`, database content in `database/`
- **Semantic Structure:** Use proper Markdown headers and Writerside-specific XML elements when needed
- **Multimedia:** Place all images in `Writerside/images/` and reference them with relative paths
- **Glossary:** Important terms should include definitions in a Glossary section at the end of lessons
- **Code Examples:** Use proper syntax highlighting for SQL, HTML, CSS, JavaScript

### Writerside-Specific Features

- **Topic files (.topic):** Use XML format for complex layouts like the starting page
- **Markdown extensions:** Writerside supports semantic attributes and XML injection in Markdown
- **Cross-references:** Link between topics using relative file paths
- **Collapsible sections:** Use `{collapsible="true"}` attribute on headers
- **Code blocks:** Use standard markdown code fences with language identifiers

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/build-docs.yml`) performs:

1. **Build Job:** Builds documentation using JetBrains Writerside Docker action
2. **Test Job:** Validates documentation for errors and broken links
3. **Deploy Job:** Deploys to GitHub Pages (requires Pages to be enabled)

All jobs run automatically on push to `main` branch or can be triggered manually via `workflow_dispatch`.

### Build Configuration

- **Instance:** `Writerside/in`
- **Artifact:** `webHelpIN2-all.zip`
- **Docker Version:** `241.18775`
- **No-index:** Set to `true` in `cfg/buildprofiles.xml` (prevents search engine indexing)

## Project-Specific Context

### Student Projects

#### Web Development Projects (documented in `web/Aula-16-Projetos-e-provas.md`):

1. **Project 1 (Prova):** Static site with HTML/CSS, live coding video demonstration (Due: March 21)
2. **Project 2 (Trabalho Extensivo):** Community-focused static site deployed to GitHub Pages (Due: April 4)
3. **Project 3:** JavaScript enhancements including i18n, theme switcher, interactive components (Due: April 18)

All web projects require:
- Minimum 3 pages
- Semantic HTML
- Internal and external links
- Multimedia content
- Proper CSS styling

#### Database Projects (documented in `database/Aula-DB-16-Projetos-e-Avaliações-DB.md`):

Projects include database modeling, implementation, and optimization exercises using SQL Server. Each student will create a complete database system for a real-world scenario.

### Course Philosophy

The course emphasizes:
- Progressive learning from fundamentals to advanced topics in both Web and Database domains
- Practical, hands-on projects with real-world applications
- Community impact through socially positive projects (web track)
- Modern web development and database management best practices
- Integration between front-end and back-end concepts
