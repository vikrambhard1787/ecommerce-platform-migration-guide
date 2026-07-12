# Current State Architecture

## Overview

The current eCommerce platform is built on a monolithic architecture using Hybris/ATG framework, a market-leading Java-based commerce platform. The system handles all aspects of the eCommerce business logic within a single application deployment, connected to Oracle DB, with third-party integrations for specialized services like Order Management System (OMS), Tax calculation, Email delivery, Payments, and Product Information Management (PIM).

## Existing Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          Load Balancer                           │
│                  (Distributes incoming traffic)                  │
└────────────────┬─────────────────────────────┬───────────────────┘
                 │                             │
        ┌────────▼──────────┐        ┌────────▼──────────┐
        │   Application     │        │   Application     │
        │  Instance 1       │        │  Instance 2       │
        │ (Vertical Scaling)│        │ (Vertical Scaling)│
        │                   │        │                   │
        │ ┌───────────────┐ │        │ ┌───────────────┐ │
        │ │    Frontend   │ │        │ │    Frontend   │ │
        │ │ JSP / React   │ │        │ │ JSP / React   │ │
        │ └───────────────┘ │        │ └───────────────┘ │
        │                   │        │                   │
        │ ┌───────────────┐ │        │ ┌───────────────┐ │
        │ │   Hybris/ATG  │ │        │ │   Hybris/ATG  │ │
        │ │  (Java-based) │ │        │ │  (Java-based) │ │
        │ │               │ │        │ │               │ │
        │ │  ┌─────────┐  │ │        │ │  ┌─────────┐  │ │
        │ │  │ Product │  │ │        │ │  │ Product │  │ │
        │ │  │ Catalog │  │ │        │ │  │ Catalog │  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │ Search  │  │ │        │ │  │ Search  │  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │My Account   │ │        │ │  │My Account   │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │  Cart   │  │ │        │ │  │  Cart   │  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │Checkout │  │ │        │ │  │Checkout │  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │Payments │  │ │        │ │  │Payments │  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │  Order  │  │ │        │ │  │  Order  │  │ │
        │ │  │Placement   │ │        │ │  │Placement   │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │  Order  │  │ │        │ │  │  Order  │  │ │
        │ │  │Processing  │ │        │ │  │Processing  │ │
        │ │  ├─────────┤  │ │        │ │  ├─────────┤  │ │
        │ │  │  Email  │  │ │        │ │  │  Email  │  │ │
        │ │  │Notif.   │  │ │        │ │  │Notif.   │  │ │
        │ │  └─────────┘  │ │        │ │  └─────────┘  │ │
        │ └───────────────┘ │        │ └───────────────┘ │
        │                   │        │                   │
        │ ┌───────────────┐ │        │ ┌───────────────┐ │
        │ │  Cache Layer  │ │        │ │  Cache Layer  │ │
        │ │ (ATG Cache/   │ │        │ │ (ATG Cache/   │ │
        │ │  EhCache)     │ │        │ │  EhCache)     │ │
        │ └───────────────┘ │        │ └───────────────┘ │
        │                   │        │                   │
        │ ┌───────────────┐ │        │ ┌───────────────┐ │
        │ │  Auth Layer   │ │        │ │  Auth Layer   │ │
        │ │(LDAP/OAuth)   │ │        │ │(LDAP/OAuth)   │ │
        │ └───────────────┘ │        │ └───────────────┘ │
        └───────────┬───────┘        └───────────┬───────┘
                    │                            │
        ┌───────────┴────────────────────────────┴────────────┐
        │                                                      │
        │             Shared Oracle Database                  │
        │   (All application data tightly coupled)            │
        │                                                      │
        └──────────────┬──────────────┬──────────────┬─────────┘
                       │              │              │
        ┌──────────────▼──┐ ┌─────────▼──┐ ┌────────▼──────────┐
        │   Search Index  │ │  Messaging │ │  Centralized     │
        │  (Solr/Endeca)  │ │  (Sync Msg)│ │  Logging         │
        └─────────────────┘ └────────────┘ └──────────────────┘
        
        │
        │ Deployment: On-Prem VMs or Cloud VMs
        │

        ┌─────────────────────────────────────────────────────┐
        │         Third-Party Integrations (Sync)              │
        │                                                      │
        │  ┌──────────────┐  ┌──────────┐  ┌──────────────┐  │
        │  │     OMS      │  │   Tax    │  │ Email Service│  │
        │  │ (Order Mgmt) │  │(Calculate)  │ (Delivery)   │  │
        │  └──────────────┘  └──────────┘  └──────────────┘  │
        │                                                      │
        │  ┌──────────────┐  ┌──────────────────────────────┐  │
        │  │   Payments   │  │      PIM (Product Info)      │  │
        │  │  (Processing)│  │   (External Master Data)     │  │
        │  └──────────────┘  └──────────────────────────────┘  │
        └─────────────────────────────────────────────────────┘
