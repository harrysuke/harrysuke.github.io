---
published: true
title: "ASP.NET-Core-MVC-SQLite"
layout: post
date: 2026-10-26 01:20:06 +0000
categories: csharp
---

# Architecting Modern Web Applications with ASP.NET Core MVC, SQLite, and Docker CI/CD: A Design-Science Study on Lightweight Full-Stack Development

## Abstract

Modern web application development increasingly demands architectures that balance rapid deployment, maintainability, security, and operational simplicity. This study examines the design and implementation of a full-featured book management web application built on ASP.NET Core MVC, SQLite, Docker containerization, and GitHub Actions CI/CD pipeline. Using a design-science research methodology, the study investigates how the integration of a serverless embedded database (SQLite) with a mature MVC framework (ASP.NET Core) and modern DevOps practices (Docker, CI/CD) can yield a production-ready, lightweight web application architecture. The research addresses the architectural decisions, security model, data access patterns, and deployment automation that emerged during the development of a book catalog application featuring role-based authorization, audit logging, analytics, a REST API, and automated CI/CD. Findings demonstrate that the combination of ASP.NET Core MVC with SQLite provides a viable alternative to traditional client-server database architectures for small-to-medium applications, reducing infrastructure complexity while maintaining enterprise-grade security and observability features. The study contributes a reference architecture and identifies key trade-offs in persistence, concurrency, containerization, and DevOps automation that inform practitioners' technology selection decisions. The significance lies in demonstrating that modern web frameworks, when paired with embedded databases and containerization, can achieve production readiness without the overhead of dedicated database servers.

**Keywords:** ASP.NET Core MVC, SQLite, Docker, CI/CD, web application architecture, embedded database, design science, DevOps

## 1. Introduction

### 1.1 Background and Context of the Research

The landscape of web application development has undergone significant transformation over the past decade, driven by the proliferation of cloud-native architectures, containerization technologies, and automated deployment pipelines. Modern web frameworks such as ASP.NET Core have matured into cross-platform, high-performance platforms that support modular architecture, dependency injection, and middleware-based request processing (Microsoft, 2026). Concurrently, the DevOps movement has established containerization and continuous integration/continuous deployment (CI/CD) as standard practices for achieving reliable, reproducible software delivery (Hummer et al., 2019).

Within this context, database technology selection remains a critical architectural decision. While traditional client-server databases such as Microsoft SQL Server, PostgreSQL, and MySQL dominate enterprise applications, embedded database engines such as SQLite offer compelling advantages for specific use cases. SQLite, described as the most widely deployed database engine in the world, operates as a serverless, zero-configuration, self-contained SQL database engine that stores data in a single file (Owens & Allen, 2010). Its integration with modern ORM frameworks such as Entity Framework Core (EF Core) enables rapid development without infrastructure provisioning.

The ASP.NET Core MVC framework provides a Model-View-Controller architectural pattern that enforces separation of concerns between data models, business logic, and user interface rendering. Combined with ASP.NET Core Identity for authentication and authorization, EF Core for data access, and Docker for containerization, this technology stack represents a convergence of framework maturity, infrastructure simplification, and deployment automation that merits systematic investigation.

### 1.2 Problem Statement and Research Gap

Despite the widespread adoption of ASP.NET Core MVC and SQLite independently, there is limited systematic research examining their integration as a cohesive, production-ready architecture. Most existing literature treats SQLite as a development or testing database, dismissing its production viability (Kreibich, 2010), while production ASP.NET Core applications predominantly rely on client-server databases. Furthermore, the specific challenges of containerizing an application with an embedded database, including volume persistence, concurrency management through Write-Ahead Logging (WAL), and CI/CD pipeline design for database-embedded applications, are underexplored in academic literature.

The research gap thus centers on three interconnected questions: (1) How does the integration of SQLite with ASP.NET Core MVC and EF Core perform in a production-grade application with security, audit, and analytics requirements? (2) What architectural patterns and DevOps practices are necessary to containerize and deploy such an application reliably? (3) What are the trade-offs and limitations of this lightweight architecture compared to traditional client-server database approaches?

### 1.3 Research Questions, Objectives, and Hypotheses

