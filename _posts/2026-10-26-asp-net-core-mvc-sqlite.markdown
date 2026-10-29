---
published: true
title: "ASP.NET-Core-MVC-SQLite"
layout: post
date: 2026-10-26 01:20:06 +0000
categories: csharp
---

# Architecting Modern Web Applications with ASP.NET Core MVC, SQLite, and Docker CI/CD: A Design-Science Study on Lightweight Full-Stack Development

## Author

Kairi Juri

## Abstract

Modern web application development increasingly demands architectures that balance rapid deployment, maintainability, security, and operational simplicity. This study examines the design and implementation of a full-featured book management web application built on ASP.NET Core MVC, SQLite, Docker containerization, and GitHub Actions CI/CD pipeline. Using a design-science research methodology, the study investigates how the integration of a serverless embedded database (SQLite) with a mature MVC framework (ASP.NET Core) and modern DevOps practices can yield a production-ready, lightweight web application architecture. The research addresses the architectural decisions, security model, data access patterns, and deployment automation that emerged during development. Findings demonstrate that the combination of ASP.NET Core MVC with SQLite provides a viable alternative to traditional client-server database architectures for small-to-medium applications, reducing infrastructure complexity while maintaining enterprise-grade security and observability features. The study contributes a reference architecture and identifies key trade-offs in persistence, concurrency, containerization, and DevOps automation.

## Keywords

ASP.NET Core MVC, SQLite, Docker, CI/CD, Web Application Architecture, Embedded Database, Design Science, DevOps

## 1. Introduction

The landscape of web application development has undergone significant transformation, driven by the proliferation of cloud-native architectures, containerization technologies, and automated deployment pipelines. Modern web frameworks such as ASP.NET Core have matured into cross-platform, high-performance platforms that support modular architecture, dependency injection, and middleware-based request processing [1]. Concurrently, the DevOps movement has established containerization and continuous integration/continuous deployment (CI/CD) as standard practices for achieving reliable, reproducible software delivery [2].

Within this context, database technology selection remains a critical architectural decision. While traditional client-server databases such as Microsoft SQL Server, PostgreSQL, and MySQL dominate enterprise applications, embedded database engines such as SQLite offer compelling advantages for specific use cases. SQLite, described as the most widely deployed database engine in the world, operates as a serverless, zero-configuration, self-contained SQL database engine that stores data in a single file [3]. Despite its ubiquity, SQLite is often dismissed as a development-only database in academic literature [4], and the specific challenges of containerizing an application with an embedded database remain underexplored.

The research gap centers on three interconnected questions: (1) How does the integration of SQLite with ASP.NET Core MVC and EF Core perform in a production-grade application with security, audit, and analytics requirements? (2) What architectural patterns and DevOps practices are necessary to containerize and deploy such an application reliably? (3) What are the trade-offs of this lightweight architecture compared to traditional client-server database approaches?

This study employs a design-science research methodology [5] to investigate these questions through the construction and evaluation of a full-featured book catalog management application.

## 2. Literature Review

### 2.1 Web Application Architecture Patterns

The Model-View-Controller (MVC) pattern has been a foundational architectural pattern in web development since its adaptation by Krasner and Pope [6] from the original Smalltalk-80 design [7]. ASP.NET Core MVC represents Microsoft's modern implementation, incorporating dependency injection, middleware pipelines, and cross-platform runtime support [1]. The framework's separation of concerns between controllers, views, and models improves maintainability and testability [8]. Fowler [9] established patterns for enterprise application architecture including the repository pattern and unit of work that are directly relevant to the data access layer examined in this study.

### 2.2 Embedded Database Systems

SQLite, first released by Hipp [3], has been described as the most widely deployed database engine globally. Despite its ubiquity, it is often dismissed as a "toy" database with recommendations limited to development contexts [4]. However, SQLite's Write-Ahead Logging (WAL) mode, introduced in version 3.7.0, significantly improved concurrency by enabling concurrent readers alongside a single writer [10]. The debate around SQLite's production viability centers on its lack of client-server architecture, limited concurrency model, and absence of stored procedures.

Entity Framework Core provides a code-first migration approach that abstracts database-specific DDL generation, enabling database engine portability [1]. Prior research on ORM patterns [9] and the active record versus data mapper debate [11] informs the repository pattern implementation examined in this study.

### 2.3 DevOps and Containerization

Docker containerization [12] revolutionized software deployment by packaging applications with their dependencies into portable, reproducible units. Hummer et al. [2] examined CI/CD practices for containerized applications, identifying automated testing and pipeline automation as critical practices. The challenge of state persistence in stateless containers, addressed through Docker volumes, is well-documented [13]. However, specific patterns for containerizing applications with embedded databases remain underexplored.