```

## Technology Stack

### Frontend Layer
- **JSP (JavaServer Pages)** - Server-side rendering
- **React** - Modern JavaScript framework for interactive components

### Backend Layer
- **Hybris/ATG** - Java-based commerce platform
- **Java** - Core application language

### Database Layer
- **Oracle DB** - Shared relational database (primary data store)

### Caching Layer
- **ATG Built-in Cache** - ATG framework caching mechanism
- **EhCache** - Open-source caching library

### Search Layer
- **Solr** - Full-text search engine
- **Endeca** - Commerce-grade search and navigation

### Authentication
- **LDAP** - Lightweight Directory Access Protocol
- **OAuth** - Open authorization protocol

### Messaging & Notifications
- **Synchronous Messaging** - Inline request-response communication
- **Email Service** - Third-party email delivery

### Deployment Infrastructure
- **On-Premise VMs** - Traditional data center deployments
- **Cloud VMs** - AWS/Azure/GCP virtual machines
- **Load Balancer** - Distributes traffic across instances
- **Vertical Scaling** - Adding more resources (CPU/RAM) to servers

## Functional Layers

The monolithic application encapsulates all business capabilities:

1. **Product Catalog** - Product information, categories, attributes, inventory
2. **Search** - Product search and faceted navigation
3. **My Account** - Customer profiles, address books, order history, preferences
4. **Cart** - Shopping cart management and persistence
5. **Checkout** - Multi-step checkout flow
6. **Payments** - Payment processing and fraud detection
7. **Order Placement** - Order creation and confirmation
8. **Order Processing** - Order fulfillment workflows
9. **Email Notifications** - Customer communications (transactional emails)

## Architecture Characteristics

### Deployment Model
- **Monolithic Deployments** - Entire application deployed as single unit
- **Shared Database** - All modules share single Oracle database
- **Tight Coupling** - Functional layers are interdependent
- **Synchronous Communication** - All internal calls are blocking, request-response based

### Release & Operations
- **Shared Release Cycle** - All features/fixes released together
- **Centralized Logging** - Single logging infrastructure for all services
- **Vertical Scaling Only** - Scale by adding resources to existing servers

## Current Deployment Model Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    Development Environment                    │
│  Full Monolithic Stack (Hybris/ATG) → Local Oracle DB       │
└────────────────────────────────────────────────────┬──────────┘
                                                     │
                                                    Merge
                                                     │
┌────────────────────────────────────────────────────▼──────────┐
│                     QA Environment                             │
│  Full Monolithic Stack (Hybris/ATG) → QA Oracle DB            │
│  Testing: Functional, Integration, Performance                │
└────────────────────────────────────────────────────┬──────────┘
                                                     │
                                                   Deploy
                                                     │
┌────────────────────────────────────────────────────▼──────────┐
│                   Regression Environment                       │
│  Full Monolithic Stack (Hybris/ATG) → Staging Oracle DB       │
│  Testing: End-to-end, Third-party Integrations                │
└────────────────────────────────────────────────────┬──────────┘
                                                     │
                                                 Approval
                                                     │
┌────────────────────────────────────────────────────▼──────────┐
│                  Production Environment                        │
│  Full Monolithic Stack (Hybris/ATG) → Production Oracle DB    │
│  Load Balanced Instances, Centralized Logging                 │
└──────────────────────────────────────────────────────────────┘

Release Cycle Duration: 6-8 weeks
Deployment Risk: High (entire system deployed together)
Rollback Complexity: Very Complex (database state, third-party integrations)
```

