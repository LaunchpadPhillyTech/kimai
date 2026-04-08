# Kimai Time Tracking System - Project Details

## Overview
Kimai is a professional, open-source time tracking application built with modern web technologies. It provides comprehensive time management, project tracking, billing, and reporting capabilities for individuals, teams, and organizations.

## System Architecture

### Technology Stack
- **Backend Framework**: PHP 8.1+ with Symfony Framework
- **Frontend**: JavaScript (ES6+) with Webpack Encore, Chart.js, FullCalendar
- **Database**: MariaDB/MySQL with Doctrine ORM
- **API**: RESTful JSON API with OpenAPI/Swagger documentation
- **Authentication**: Multi-provider (SAML, LDAP, Database, 2FA)
- **Containerization**: Docker with multi-stage builds
- **Build System**: Webpack Encore, Babel, Sass preprocessing

### Architectural Patterns
- **MVC Pattern**: Symfony's controller-service-entity architecture
- **Dependency Injection**: Symfony's DI container for service management
- **RESTful API**: Standardized endpoints following REST principles
- **Event-Driven**: Symfony's event dispatcher for extensibility
- **Repository Pattern**: Doctrine repositories for data access
- **Service Layer**: Business logic separated into dedicated services

### Core Components

#### Backend Structure
```
src/
├── API/                # REST API controllers
├── Entity/             # Doctrine ORM entities (data models)
├── Service/            # Business logic services
├── Repository/         # Data access layer
├── EventListener/      # Event handlers
├── Security/           # Authentication & authorization
├── Form/               # Form types and validation
└── Command/            # CLI commands
```

#### Frontend Structure
```
assets/
├── js/                 # JavaScript components
├── sass/               # Styling (SCSS)
└── [app].js           # Entry points (app, calendar, chart, etc.)

templates/
├── base.html.twig     # Base template
├── widget/            # Dashboard widgets
└── [module]/          # Feature-specific templates
```

## Business Logic

### Core Entities and Relationships

#### Primary Entities
- **Timesheet**: Central entity tracking time entries
- **User**: System users with role-based permissions
- **Project**: Work projects linked to customers
- **Activity**: Tasks/activities within projects
- **Customer**: Client organizations
- **Invoice**: Billing documents

#### Entity Relationships
```
Customer (1) ←→ (N) Project (1) ←→ (N) Activity
                    ↓                    ↓
User (N) ←→ (N) Timesheet ←→ (1) Activity
    ↓
Invoice (N) ←→ (1) User
```

### Key Business Processes

#### Time Tracking Workflow
1. **User Authentication**: Multi-provider login (SAML, LDAP, local)
2. **Project Selection**: Choose active project and activity
3. **Time Recording**: Start/stop timers or manual entry
4. **Validation**: Automatic duration calculation and validation
5. **Approval**: Optional timesheet approval workflows

#### Project Management
- **Hierarchical Structure**: Customers → Projects → Activities
- **Budget Management**: Time and monetary budgets per project/activity
- **Visibility Controls**: Project access based on user roles/teams
- **Rate Management**: Configurable billing rates per user/project

#### Reporting & Analytics
- **Statistical Services**: Activity, project, and user statistics
- **Export Capabilities**: CSV, PDF, XLSX, ODS, DOCX formats
- **Dashboard Widgets**: Customizable analytics displays
- **Advanced Filtering**: Multi-criteria search and filtering

#### Invoicing System
- **Rate Calculation**: Automated billing based on time entries
- **Multiple Formats**: Standard invoicing with ZUGFeRD support
- **Status Tracking**: Invoice lifecycle management (pending, paid, canceled)
- **Customer Integration**: Linked billing to customer records

### User Roles and Permissions
- **ROLE_USER**: Basic time tracking capabilities
- **ROLE_TEAMLEAD**: Team management and reporting
- **ROLE_ADMIN**: System administration and configuration
- **ROLE_SUPER_ADMIN**: Full system access and user management

## Current AI Usage Analysis

### AI Implementation Status: **NONE FOUND**

After thorough code analysis, Kimai currently does not implement any AI or machine learning features:

- **No AI Dependencies**: No AI/ML libraries in composer.json or package.json
- **No Intelligent Features**: No predictive suggestions, auto-categorization, or smart recommendations
- **No Analytics AI**: Reporting is based on traditional statistical aggregation
- **No Automation AI**: Existing automation is rule-based, not AI-driven

### Current Automation Features (Non-AI)
- **Automatic Duration Calculation**: Time difference computation
- **Rule-Based Rounding**: Configurable time rounding rules
- **Scheduled Exports**: Automated report generation
- **Budget Alerts**: Threshold-based notifications
- **Rate Application**: Automatic billing rate calculations

## Data Flow Architecture

### Time Entry Process
```
User Interface → Controller → Service Layer → Repository → Database
                             ↓
                         Validation & Business Rules
                             ↓
                         Event Dispatching → Listeners
```

### API Request Flow
```
API Client → Authentication → Controller → Service → Entity → Repository → Database
                                ↓
                           JSON Response ← Serialization
```

## Security Architecture

### Authentication Methods
- **Local Database**: Encrypted password storage
- **LDAP Integration**: Active Directory support
- **SAML SSO**: Enterprise single sign-on
- **Two-Factor Authentication**: TOTP implementation

### Authorization
- **Role-Based Access Control (RBAC)**: Hierarchical permissions
- **Project-Level Security**: Granular access controls
- **API Security**: Token-based authentication
- **CSRF Protection**: Cross-site request forgery prevention

## Configuration Management

### Key Configuration Files
- **security.yaml**: Authentication and authorization rules
- **doctrine.yaml**: Database connection and ORM settings
- **kimai.yaml**: Business rules and time-tracking configuration
- **routes.yaml**: API and web route definitions

### Environment Configuration
- **Docker Support**: Multi-stage containerization
- **Environment Variables**: Configurable deployment settings
- **Development Tools**: Debug toolbar, profiler integration

## Integration Capabilities

### API Features
- **OpenAPI Documentation**: Swagger UI at `/api/doc`
- **RESTful endpoints**: Standard CRUD operations
- **Authentication**: Token-based API access
- **Rate Limiting**: Configurable API throttling

### Export Integrations
- **Multiple Formats**: CSV, Excel, PDF, OpenDocument
- **Electronic Invoicing**: ZUGFeRD standard support
- **Webhook Support**: Event-driven integrations

## Performance Characteristics

### Scalability Features
- **Database Optimization**: Indexed queries and efficient relationships
- **Caching**: Symfony cache layers
- **Asset Optimization**: Webpack bundling and minification
- **Docker Deployment**: Scalable container architecture

### Monitoring
- **Symfony Profiler**: Development performance insights
- **Database Query Analysis**: Doctrine query optimization
- **Error Tracking**: Comprehensive logging system

## Development and Deployment

### Development Workflow
- **Symfony CLI**: Local development server
- **Database Migrations**: Version-controlled schema changes
- **Asset Building**: Webpack Encore for frontend builds
- **Testing**: PHPUnit integration for backend testing

### Deployment Options
- **Docker Containers**: Production-ready images
- **Traditional Hosting**: Standard PHP hosting support
- **Cloud Deployment**: AWS, Azure, GCP compatible

---

*This analysis represents the current state of the Kimai time tracking system as of the codebase review. The system shows strong foundational architecture suitable for AI enhancement opportunities.*