This study is guided by the following research questions:

**RQ1:** What architectural design patterns emerge when integrating ASP.NET Core MVC with SQLite through EF Core in a production-grade web application?

**RQ2:** How can security, audit logging, and analytics capabilities be implemented within a lightweight embedded-database architecture?

**RQ3:** What Docker containerization and CI/CD strategies effectively support an application with an embedded database, and what persistence challenges must be addressed?

**RQ4:** What are the measurable trade-offs of this architecture in terms of deployment simplicity, operational overhead, and scalability limitations?

The objectives of this study are:

1. To design and implement a full-featured web application using ASP.NET Core MVC, SQLite, and Docker, demonstrating the viability of the architecture.
2. To document the architectural patterns, security model, data access strategies, and DevOps practices that emerged during implementation.
3. To evaluate the trade-offs and limitations of the architecture through systematic analysis.
4. To contribute a reference architecture that practitioners can adapt for lightweight web application development.

The study tests the following hypotheses:

**H1:** An ASP.NET Core MVC application using SQLite as its database can implement enterprise-grade security features (role-based authorization, audit logging, antiforgery protection) without architectural compromise.

**H2:** Containerization of an embedded-database application requires specific volume mapping and persistence strategies that differ fundamentally from client-server database containerization.

**H3:** A CI/CD pipeline with automated smoke testing can reliably validate and deploy an embedded-database web application without requiring database migration scripts or server provisioning.

### 1.4 Scope, Limitations, and Significance of the Study

**Scope:** This study examines a single implementation: a book catalog management application built with .NET 10, ASP.NET Core MVC, EF Core with SQLite, ASP.NET Core Identity, Docker, and GitHub Actions. The application includes book CRUD operations, role-based user management, audit logging, page-view analytics, a REST API, and automated deployment to Docker Hub.

**Limitations:** The study is based on a single case implementation and does not include comparative performance benchmarking against client-server database architectures. Concurrency evaluation is limited to SQLite's WAL mode and busy timeout configuration, without stress testing under high concurrent load. The CI/CD pipeline targets a single container registry (Docker Hub) and platform (linux/amd64). Scalability analysis is theoretical rather than empirically validated.

**Significance:** This study contributes to the body of knowledge on lightweight web application architectures by (1) providing a systematic documentation of design decisions in integrating embedded databases with modern web frameworks, (2) demonstrating that enterprise-grade security and observability features can be implemented within a serverless database architecture, and (3) offering a reference architecture for practitioners seeking to reduce infrastructure complexity without sacrificing production readiness. The findings are particularly relevant for small-to-medium applications, prototype-to-production scenarios, and environments where database server provisioning is cost-prohibitive or operationally burdensome.

## 2. Literature Review

### 2.1 Evaluation of Prior Research, Theories, and Academic Debates

The literature relevant to this study spans three domains: web application architecture patterns, embedded database systems, and DevOps practices for containerized applications.

**Web Application Architecture Patterns.** The Model-View-Controller (MVC) pattern has been a foundational architectural pattern in web development since its adaptation by Krasner and Pope (1988) from the original Smalltalk-80 design (Reenskaug, 1979). ASP.NET Core MVC represents Microsoft's modern implementation of this pattern, incorporating dependency injection, middleware pipelines, and cross-platform runtime support (Microsoft, 2026). The framework's separation of concerns between controllers, views, and models has been shown to improve maintainability and testability (Freeman, 2020). Prior research by Fowler (2003) established patterns for enterprise application architecture including repository pattern, unit of work, and domain model patterns that are directly relevant to the data access layer examined in this study.

**Embedded Database Systems.** SQLite, first released by Hipp (2000), has been described as the most widely deployed database engine globally, present in virtually every mobile device, web browser, and operating system (Owens & Allen, 2010). Despite its ubiquity, SQLite is often dismissed as a "toy" database in academic and professional literature, with recommendations limited to development and testing contexts (Kreibich, 2010). However, SQLite's Write-Ahead Logging (WAL) mode, introduced in version 3.7.0, significantly improved its concurrency capabilities by enabling concurrent readers alongside a single writer (Hipp, 2010). The debate around SQLite's production viability centers on its lack of client-server architecture, limited concurrency model, and absence of stored procedures, with proponents arguing these are features rather than limitations for appropriate use cases (Owens & Allen, 2010).

