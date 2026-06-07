# Research Workflow

## Phase 0: Technology Stack Identification & Search Strategy

### 0.1 Confirm Stack Identity

Execute the following searches to confirm accurate information about the technology stack:

```
{tech-stack} official website documentation
{tech-stack} GitHub repository
{tech-stack} latest version 2025 2026
```

Record:
- Official name and abbreviation
- Latest stable version number
- Primary maintainer/organization
- License type
- Activity level (recent commit time, issue response speed)

### 0.2 Develop Search Strategy

Choose search priorities based on technology stack type:

| Stack Type | Additional Search Focus |
|-----------|------------------------|
| Frontend framework | Build tools, state management libraries, routing, CSS solutions |
| Backend framework | ORM, API design, authentication/authorization, database |
| Cross-platform mobile | Native bridging, hot reload, packaging/signing |
| Cross-platform desktop | IPC communication, system permissions, auto-updater |
| Programming language | Package manager, standard library, concurrency model |
| DevOps/Tools | Configuration management, plugin ecosystem, integration |

## Phase 1: Architecture & Directory Structure Research

### 1.1 Official Recommended Structure

Search:
```
{tech-stack} project structure best practice
{tech-stack} official recommended folder structure
{tech-stack} scaffolding template
```

### 1.2 Community Practices

Search:
```
{tech-stack} production project architecture
{tech-stack} enterprise project structure
{tech-stack} monorepo vs single repo
```

### 1.3 Layered Patterns

Search:
```
{tech-stack} layered architecture pattern
{tech-stack} service repository pattern
{tech-stack} feature-based organization
```

Record:
- Officially recommended directory structure
- Community mainstream practices (high-star GitHub projects)
- Layered patterns (MVC/MVVM/Layered/Feature-based)
- Special file conventions (e.g. `_layout.tsx`, `_test.go`, etc.)

## Phase 2: Coding Style & Conventions Research

### 2.1 Official Style Guide

Search:
```
{tech-stack} official style guide
{tech-stack} naming conventions
{tech-stack} code formatting
```

### 2.2 Lint/Format Tools

Search:
```
{tech-stack} linter {popular-linter-name}
{tech-stack} formatter configuration
{tech-stack} code quality tools
```

### 2.3 Type Safety (if applicable)

Search:
```
{tech-stack} TypeScript best practice (or corresponding type system)
{tech-stack} strict mode configuration
{tech-stack} type safety patterns
```

Record:
- Official or community recommended style guides
- Mainstream lint/format tools and configurations
- Type system best practices
- Naming conventions (files, variables, components, classes, etc.)

## Phase 3: Core Mechanisms Research

Select core mechanisms based on technology stack type:

### Frontend Framework
- Component lifecycle/rendering mechanism
- State management solutions
- Routing solutions
- Styling solutions
- Server-side rendering (SSR/SSG)

### Backend Framework
- Request processing pipeline
- ORM/database access
- Middleware mechanism
- Authentication/authorization
- API design (REST/GraphQL/RPC)

### Cross-platform Framework
- Native communication mechanism
- Platform adaptation strategies
- Packaging and distribution
- Hot update solutions

### Programming Language
- Package manager
- Concurrency/parallelism model
- Memory management
- Standard library usage

Search template:
```
{tech-stack} {core-mechanism} best practice
{tech-stack} {core-mechanism} pattern 2025
{tech-stack} official {core-mechanism} guide
```

## Phase 4: Testing Strategy Research

Search:
```
{tech-stack} testing best practice
{tech-stack} unit test framework
{tech-stack} integration testing
{tech-stack} e2e testing {tool-name}
{tech-stack} mocking strategy
```

Record:
- Officially recommended testing frameworks
- Testing pyramid practices for each layer
- Mock/Stub strategies
- Test file organization and naming

## Phase 5: Performance Optimization Research

Search:
```
{tech-stack} performance optimization
{tech-stack} profiling tools
{tech-stack} memory management
{tech-stack} async performance pattern
```

Record:
- Common performance bottlenecks
- Optimization patterns (lazy loading, caching, virtualization, etc.)
- Performance profiling tools
- Async/concurrency best practices

## Phase 6: Security & Error Handling Research

Search:
```
{tech-stack} security best practice
{tech-stack} error handling pattern
{tech-stack} input validation
{tech-stack} authentication authorization
```

Record:
- Common security vulnerabilities and protections
- Error handling patterns
- Input validation strategies
- Sensitive data handling

## Phase 7: Toolchain & Ecosystem Research

Search:
```
{tech-stack} build tool configuration
{tech-stack} dev environment setup
{tech-stack} CI CD pipeline
{tech-stack} dependency management
```

Record:
- Build tools and configurations
- Development environment recommendations
- CI/CD practices
- Dependency management strategies (version locking, update workflow)

## Phase 8: Special Features Research

Search for stack-specific advanced features:
```
{tech-stack} advanced features
{tech-stack} plugin system
{tech-stack} ecosystem tools
```

Record:
- Plugin/extension system
- Debugging tools
- Internationalization support
- Accessibility support

## Search Rules

1. **Official first**: Prioritize `site:{official-domain}` scoped searches
2. **Version constraint**: Include `2025`, `2026`, or latest version numbers in search queries to exclude outdated content
3. **GitHub validation**: Search `repo:{org}/{repo}` for actual code practices
4. **Cross-verify**: Each principle must be confirmed by at least 2 sources
5. **Record sources**: Record source URL and date for every discovered principle