## Release Plan & Deployment Process

```
DEV (Development)
  ↓ (code merged)
QA (Quality Assurance)
  ↓ (integration testing)
REGRESSION (User Acceptance Testing)
  ↓ (end-to-end validation)
PROD (Production Deployment)

Issues:
• Long release cycles (6-8 weeks)
• Entire application deployed together
• High risk of regressions
• Complex rollback procedures
• One team managing all changes
• Centralized release management
```

## Third-Party Integrations

All integrations use **synchronous communication** patterns:

1. **OMS (Order Management System)** - Order fulfillment, shipping, tracking
2. **Tax Service** - Tax rate calculation and compliance
3. **Email Service** - Customer notification delivery
4. **Payment Gateway** - Credit card processing, authorization
5. **PIM (Product Information Management)** - Master product data, enrichment

Integration challenges:
- Tight coupling with core application
- Synchronous calls create bottlenecks
- External service failures impact entire platform
- No independent scaling of integrations

## Current Challenges

### Technical Challenges
- **High Complexity** - Monolithic codebase difficult to understand and modify
- **Slow Deployment** - 6-8 week release cycles limit feature velocity
- **Poor Scalability** - Vertical scaling has hardware limits
- **Tight Coupling** - Changes to one feature may impact others unexpectedly
- **Shared Database** - Schema modifications affect entire system
- **Limited Flexibility** - All services use same tech stack and release cycle
- **Technology Debt** - Legacy Hybris/ATG framework, JSP pages, dated libraries

### Operational Challenges
- **Rollback Complexity** - Difficult to rollback specific features
- **Testing Burden** - Must test entire system for each change
- **Deployment Risk** - Higher chance of production incidents
- **Limited Monitoring** - Centralized logging makes debugging difficult
- **No Independent Scaling** - Cannot scale individual features
- **Team Dependency** - All teams must coordinate releases

## Examples of Technology Debt

1. **JSP Pages** - Mixing markup with business logic, hard to test
2. **Monolithic Hybris/ATG** - Vendor lock-in, difficult to replace components
3. **Synchronous Messaging** - Creates bottlenecks and cascading failures
4. **Single Shared Database** - Hard to implement independent data models
5. **Centralized Logging** - Difficult to trace requests across components
6. **Vertical Scaling Only** - Cannot leverage modern cloud-native patterns
7. **LDAP/OAuth Mixed** - Inconsistent authentication approach

## Business Impact

### Current Limitations
- **Feature Time-to-Market** - 6-8 weeks from development to production
- **Limited Innovation** - Complex processes slow down experimentation
- **Scalability Limits** - Vertical scaling has cost and physical limitations
- **Risk Aversion** - Teams hesitant to deploy due to regression risks
- **Customer Experience** - Limited ability to quickly respond to market demands
- **Operational Costs** - High infrastructure costs, large teams required

### Revenue Impact
- Slower feature rollout vs. competitors
- Cannot rapidly A/B test new features
- Lost sales opportunities due to slow innovation
- High operational costs reduce profit margins

## Reason for Change

### Strategic Drivers

1. **Enable Faster Innovation**
   - Reduce time-to-market from 6-8 weeks to days
   - Enable rapid experimentation and A/B testing
   - Respond quickly to competitive threats

2. **Improve Scalability**
   - Move from vertical to horizontal scaling
   - Leverage cloud-native infrastructure
   - Handle traffic spikes without overprovisioning

3. **Reduce Operational Risk**
   - Independent deployments reduce blast radius
   - Faster rollback for failed deployments
   - Improved monitoring and alerting

4. **Enable Team Autonomy**
   - Teams own specific features end-to-end
   - Independent release cycles
   - Faster decision-making

5. **Reduce Technical Debt**
   - Move away from monolithic Hybris/ATG
   - Modern tech stack (microservices, cloud-native)
   - Improved testability and maintainability

6. **Optimize Costs**
   - Horizontal scaling is more cost-effective
   - Better resource utilization
   - Pay-as-you-go cloud model
   - Reduce operational overhead