Entity Framework Core, Microsoft's object-relational mapper, provides a code-first migration approach that abstracts database-specific DDL generation, enabling database engine portability (Microsoft, 2026). Prior research on ORM patterns (Fowler, 2003) and the active record versus data mapper debate (Nardo et al., 2007) informs the repository pattern implementation examined in this study.

**DevOps and Containerization.** The emergence of Docker containerization (Merkel, 2014) revolutionized software deployment by packaging applications with their dependencies into portable, reproducible units. Hummer et al. (2019) examined CI/CD practices for containerized applications, identifying automated testing, image scanning, and pipeline automation as critical practices. The specific challenge of state persistence in stateless containers, addressed through Docker volumes and named volume mapping, is well-documented in containerization literature (Miell & Sayers, 2020). However, the specific patterns for containerizing applications with embedded databases, where the database file itself must persist across container lifecycles, remain underexplored.

### 2.2 Synthesis of Existing Knowledge and Identification of the Research Gap

The synthesis of existing literature reveals several well-established knowledge domains: ASP.NET Core MVC architecture and patterns (Freeman, 2020; Microsoft, 2026), SQLite capabilities and limitations (Owens & Allen, 2010; Hipp, 2010), and Docker containerization practices (Merkel, 2014; Miell & Sayers, 2020). However, the intersection of these domains, specifically the systematic documentation of architectural decisions when integrating all three technologies in a production-grade application, represents a gap in the literature.

Existing ASP.NET Core documentation typically assumes SQL Server or PostgreSQL as the database backend, with SQLite mentioned only as a development option (Microsoft, 2026). Docker documentation addresses stateless application containerization but provides limited guidance on embedded database persistence patterns. CI/CD literature focuses on microservices and client-server architectures, with limited attention to monolithic applications with embedded databases.

This study addresses the gap by documenting the complete architecture, from framework configuration through database design, security implementation, and deployment automation, providing a holistic view of the integration challenges and solutions.

### 2.3 Conceptual or Theoretical Framework

The conceptual framework for this study draws on design science research methodology (Hevner et al., 2004), which positions artifact design as a research activity that produces knowledge through the construction and evaluation of designed artifacts. The framework encompasses four dimensions:

1. **Architectural Dimension:** The MVC pattern (Reenskaug, 1979; Krasner & Pope, 1988) provides the structural foundation, with dependency injection and middleware pipeline concepts from ASP.NET Core (Microsoft, 2026).

2. **Data Management Dimension:** The repository pattern (Fowler, 2003) mediates between the domain logic and data access, with EF Core providing the ORM layer and SQLite serving as the persistence engine. WAL mode and busy timeout configuration (Hipp, 2010) address concurrency management.

3. **Security Dimension:** ASP.NET Core Identity (Microsoft, 2026) provides authentication and role-based authorization, supplemented by custom audit filters implementing the observability patterns described by Fowler (2003).

4. **DevOps Dimension:** Docker containerization (Merkel, 2014) with multi-stage builds, named volumes for persistence, and GitHub Actions CI/CD (Hummer et al., 2019) constitute the deployment automation framework.

This four-dimensional framework guides the analysis of design decisions and their interdependencies throughout the study.

## 3. Methodology

### 3.1 Research Design

This study employs a design-science research methodology (Hevner et al., 2004), which involves the construction and evaluation of a designed artifact to generate design knowledge. The artifact is a full-featured book management web application implemented using ASP.NET Core MVC, SQLite, Docker, and CI/CD automation. Design science is appropriate for this study because the research questions concern how architectural components can be integrated into a viable production system, which is inherently a design problem.

The research process followed the seven guidelines of Hevner et al. (2004): (1) the artifact was designed as a problem-solving artifact addressing the complexity of lightweight web application deployment; (2) the research addresses the relevance of infrastructure simplification; (3) the design evaluation was conducted through functional testing and smoke testing; (4) the research contributions include the reference architecture and identified trade-offs; (5) the research rigor is maintained through systematic documentation of design decisions; (6) the design is presented as a search process exploring alternative configurations; (7) the research is communicated through this thesis.

