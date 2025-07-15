# LARP Mailing System - Project Design Document

## Project Overview

A React web-based application that integrates with WordPress/WooCommerce to manage customer data, events, and email campaigns for LARP (Live Action Role Playing) events. The system provides intelligent email management with duplicate prevention, automated mailing lists, and comprehensive analytics.

## Technology Stack

### Frontend
- **React.js 18+** with TypeScript
- **Material-UI (MUI) v5** for UI components and theming
- **Redux Toolkit** for global state management
- **React Query (TanStack Query)** for server state and caching
- **React Router v6** for navigation
- **React-Quill** for rich text email editing

### Backend
- **Node.js** with Express.js
- **PostgreSQL** as primary database
- **Sequelize ORM** for database operations
- **JWT** for authentication
- **Node-cron** for scheduled tasks
- **Nodemailer** with pluggable email providers

### Infrastructure
- **Digital Ocean** for hosting
- **Gmail Workspace** (initial) → **SendGrid/Mailgun** (scalable)
- **Redis** for caching and session management

## Core System Requirements

### Data Synchronization
- **WooCommerce Integration**: Sync customers, orders, and products hourly
- **Data Transformation**: Convert WooCustomers to AppCustomers with email extraction from order metadata
- **Product-Event Association**: Manual admin interface with linguistic suggestion system
- **Incremental Sync**: Process only changed data with comprehensive logging

### Customer Management
- **AppCustomer Entity**: Individual customers with unique emails and event associations
- **Character Tracking**: Link customers to character names from LARP events
- **Event Participation**: Track which events each customer is attending
- **Source Tracking**: Maintain connection to original WooCommerce order data

### Event Management
- **Event Creation**: Generate events by grouping WooCommerce products
- **Product Association**: Link multiple products to single events
- **Attendee Management**: Automatic list generation of event participants
- **Analytics Dashboard**: Track event performance and customer engagement

### Mailing System
- **Template Management**: Rich text email templates with dynamic tag support
- **Mailing Lists**: Three types - Auto Event Attendees, Custom Event-Specific, Custom General
- **Duplicate Prevention**: Prevent sending same template to same customer for same event
- **Bulk Processing**: Queue-based system for large email campaigns
- **Delivery Tracking**: Monitor email status and engagement metrics

### User Management
- **Role-Based Access**: Admin and Normal User permission levels
- **Theme Customization**: Per-user interface preferences
- **Activity Logging**: Track user actions and system changes

## Database Architecture

### Core Entities
- **WooCommerce Sync Tables**: Raw data storage for customers, orders, products, order items
- **Application Tables**: AppCustomers, Events, Event-Product associations, Customer-Event participation
- **Mailing System**: Lists, templates, campaigns, delivery logs
- **System Management**: Users, permissions, sync logs, system settings

### Key Relationships
- WooCustomers → Multiple AppCustomers (via order item metadata)
- Products → Events (many-to-many with admin confirmation)
- AppCustomers → Events (participation tracking)
- Email Templates → Campaigns → Delivery Logs

## User Interface Design

### Navigation Structure
- **Dashboard**: System overview and key metrics
- **Customers**: List view and individual customer profiles
- **Events**: Event management and analytics
- **Email Campaigns**: Template management and campaign creation
- **Sync Management**: WordPress integration controls
- **Analytics**: Business intelligence dashboard
- **Settings**: User management and system configuration

### Design Principles
- **Material Design**: Clean, intuitive interface with customizable themes
- **Responsive Layout**: Mobile-friendly design with adaptive components
- **Data Visualization**: Charts and metrics for business intelligence
- **Efficient Workflows**: Streamlined processes for common tasks

## Development Phases

### Phase 1: Foundation & Setup (Weeks 1-2)
- Project infrastructure and authentication system
- Database schema implementation
- Basic UI layout and navigation

### Phase 2: WordPress Integration (Weeks 3-4)
- WooCommerce API connection and data sync
- Customer data transformation logic
- Product-event association interface

### Phase 3: Customer & Event Management (Weeks 5-6)
- Customer and event management interfaces
- Basic analytics and reporting
- Search and filtering capabilities

### Phase 4: Email System Core (Weeks 7-8)
- Email composer with rich text editing
- Template management system
- Mailing list creation and management

### Phase 5: Advanced Email Features (Weeks 9-10)
- Bulk email campaigns with queue processing
- Duplicate prevention and confirmation system
- Email tracking and analytics

### Phase 6: User Management & Permissions (Week 11)
- Multi-user support with role-based access
- User management interface
- Theme customization system

### Phase 7: Advanced Analytics & BI (Week 12)
- Comprehensive business intelligence dashboard
- Advanced reporting and data visualization
- Export capabilities for all data types

### Phase 8: Polish & Deployment (Week 13)
- UI/UX refinements and performance optimization
- Digital Ocean deployment setup
- Documentation and testing

## Technical Considerations

### Security
- JWT authentication with role-based permissions
- Input validation and SQL injection prevention
- Secure email content handling
- User activity auditing

### Performance
- Database indexing and query optimization
- Intelligent caching with React Query
- Pagination for large datasets
- Email queue processing for bulk operations

### Scalability
- Modular architecture for easy expansion
- Pluggable email provider system
- Horizontal scaling capability
- Microservice-ready design

### Monitoring
- Comprehensive logging system
- Email delivery tracking
- Sync operation monitoring
- Performance metrics collection

## Future Enhancements

### Post-MVP Features
- Advanced linguistic matching for product-event association
- Email automation workflows and triggers
- Customer segmentation and targeting tools
- Mobile application companion
- API for third-party integrations
- Multi-language support

### Long-term Vision
- Machine learning insights for customer behavior
- Advanced personalization engines
- Real-time notification system
- Integration with additional LARP management tools
- Community features for event participants

This design document serves as the master reference for the LARP Mailing System development project. All development decisions should align with the architecture and requirements outlined in this document.
