# Master Consolidated To-Do & Topics List
*This document contains every minute detail, topic, and sub-topic extracted from all notes, daily EODs, and session files across the repository.*

## 1. Pre-Orders & Business Requirements (OMS Redesign)
- [ ] Read blogs and understand the business requirement for Pre-Orders.
- [ ] Identify entities required for Pre-Orders in OMS.
- [ ] Check internal systems and data models for Pre-Orders.
- [ ] Write the initial MySQL query for the Pre-Orders business case.
- [ ] Create a real-world use case specifically for Pre-Orders.

## 2. Apache OFBiz Setup & Environment
- [ ] Complete local setup of Apache OFBiz 24.9 (`release24.09` branch).
- [ ] Understand the meaning behind setup commands (`init-gradle-wrapper.sh`, `gradlew cleanAll loadAll`, `gradlew ofbiz`).
- [ ] Learn about build tool concepts (Gradle wrapper) and dependency management.
- [ ] Start and complete Assignment 4 (Apache Moqui / OFBiz framework).
- [ ] Visit the Apache website to check bug mirroring and issue tracking.

## 3. Data Modeling & Database (DBMS)
### Party Data Model
- [ ] Complete the "Party data model assignment" (by the weekend).
- [ ] Study Chapter 2 of the Data Model Resource Book.
- [ ] Focus on Relationships, Roles, Contact Mechanisms, and Communication Events.
- [ ] Complete Activity 3 and Activity 4 of Module 4 (Party Activity).
### Product Data Model
- [ ] Study Chapter 3 of the Data Model Resource Book ("Products").
- [ ] Verify and complete the Product entity understanding.
- [ ] Master the 12-step sequence: Product, ProductType, GoodIdentification, ProductPrice, ProductAssoc, ProductCategory, ProductFeature, etc.
- [ ] Understand variations: Variants, Kits, Bundles, and Product Features.
### DBMS & SQL Practice
- [ ] Study DBMS architecture, fundamentals, and Database Instances.
- [ ] Practice SQL queries on JOINS, NESTED GROUP BY, and CTEs (Common Table Expressions).
- [ ] Write queries for data validation and debugging.

## 4. Security, Authentication & User Management
- [ ] Track down `UserLogin` entities by matching the UI to the actual database.
- [ ] Validate `UserLogin` entity behavior through code and database mapping.
- [ ] Understand Security entities: `Security_permission`, `Security_Group`, `UserLoginSecurityGroup`.
- [ ] Dive into Single Sign-On (SSO):
  - [ ] Check where SSO is used in the code.
  - [ ] Review how UserLogin works behind the scenes in the Java framework.
  - [ ] Learn about SSO identity providers, SAML, OpenID Connect, and Atlassian Crowd.
- [ ] Study Encryption Algorithms: 1-way vs 2-way encryption, SHA 256/512, AES, RSA, ECC.
- [ ] Understand Token generation and calling REST APIs securely.

## 5. Order Lifecycle & Workflows
- [ ] Explore and document the complete Order Lifecycle (Created > Broker > Allocate > Ship > Complete > Shopify Sync).
- [ ] Review all order lifecycle diagrams and system flows.
- [ ] Study the Point of Sale (POS) flow, ATP (Available to Promise), and QOH (Quantity on Hand) decrements.
- [ ] Work on **Transfer Order UML** diagrams.
- [ ] Study State Transition Entities: `InventoryItemDetail` (IID) and `OrderFacilityChange`.

## 6. Shopify Integration & Product Sync
- [ ] Create a real-world use case for Product Sync.
- [ ] Trace the service execution flow for Product Import.
- [ ] Understand Bulk Product Import flow and analyze related system messages.
- [ ] Perform code tracing of the **Shopify Order Integration Flow**:
  - [ ] Clarify and trace the pipeline: Shopify -> Webhook (`orders/updated`) -> EventBridge -> SQS.
  - [ ] Study GraphQL payload generation via FTL wrapper templates.
  - [ ] Review the Facade layer that sends the cleaned HTTP POST request back to Shopify.
- [ ] Understand Shopify `ShopifyShop` entities and compare JSON payloads for status variations.
- [ ] Note the bug discovered: Deprecated field `LineItem.FulfillmentService`.

## 7. API, GraphQL & Microservices
- [ ] Download and use Postman for extensive REST API testing.
- [ ] Study Microservices communication mechanisms and how services interact.
- [ ] Understand AWS SQS (Queue-based asynchronous communication flows).
- [ ] Study GraphQL fundamentals: queries, schema design, resolver flows, mutations, subscriptions.
- [ ] Learn to handle large-scale data via Shopify Bulk Operations (GraphQL Admin API).

## 8. Playwright Automation & Testing
- [ ] Learn Playwright basics, browser launch configurations (headed vs headless).
- [ ] Explore network interception, API calling mechanisms, and monitoring.
- [ ] Study selectors, assertions, test structuring, and End-to-End (E2E) integration.
- [ ] Write automation scripts for practical testing scenarios and API validation.

## 9. General Engineering & Other Assignments
- [ ] **SaaSTech Academy**: Begin Module 0.
- [ ] **Reconciliation**: Finalize and wrap up the Reconciliation assignment.
- [ ] **UML**: Read "UML Distilled" (Complete Chapter 1, start Chapter 2).
- [ ] **Receiving App**: Clone GitHub repo, install dependencies, setup local env, fix bugs.
- [ ] **Figma**: Explore simple UI/UX concepts using Figma.
- [ ] **PR Maintenance**: Correct GitHub PR issues (e.g., changing global image URLs to local).
- [ ] **Documentation**: Revise Gift Card, Returns, Exchanges, and BOPIS lifecycles. Update store operations docs.
- [ ] **Reading/Blogs**: Read 1 blog per week by the end of the month (Anil Sir's assignment).
- [ ] **New Keywords**: Review Wave Picking, SSO, tool factory pattern.


## 10. Insights & Notes from Anil Sir dated may 09 2026
- **Business Automation & Supply Chain**: Core focus areas for the system.
- **ORM Comparison (OFBiz Entity Engine vs. Hibernate)**: 
  - **Hibernate** maps static Java classes (POJOs) to database tables, which requires compiling classes for every entity.
  - **OFBiz Entity Engine** uses XML definitions and a dynamic dictionary approach (`GenericValue` maps). This makes it highly flexible, metadata-driven, and allows changing the data model without needing to recompile Java code.
- **Database Choices (Oracle vs. MySQL vs. PostgreSQL)**:
  - Why HotWax uses MySQL: It is heavily optimized for read-heavy operations, aligns with the "freedom philosophy" (open-source), and offers simplicity at scale.
  - CRUD transaction management has historically been a strong point for PostgreSQL, but MySQL's combination of simplicity and scalability won out.
- **Abstract Syntax Trees (AST)**: Compiles to SQL (or other querying languages), providing essential database independence.