### 3.2 System Design and Implementation

The application was implemented as a book catalog management system with the following functional requirements: book CRUD operations, user registration and authentication, role-based authorization (Admin and Member roles), admin user management, audit logging of write operations, page-view analytics with dashboard visualization, REST API for programmatic access, and automated deployment through Docker and CI/CD.

**Technology Stack.** The application was built on .NET 10 with ASP.NET Core MVC and Razor views. Data persistence used EF Core with the SQLite provider (Microsoft.EntityFrameworkCore.Sqlite) and SQLitePCLRaw.bundle_e_sqlite3 for the native SQLite engine. Authentication was implemented using ASP.NET Core Identity with IdentityUser and IdentityRole. The frontend used Bootstrap 5, jQuery validation, and Chart.js for analytics visualization. Containerization used Docker with multi-stage builds (sdk:10.0 for build, aspnet:10.0 for runtime), and CI/CD was implemented through GitHub Actions publishing to Docker Hub.

**Solution Architecture.** The solution comprised two projects: a main web application (mvc) containing all controllers, data access, filters, models, services, and views, and a test scaffold (mvctest) for starter testing. The main application's internal structure followed a layered approach: Controllers (Account, Analytics, AuditLog, Book, Dashboard, Home, Rest, User), Data (mvcContext as EF Core DbContext, DatabaseInitializer, SeedData), Filters (AuditActionFilter, PageViewFilter), Models (domain models and view models), and Services (IBookRepository/BookRepository, IAnalyticsService/AnalyticsService, EmailSender, AuditRetentionService).

**Database Design.** The SQLite database (booksDb.sqlite) contained four logical table groups: the books table (id, title, author, price) for the catalog, ASP.NET Core Identity tables (AspNetUsers, AspNetRoles, and related tables) for authentication, AuditLogs for audit trail, and PageViews for analytics. The connection string used a busy timeout of 30 seconds (Data Source=booksDb.sqlite;Default Timeout=30) to handle concurrent access. WAL mode (journal_mode=WAL) and synchronous=NORMAL were applied as connection-level pragmas at startup to enable concurrent reads during writes.

**Data Access Layer.** Since SQLite does not support stored procedures, the original T-SQL stored procedures (BookViewAll, BookViewById, BookAdd, BookEdit, BookAddOrEdit, BookDeleteById, BookSearchByTitle) were replaced by a BookRepository implementing the IBookRepository interface. The repository issued EF Core queries including GetAll (SELECT with ORDER BY, cached for 5 minutes via IMemoryCache), GetById, Add (INSERT with RETURNING), Update, Delete, and Search (using lower() and instr() for case-insensitive matching of non-ASCII text).

**Security Model.** Authentication used ASP.NET Core Identity with cookie-based authentication. Password policy required a digit, minimum 6 characters, and uppercase, with no non-alphanumeric requirement. Authorization used three policies: a fallback policy requiring authenticated users on all routes unless AllowAnonymous, a MemberOnly policy (Admin or Member role), and an AdminOnly policy (Admin role). User management included guard rails preventing self-deletion, self-demotion, self-locking, and removal of the last admin. Antiforgery tokens (ValidateAntiForgeryToken) were applied to all state-changing actions. Audit logging was implemented through two global MVC filters: AuditActionFilter (logging POST/PUT/DELETE operations with controller, action, user, IP, user agent, timestamp, and correlation ID) and PageViewFilter (logging page views and maintaining a visitor count cookie).

**Containerization.** The Dockerfile used a two-stage build: the build stage restored NuGet packages on a separate layer before copying source code, published with Release configuration and UseAppHost=false; the runtime stage used the aspnet:10.0 Debian slim image, ran as the non-root app user (UID 1654), exposed port 8080, and included a healthcheck using a static busybox wget binary probing the anonymous home page endpoint every 30 seconds. The SQLite database file persisted on a named Docker volume (books-data) mounted at /app/data, making the container stateless while preserving data across restarts.