### 2.4 Research Gap

The intersection of ASP.NET Core MVC, SQLite, and Docker containerization in a production-grade application represents a gap in the literature. Existing documentation typically assumes SQL Server or PostgreSQL as the database backend [1], Docker documentation addresses stateless application containerization [13], and CI/CD literature focuses on microservices and client-server architectures [2]. This study addresses the gap by documenting the complete architecture from framework configuration through deployment automation.

## 3. Methodology

### 3.1 Research Design

This study employs a design-science research methodology [5], involving the construction and evaluation of a designed artifact to generate design knowledge. The artifact is a full-featured book management web application implemented using ASP.NET Core MVC, SQLite, Docker, and CI/CD automation. Design science is appropriate because the research questions concern how architectural components can be integrated into a viable production system, which is inherently a design problem.

### 3.2 System Design and Implementation

The application was implemented as a book catalog management system with the following functional requirements: book CRUD operations, user registration and authentication, role-based authorization (Admin and Member roles), admin user management, audit logging of write operations, page-view analytics with dashboard visualization, REST API for programmatic access, and automated deployment through Docker and CI/CD.

**Technology Stack.** The application was built on .NET 10 with ASP.NET Core MVC and Razor views. Data persistence used EF Core with the SQLite provider and SQLitePCLRaw.bundle_e_sqlite3 for the native SQLite engine. Authentication used ASP.NET Core Identity with IdentityUser and IdentityRole. The frontend used Bootstrap 5, jQuery validation, and Chart.js. Containerization used Docker with multi-stage builds (sdk:10.0 for build, aspnet:10.0 for runtime), and CI/CD was implemented through GitHub Actions publishing to Docker Hub.

**Solution Architecture.** The solution comprised a main web application containing Controllers (Account, Analytics, AuditLog, Book, Dashboard, Home, Rest, User), Data (mvcContext as EF Core DbContext, DatabaseInitializer, SeedData), Filters (AuditActionFilter, PageViewFilter), Models, and Services (IBookRepository/BookRepository, IAnalyticsService/AnalyticsService, EmailSender, AuditRetentionService).

**Database Design.** The SQLite database (booksDb.sqlite) contained four logical table groups: the books table (id, title, author, price), ASP.NET Core Identity tables, AuditLogs for audit trail, and PageViews for analytics. WAL mode (journal_mode=WAL) and synchronous=NORMAL were applied at startup to enable concurrent reads during writes. The connection string used a busy timeout of 30 seconds.

**Data Access Layer.** Since SQLite does not support stored procedures, the original T-SQL procedures were replaced by a BookRepository implementing the IBookRepository interface. The repository issued EF Core queries including GetAll (cached for 5 minutes via IMemoryCache), GetById, Add (INSERT with RETURNING), Update, Delete, and Search (using lower() and instr() for case-insensitive matching).

**Security Model.** Authentication used ASP.NET Core Identity with cookie-based authentication. Authorization used three policies: a fallback policy requiring authenticated users on all routes unless AllowAnonymous, a MemberOnly policy (Admin or Member role), and an AdminOnly policy (Admin role). User management included guard rails preventing self-deletion, self-demotion, self-locking, and removal of the last admin. Antiforgery tokens were applied to all state-changing actions. Audit logging was implemented through two global MVC filters: AuditActionFilter (logging POST/PUT/DELETE operations) and PageViewFilter (logging page views and maintaining a visitor count cookie).

**Containerization.** The Dockerfile used a two-stage build: the build stage restored NuGet packages on a separate layer, published with Release configuration; the runtime stage used the aspnet:10.0 Debian slim image, ran as the non-root app user (UID 1654), exposed port 8080, and included a healthcheck using a static busybox wget binary. The SQLite database file persisted on a named Docker volume (books-data) mounted at /app/data.

**CI/CD Pipeline.** The GitHub Actions workflow contained two jobs: a verify job that restored, built, and published the .NET project, and a docker job that built the image, ran it, polled the HTTP endpoint for up to 60 seconds, verified the SQLite database file was created on the volume, and then pushed the multi-tagged image to Docker Hub.

### 3.3 Evaluation Methods

Evaluation was conducted through: (1) functional testing via a Python smoke test exercising the complete user management flow with 47 pass/fail checks; (2) CI/CD pipeline smoke testing verifying the Docker container started, responded to HTTP requests, and created the database file; and (3) architectural analysis comparing the implementation against design patterns from the literature.

## 4. Results and Findings

### 4.1 Architectural Design Patterns

The implementation yielded several notable architectural patterns:

**Repository Pattern without Stored Procedures.** The migration from SQL Server stored procedures to EF Core queries demonstrated that the repository pattern could effectively replace stored procedure logic while maintaining the same interface contract. The GetAll method incorporated a 5-minute IMemoryCache cache. The Search method used lower() and instr() functions instead of LIKE for case-insensitive matching, treating % and _ as literal characters.

**Automatic Database Initialization.** The DatabaseInitializer executed at application startup, applying EF Core migrations (creating booksDb.sqlite on first run), enabling WAL mode, and seeding demo data only when the table was empty. Failure paths were logged and non-fatal, ensuring the application could start even if the database file was locked. This pattern eliminated the need for separate migration scripts in deployment.

**Global Filter-Based Observability.** The AuditActionFilter and PageViewFilter, registered as global MVC filters, provided transparent audit logging and analytics without requiring per-action instrumentation. An AuditRetentionService periodically purged audit logs older than a configurable retention period (default 180 days).

### 4.2 Security Model Implementation

**Role-Based Authorization.** The three-policy model provided comprehensive access control. The fallback policy ensured all routes required authentication unless explicitly marked AllowAnonymous. The /User management screen was gated at the controller level (AdminOnly policy), with navbar links merely hidden for non-admins, recognizing that UI hiding is not a security boundary.

**User Management Guard Rails.** Four critical guard rails were implemented: no self-deletion, no self-demotion, no self-locking, and no last-admin removal. These were enforced server-side with UI elements also disabled for defense in depth.

**Audit Trail Integrity.** Every state-changing operation wrote an AuditLogs row, capturing the action context, user identity, and outcome. The audit log was exposed through a paged admin interface with filtering and CSV export, with paging implemented via SQLite's LIMIT and OFFSET.

### 4.3 Database Design and Concurrency

**WAL Mode and Busy Timeout.** The combination of journal_mode=WAL and Default Timeout=30 enabled concurrent read access during write operations, critical for the audit filters writing rows while users queried the catalog. The synchronous=NORMAL pragma provided a balance between data integrity and write performance.

**Single-File Database Advantages.** The single-file nature of SQLite simplified backup operations (file copy or VACUUM INTO), eliminated database server provisioning, and enabled automatic database creation and migration on first run.

**Stored Procedure Migration.** The absence of stored procedures in SQLite required all data access logic to be moved into the BookRepository. This improved testability and maintainability, as the logic was now in C# code subject to unit testing and version control.

### 4.4 Containerization and Persistence

**Named Volume Persistence.** The SQLite database file persisted on a named Docker volume (books-data) at /app/data, making the container stateless. The docker-compose down command (without -v) preserved data. The volume was created and chowned to the non-root app user in the Dockerfile.

**Health Check Strategy.** The healthcheck probed the anonymous home page endpoint every 30 seconds after a 25-second start period, using a static busybox wget binary rather than curl installed via apt-get, keeping the build reproducible on networks with restricted package mirror access.

**Cookie Key Ring Caveat.** ASP.NET Core Data Protection keys were stored inside the container, meaning container recreation logged out all users. Persisting the key ring required mounting an additional volume at the key ring path.

### 4.5 CI/CD Pipeline Results

**Pre-Deployment Smoke Testing.** The Docker job ran the container, polled the HTTP endpoint for up to 60 seconds, verified the SQLite database file was created on the volume, and dumped container logs before any push to Docker Hub. This ensured broken images never reached the registry.

**Tag Strategy.** The pipeline generated multiple tags: latest for main/master pushes, branch names, sha prefixes, and semantic version tags for version tags. Pull requests triggered build and smoke test only (no push). Documentation-only changes were skipped.

**Build Caching.** Both jobs shared GitHub Actions layer caching (type=gha). The Dockerfile separated NuGet package restoration into its own layer before copying source code, ensuring code changes did not trigger package re-downloads.

## 5. Discussion

### 5.1 Interpretation of Results

The findings support the study's hypotheses. The security model demonstrated that enterprise-grade security features (role-based authorization, guard rails, audit logging, antiforgery protection) were fully implementable within the SQLite-based architecture without compromise. The security capabilities were framework-level (ASP.NET Core Identity, MVC filters, authorization policies) rather than database-level, making the database engine choice largely orthogonal to security implementation.

The containerization findings confirmed that embedded-database containerization requires fundamentally different persistence strategies from client-server database containerization. In a client-server architecture, the database runs as a separate container; in the embedded-database architecture, the application container itself is stateful, requiring careful volume mapping to achieve statelessness. The named volume strategy effectively separated state from container lifecycle, but the cookie key ring caveat demonstrated that all stateful components must be identified and persisted.