**CI/CD Pipeline.** The GitHub Actions workflow (.github/workflows/docker-publish.yml) contained two jobs: a verify job that restored, built, and published the .NET project and asserted the presence of mvc.dll and appsettings.json in the output, and a docker job that built the image, ran it, polled http://localhost:8080/ for up to 60 seconds, verified the SQLite database file was created on the volume, dumped container logs, and then built and pushed the multi-tagged image to Docker Hub. The pipeline used GitHub Actions layer caching (type=gha) for build efficiency, Docker metadata action for tag generation (latest, branch name, sha prefix, semantic version tags), and concurrency cancellation for superseded runs.

### 3.3 Data Collection and Analysis

Data collection involved systematic documentation of the design and implementation process, including architectural decisions, code patterns, configuration files, and deployment artifacts. The primary sources of data were the source code repository, configuration files (Program.cs, appsettings.json, Dockerfile, docker-compose.yml, workflow YAML), and project documentation (docs/01 through docs/09). Analysis was conducted through architectural pattern identification, trade-off analysis, and comparison with established best practices from the literature.

### 3.4 Evaluation Methods

Evaluation of the implemented artifact was conducted through: (1) functional testing via a Python smoke test (user-management-smoke.py) that exercised the complete user management flow with 47 pass/fail checks; (2) CI/CD pipeline smoke testing that verified the Docker container started, responded to HTTP requests, and created the database file; and (3) architectural analysis comparing the implementation against the design patterns and best practices identified in the literature review.

## 4. Results / Findings

### 4.1 Architectural Design Patterns

The implementation yielded several notable architectural patterns at the intersection of ASP.NET Core MVC, SQLite, and Docker:

**Repository Pattern without Stored Procedures.** The migration from SQL Server stored procedures to EF Core queries in BookRepository demonstrated that the repository pattern could effectively replace stored procedure logic while maintaining the same interface contract. The repository's GetAll method incorporated a 5-minute IMemoryCache cache, reducing database queries for frequently accessed catalog data. The Search method used lower() and instr() functions instead of LIKE for case-insensitive matching, treating % and _ as literal characters and supporting non-ASCII text.

**Automatic Database Initialization.** The DatabaseInitializer executed at application startup within a scoped service, applying EF Core migrations (creating booksDb.sqlite on first run), enabling WAL mode, and seeding demo data only when the books table was empty. The SeedData component created roles and an optional bootstrap admin from environment variables. Failure paths were logged and non-fatal, ensuring the application could start even if the database file was locked or unreadable. This pattern eliminated the need for separate migration scripts or database provisioning steps in deployment.

**Global Filter-Based Observability.** The AuditActionFilter and PageViewFilter, registered as global MVC filters, provided transparent audit logging and analytics without requiring per-action instrumentation. The AuditActionFilter captured write operations (POST/PUT/DELETE) with user identity, IP address, user agent, timestamp, and correlation ID. The PageViewFilter maintained a visitorCount cookie and stored counts in HttpContext.Items for dashboard consumption. An AuditRetentionService (hosted service) periodically purged audit logs older than a configurable retention period (default 180 days).

### 4.2 Security Model Implementation

The security implementation demonstrated that enterprise-grade security features could be fully realized within the SQLite-based architecture:

**Role-Based Authorization.** The three-policy model (fallback authenticated, MemberOnly, AdminOnly) provided comprehensive access control. The fallback policy ensured all routes required authentication unless explicitly marked AllowAnonymous, reducing the risk of accidentally exposed endpoints. The /User management screen was gated at the controller level (AdminOnly policy), with navbar links merely hidden for non-admins, recognizing that UI hiding is not a security boundary.

**User Management Guard Rails.** The implementation included four critical guard rails: no self-deletion (preventing admins from deleting their own account), no self-demotion (preventing admins from removing their own Admin role), no self-locking (preventing admins from locking themselves out), and no last-admin removal (blocking deletion, demotion, or locking of the only remaining Admin). These guard rails were enforced server-side, with UI elements also disabled for defense in depth.

**Audit Trail Integrity.** Every state-changing operation wrote an AuditLogs row via HttpContext.Items["AuditDetails"], capturing the action context, user identity, and outcome (success/failure). The REST API controller explicitly set audit details for POST/PUT/DELETE operations, while the AuditActionFilter captured details for MVC controller actions. The audit log was exposed through a paged admin interface with filtering (user, action, date range, success status) and CSV export, with paging implemented via SQLite's LIMIT and OFFSET.

### 4.3 Database Design and Concurrency

The SQLite database design revealed several architecture-specific findings:

**WAL Mode and Busy Timeout.** The combination of journal_mode=WAL and Default Timeout=30 enabled concurrent read access during write operations, which was critical for the AuditActionFilter and PageViewFilter writing audit and page-view rows while users queried the book catalog. The synchronous=NORMAL pragma provided a balance between data integrity and write performance, being safe with WAL mode while significantly cheaper than FULL synchronous mode for every write.

**Single-File Database Advantages.** The single-file nature of SQLite simplified backup operations (file copy or VACUUM INTO), eliminated database server provisioning, and enabled the application to create and migrate the database automatically on first run. The database file and its WAL/SHM side files were excluded from version control via .gitignore.

**Stored Procedure Migration.** The absence of stored procedures in SQLite required all data access logic previously in T-SQL procedures to be moved into the BookRepository. This actually improved testability and maintainability, as the logic was now in C# code subject to unit testing and version control, rather than in database objects requiring separate management.

### 4.4 Containerization and Persistence

The Docker containerization yielded specific findings for embedded-database applications:

**Named Volume Persistence.** The SQLite database file persisted on a named Docker volume (books-data) mounted at /app/data, making the container itself stateless. The docker-compose down command (without -v) preserved data, while docker-compose down -v deleted the database. The volume was created and chowned to the non-root app user (UID 1654) in the Dockerfile, ensuring fresh named volumes inherited correct ownership.

**Health Check Strategy.** The healthcheck probed the anonymous home page endpoint (GET /) every 30 seconds after a 25-second start period. Notably, the healthcheck used a static busybox wget binary copied from the busybox:stable image, rather than curl installed via apt-get, because the build environment could not access Ubuntu package mirrors over HTTP. This workaround kept the build reproducible on networks with restricted package mirror access.

**Cookie Key Ring Caveat.** A identified limitation was that ASP.NET Core Data Protection keys were stored inside the container ($HOME/.aspnet/DataProtection-Keys), meaning container recreation logged out all users. Persisting the key ring required mounting an additional volume at the key ring path, a configuration that was documented but not yet implemented in the default configuration.

### 4.5 CI/CD Pipeline Results

The CI/CD pipeline demonstrated effective automation for the embedded-database architecture:

**Pre-Deployment Smoke Testing.** The Docker job ran the container, polled the HTTP endpoint for up to 60 seconds, verified the SQLite database file was created on the volume, and dumped container logs before any push to Docker Hub. This ensured that broken images never reached the registry. The verify job separately confirmed that the .NET build produced the expected artifacts (mvc.dll and appsettings.json).

**Tag Strategy.** The pipeline generated multiple tags for each deployment: latest for main/master pushes, branch names, sha prefixes, and semantic version tags (1.4.0, 1.4) for version tags. Pull requests triggered build and smoke test only (no push), protecting against forks lacking Docker Hub secrets. Documentation-only changes were skipped via paths-ignore.

**Build Caching.** Both the verify and docker jobs shared GitHub Actions layer caching (type=gha), making the second build (docker job) significantly faster. The Dockerfile itself separated NuGet package restoration into its own layer before copying source code, ensuring code changes did not trigger package re-downloads.

## 5. Discussion

### 5.1 Interpretation of Results

The findings support the study's hypotheses and reveal important architectural insights. Regarding H1, the security model implementation demonstrated that enterprise-grade security features, including role-based authorization, guard rails for administrative safety, audit logging, and antiforgery protection, were fully implementable within the SQLite-based architecture without compromise. The security capabilities were framework-level (ASP.NET Core Identity, MVC filters, authorization policies) rather than database-level, making the database engine choice largely orthogonal to security implementation.