The CI/CD pipeline validated that automated smoke testing can reliably validate an embedded-database application without requiring database migration scripts or server provisioning. The application's automatic database creation and migration at startup eliminated the need for migration steps in the deployment pipeline.

### 5.2 Comparison with Prior Research

While Owens and Allen [3] defended SQLite's production viability, this study provides concrete evidence through a production-grade implementation with security, audit, and analytics requirements. The repository pattern implementation without stored procedures aligns with Fowler's [9] repository pattern but demonstrates the specific adaptation required when migrating from a stored-procedure-based architecture to an ORM-based one. The containerization findings extend the work of Merkel [12] and Miell and Sayers [13] by addressing embedded-database persistence patterns.

### 5.3 Theoretical Contributions

The study contributes the concept of "orthogonal security" -- the insight that security implementation in modern web frameworks is largely independent of database engine choice. It also identifies the concept of "state surface area" -- the complete set of stateful components that must be persisted for reliable container redeployment, extending beyond the database to include data protection keys and session state.

### 5.4 Limitations

The study is based on a single case implementation without comparative performance benchmarking. Concurrency evaluation is limited to WAL mode configuration without stress testing under high concurrent load. The CI/CD pipeline targets a single platform (linux/amd64) and registry (Docker Hub). The single-case design limits generalizability.

## 6. Conclusion

This study examined the design and implementation of a production-grade web application using ASP.NET Core MVC, SQLite, Docker, and CI/CD automation, employing a design-science research methodology. The findings demonstrate that the integration of a serverless embedded database with a mature MVC framework and modern DevOps practices yields a viable production-ready architecture for small-to-medium web applications. The study documented the architectural patterns (repository pattern without stored procedures, automatic database initialization, global filter-based observability), security model (three-policy authorization, guard rails, audit trail), database design decisions (WAL mode, busy timeout, single-file advantages), containerization strategies (named volume persistence, health check workaround, state surface identification), and CI/CD practices (pre-deployment smoke testing, automatic schema migration, build caching).

The theoretical contributions of orthogonal security and state surface area extend existing frameworks and provide conceptual tools for practitioners and researchers evaluating lightweight architectures. Future research should include comparative performance benchmarking between SQLite and client-server databases, empirical concurrency stress testing of WAL-mode SQLite, investigation of multi-architecture container builds, and extension to microservices architectures using embedded databases.

## References

[1] Microsoft, "ASP.NET Core Documentation," Microsoft Learn, 2026. [Online]. Available: https://learn.microsoft.com/aspnet/core.

[2] M. Hummer, C. Rosenberger, and J. Eder, "Continuous testing for continuous delivery: A systematic literature review," in Proceedings of the 34th ACM/SIGAPP Symposium on Applied Computing, 2019, pp. 1738-1745.

[3] M. Owens and G. Allen, The Definitive Guide to SQLite, 2nd ed. New York, NY, USA: Apress, 2010.

[4] J. A. Kreibich, Using SQLite: A Small, Fast, Self-Contained SQL Database Engine. Sebastopol, CA, USA: O'Reilly Media, 2010.

[5] A. R. Hevner, S. T. March, J. Park, and S. Ram, "Design science in information systems research," MIS Quarterly, vol. 28, no. 1, pp. 75-105, 2004.

[6] G. E. Krasner and S. T. Pope, "A cookbook for using the model-view-controller user interface paradigm in Smalltalk-80," Journal of Object-Oriented Programming, vol. 1, no. 3, pp. 26-49, 1988.

[7] T. Reenskaug, "Models-views-controllers," Xerox PARC Technical Note, 1979. [Online]. Available: https://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html.

[8] A. Freeman, Pro ASP.NET Core 3: Develop Cloud-Ready Web Applications Using ASP.NET Core 3. New York, NY, USA: Apress, 2020.

[9] M. Fowler, Patterns of Enterprise Application Architecture. Boston, MA, USA: Addison-Wesley, 2003.

[10] D. R. Hipp, "Write-ahead logging for SQLite," SQLite Documentation, 2010. [Online]. Available: https://www.sqlite.org/wal.html.

[11] M. D. Nardo, S. Ambler, C. Czenkus, et al., "Patterns of enterprise application architecture: Active record," Martin Fowler's website, 2007. [Online]. Available: https://martinfowler.com/eaaCatalog/activeRecord.html.

[12] D. Merkel, "Docker: Lightweight Linux containers for consistent development and deployment," Linux Journal, vol. 2014, no. 239, Art. 2, 2014.

[13] I. Miell and A. Sayers, Docker in Practice, 2nd ed. Shelter Island, NY, USA: Manning Publications, 2020.