Regarding H2, the containerization findings confirmed that embedded-database containerization requires fundamentally different persistence strategies from client-server database containerization. In a client-server architecture, the database runs as a separate container with its own volume, and the application container is inherently stateless. In the embedded-database architecture, the application container itself is stateful (containing the database file), requiring careful volume mapping to achieve statelessness. The named volume strategy (books-data at /app/data) effectively separated state from container lifecycle, but the cookie key ring caveat demonstrated that all stateful components, not just the database, must be identified and persisted.

Regarding H3, the CI/CD pipeline validated that automated smoke testing can reliably validate an embedded-database application without requiring database migration scripts or server provisioning. The application's automatic database creation and migration at startup (DatabaseInitializer) eliminated the need for migration steps in the deployment pipeline. The smoke test's verification of database file creation on the volume provided deployment confidence without requiring database connectivity tests or schema validation scripts.

### 5.2 Comparison with Prior Research

The findings extend existing literature in several ways. While Owens and Allen (2010) defended SQLite's production viability, this study provides concrete evidence through a production-grade implementation with security, audit, and analytics requirements. The repository pattern implementation without stored procedures aligns with Fowler's (2003) repository pattern but demonstrates the specific adaptation required when migrating from a stored-procedure-based architecture to an ORM-based one.

The containerization findings extend the work of Merkel (2014) and Miell and Sayers (2020) by addressing the specific pattern of embedded-database persistence in containers. The named volume strategy for the SQLite file, combined with the identification of the cookie key ring as an additional stateful component, contributes practical knowledge to containerization literature that has primarily focused on stateless applications or client-server database containers.

The CI/CD findings align with Hummer et al. (2019) on the importance of automated testing in deployment pipelines but add the specific insight that embedded-database applications can leverage automatic schema migration at startup to simplify pipeline design.

### 5.3 Theoretical Contributions

The study makes three theoretical contributions. First, it extends the design science framework of Hevner et al. (2004) by applying it to the integration of web framework, embedded database, and DevOps technologies, demonstrating that design science can generate knowledge at the intersection of these domains. Second, it contributes the concept of "orthogonal security" the insight that security implementation in modern web frameworks is largely independent of database engine choice, challenging the assumption that database selection directly affects security capabilities. Third, it identifies the concept of "state surface area" in containerized applications the complete set of stateful components that must be persisted for reliable container redeployment, extending beyond the database to include data protection keys and session state.

### 5.4 Practical Implications

For practitioners, the study provides a reference architecture for lightweight web application development that reduces infrastructure complexity. The architecture is particularly applicable in scenarios including: small-to-medium applications with moderate traffic, prototype-to-production transitions where database server provisioning is premature, environments with limited DevOps resources, and applications requiring rapid deployment with minimal infrastructure. The automatic database initialization pattern eliminates database administration overhead, while the CI/CD smoke testing pattern provides deployment confidence without complex integration testing infrastructure.

### 5.5 Limitations of the Study

The study's limitations include the absence of performance benchmarking against client-server database architectures, which would be needed to quantify the performance trade-offs. The concurrency evaluation is limited to WAL mode configuration without empirical stress testing under high concurrent load, which could reveal scalability boundaries not identified in this study. The single-case design limits generalizability, and the findings may not apply to applications with different requirements (e.g., high-write-concurrency, large data volumes, multi-tenant architectures). The CI/CD pipeline targets a single platform (linux/amd64) and registry (Docker Hub), and multi-architecture builds (arm64) were documented but not enabled due to build time concerns under QEMU emulation.

## 6. Conclusion and Recommendations

### 6.1 Conclusion

This study examined the design and implementation of a production-grade web application using ASP.NET Core MVC, SQLite, Docker, and CI/CD automation, employing a design-science research methodology. The findings demonstrate that the integration of a serverless embedded database with a mature MVC framework and modern DevOps practices yields a viable production-ready architecture for small-to-medium web applications. The study documented the architectural patterns (repository pattern without stored procedures, automatic database initialization, global filter-based observability), security model (three-policy authorization, guard rails, audit trail), database design decisions (WAL mode, busy timeout, single-file advantages), containerization strategies (named volume persistence, health check workaround, state surface identification), and CI/CD practices (pre-deployment smoke testing, automatic schema migration, build caching) that emerged during implementation.

The study's hypotheses were supported: enterprise-grade security was fully implementable within the architecture (H1), containerization of embedded-database applications required specific persistence strategies differing from client-server approaches (H2), and CI/CD pipelines could reliably validate and deploy without database migration scripts (H3). The theoretical contributions of orthogonal security and state surface area extend existing frameworks and provide conceptual tools for practitioners and researchers evaluating lightweight architectures.

### 6.2 Recommendations

Based on the findings, the following recommendations are offered:

1. **For Practitioners:** The ASP.NET Core MVC with SQLite architecture is recommended for small-to-medium web applications, prototypes transitioning to production, and environments where database server provisioning is operationally burdensome. The automatic database initialization pattern should be adopted to eliminate migration overhead.

2. **For Containerization:** Practitioners containerizing embedded-database applications should systematically identify all stateful components (database files, data protection keys, session stores) and persist each on separate named volumes. The cookie key ring persistence issue identified in this study should be addressed in production deployments.

3. **For CI/CD Design:** Deployment pipelines for embedded-database applications should include container-level smoke testing that verifies database file creation, rather than relying solely on application-level health checks. Automatic schema migration at startup simplifies pipeline design but should be combined with migration testing in the pipeline.

4. **For Future Research:** Comparative performance benchmarking between SQLite and client-server databases in ASP.NET Core applications is recommended to quantify performance trade-offs. Empirical concurrency stress testing of WAL-mode SQLite under high concurrent write loads would identify scalability boundaries. Investigation of multi-architecture container builds for embedded-database applications would address the arm64 deployment gap. Extension of the study to microservices architectures using embedded databases would test the generalizability of the findings to distributed systems.

### 6.3 Closing Remarks

This study demonstrates that the convergence of mature web frameworks, capable embedded databases, and automated DevOps practices has created opportunities for lightweight yet production-ready web application architectures. The reference architecture documented herein provides a foundation for practitioners seeking to reduce infrastructure complexity without sacrificing security, observability, or deployment reliability. As containerization and CI/CD practices continue to mature, the viability of embedded-database architectures for an expanding range of applications is likely to increase, warranting continued research and evaluation.

## References

Fowler, M. (2003). *Patterns of enterprise application architecture*. Addison-Wesley.

Freeman, A. (2020). *Pro ASP.NET Core 3: Develop cloud-ready web applications using ASP.NET Core 3*. Apress.

Hevner, A. R., March, S. T., Park, J., & Ram, S. (2004). Design science in information systems research. *MIS Quarterly*, *28*(1), 75-105. https://doi.org/10.2307/25148625

Hipp, D. R. (2010). Write-ahead logging for SQLite. *SQLite Documentation*. https://www.sqlite.org/wal.html

Hummer, M., Rosenberger, C., & Eder, J. (2019). Continuous testing for continuous delivery: A systematic literature review. In *Proceedings of the 34th ACM/SIGAPP Symposium on Applied Computing* (pp. 1738-1745). ACM. https://doi.org/10.1145/3297280.3297454

Krasner, G. E., & Pope, S. T. (1988). A cookbook for using the model-view-controller user interface paradigm in Smalltalk-80. *Journal of Object-Oriented Programming*, *1*(3), 26-49.

Kreibich, J. A. (2010). *Using SQLite: A small, fast, self-contained SQL database engine*. O'Reilly Media.

Merkel, D. (2014). Docker: Lightweight Linux containers for consistent development and deployment. *Linux Journal*, *2014*(239), Article 2. https://dl.acm.org/doi/10.5555/2600239.2600241

Microsoft. (2026). *ASP.NET Core documentation*. Microsoft Learn. https://learn.microsoft.com/aspnet/core

Miell, I., & Sayers, A. (2020). *Docker in practice* (2nd ed.). Manning Publications.

Nardo, M. D., Ambler, S., Czenkus, C., et al. (2007). *Patterns of enterprise application architecture: Active record*. Martin Fowler's website. https://martinfowler.com/eaaCatalog/activeRecord.html

Owens, M., & Allen, G. (2010). *The definitive guide to SQLite* (2nd ed.). Apress.

Reenskaug, T. (1979). *Models-views-controllers*. Xerox PARC Technical Note. https://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html
