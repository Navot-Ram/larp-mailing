# LARP Mailing System - Technical Reference

## Database Schema

### WooCommerce Sync Tables
```sql
-- Raw WooCommerce data storage
woo_customers (
  id SERIAL PRIMARY KEY,
  woo_customer_id INTEGER UNIQUE,
  email VARCHAR(255),
  name VARCHAR(255),
  sync_date TIMESTAMP,
  raw_data JSONB
);

woo_orders (
  id SERIAL PRIMARY KEY,
  woo_order_id INTEGER UNIQUE,
  woo_customer_id INTEGER REFERENCES woo_customers(woo_customer_id),
  order_date TIMESTAMP,
  total DECIMAL(10,2),
  status VARCHAR(50),
  raw_data JSONB
);

woo_products (
  id SERIAL PRIMARY KEY,
  woo_product_id INTEGER UNIQUE,
  name VARCHAR(255),
  description TEXT,
  price DECIMAL(10,2),
  event_id INTEGER REFERENCES events(id),
  auto_assigned BOOLEAN DEFAULT FALSE,
  admin_confirmed BOOLEAN DEFAULT FALSE,
  raw_data JSONB
);

woo_order_items (
  id SERIAL PRIMARY KEY,
  woo_order_id INTEGER REFERENCES woo_orders(woo_order_id),
  woo_product_id INTEGER REFERENCES woo_products(woo_product_id),
  quantity INTEGER,
  meta_data JSONB -- Contains participant emails, character names, etc.
);
```

### Application Core Tables
```sql
-- App-specific customer entities
app_customers (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE,
  name VARCHAR(255),
  character_name VARCHAR(255),
  source_woo_order_item_id INTEGER REFERENCES woo_order_items(id),
  created_from_meta JSONB,
  created_date TIMESTAMP DEFAULT NOW(),
  status VARCHAR(50) DEFAULT 'active'
);

-- Events management
events (
  id SERIAL PRIMARY KEY,
  event_uid VARCHAR(100) UNIQUE,
  name VARCHAR(255),
  description TEXT,
  start_date TIMESTAMP,
  end_date TIMESTAMP,
  status VARCHAR(50) DEFAULT 'active',
  created_date TIMESTAMP DEFAULT NOW()
);

-- Product-Event associations
event_products (
  id SERIAL PRIMARY KEY,
  event_id INTEGER REFERENCES events(id),
  woo_product_id INTEGER REFERENCES woo_products(woo_product_id),
  auto_assigned BOOLEAN DEFAULT FALSE,
  admin_confirmed BOOLEAN DEFAULT FALSE,
  similarity_score DECIMAL(3,2), -- For linguistic matching
  assigned_date TIMESTAMP DEFAULT NOW()
);

-- Customer-Event participation
customer_events (
  id SERIAL PRIMARY KEY,
  app_customer_id INTEGER REFERENCES app_customers(id),
  event_id INTEGER REFERENCES events(id),
  participation_status VARCHAR(50) DEFAULT 'registered',
  registration_date TIMESTAMP DEFAULT NOW(),
  UNIQUE(app_customer_id, event_id)
);
```

### Mailing System Tables
```sql
-- Mailing lists
mailing_lists (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  description TEXT,
  type VARCHAR(50), -- 'auto_event_attendees', 'custom_event_specific', 'custom_general'
  event_id INTEGER REFERENCES events(id), -- NULL for general lists
  is_automated BOOLEAN DEFAULT FALSE,
  created_by INTEGER REFERENCES users(id),
  created_date TIMESTAMP DEFAULT NOW()
);

-- List membership
list_members (
  id SERIAL PRIMARY KEY,
  list_id INTEGER REFERENCES mailing_lists(id),
  app_customer_id INTEGER REFERENCES app_customers(id),
  added_date TIMESTAMP DEFAULT NOW(),
  added_by INTEGER REFERENCES users(id),
  UNIQUE(list_id, app_customer_id)
);

-- Email templates
email_templates (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  subject VARCHAR(255),
  content TEXT, -- Rich HTML content
  template_type VARCHAR(50), -- 'event_specific', 'general'
  available_tags JSONB, -- Dynamic tags available for this template
  created_by INTEGER REFERENCES users(id),
  created_date TIMESTAMP DEFAULT NOW(),
  last_modified TIMESTAMP DEFAULT NOW()
);

-- Email campaigns
email_campaigns (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  template_id INTEGER REFERENCES email_templates(id),
  list_id INTEGER REFERENCES mailing_lists(id),
  event_id INTEGER REFERENCES events(id), -- NULL for general campaigns
  scheduled_date TIMESTAMP,
  sent_date TIMESTAMP,
  status VARCHAR(50) DEFAULT 'draft', -- 'draft', 'scheduled', 'sending', 'sent', 'failed'
  total_recipients INTEGER,
  sent_count INTEGER DEFAULT 0,
  created_by INTEGER REFERENCES users(id)
);

-- Individual email delivery tracking
email_delivery_log (
  id SERIAL PRIMARY KEY,
  campaign_id INTEGER REFERENCES email_campaigns(id),
  app_customer_id INTEGER REFERENCES app_customers(id),
  email_address VARCHAR(255),
  sent_date TIMESTAMP,
  delivery_status VARCHAR(50), -- 'sent', 'delivered', 'opened', 'clicked', 'bounced', 'failed'
  requires_confirmation BOOLEAN DEFAULT FALSE, -- For duplicate prevention
  confirmed_by INTEGER REFERENCES users(id),
  confirmation_date TIMESTAMP,
  error_message TEXT
);
```

### System Management Tables
```sql
-- User management
users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100) UNIQUE,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  role VARCHAR(50) DEFAULT 'user', -- 'admin', 'user'
  permissions JSONB,
  theme_preferences JSONB,
  last_login TIMESTAMP,
  created_date TIMESTAMP DEFAULT NOW(),
  status VARCHAR(50) DEFAULT 'active'
);

-- Sync operation logs
sync_logs (
  id SERIAL PRIMARY KEY,
  sync_type VARCHAR(50), -- 'customers', 'orders', 'products', 'full'
  status VARCHAR(50), -- 'running', 'completed', 'failed'
  records_processed INTEGER,
  records_created INTEGER,
  records_updated INTEGER,
  errors JSONB,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  triggered_by INTEGER REFERENCES users(id) -- NULL for automated sync
);
```

## React Component Architecture

### Application Structure
```
src/
├── components/
│   ├── common/              # Reusable UI components
│   │   ├── SmartTable.jsx
│   │   ├── MetricCard.jsx
│   │   ├── LoadingSpinner.jsx
│   │   ├── ConfirmDialog.jsx
│   │   └── ExportButton.jsx
│   ├── layout/              # Layout components
│   │   ├── AppLayout.jsx
│   │   ├── Sidebar.jsx
│   │   ├── Header.jsx
│   │   └── PageContainer.jsx
│   ├── auth/                # Authentication
│   │   ├── LoginForm.jsx
│   │   ├── UserMenu.jsx
│   │   └── ProtectedRoute.jsx
│   ├── customers/           # Customer management
│   │   ├── CustomerListPage.jsx
│   │   ├── CustomerDetailPage.jsx
│   │   ├── CustomerCard.jsx
│   │   ├── CustomerFilters.jsx
│   │   └── CustomerEditForm.jsx
│   ├── events/              # Event management
│   │   ├── EventListPage.jsx
│   │   ├── EventDetailPage.jsx
│   │   ├── EventCard.jsx
│   │   ├── EventCreator.jsx
│   │   ├── ProductAssignment.jsx
│   │   └── EventAnalytics.jsx
│   ├── mailing/             # Email system
│   │   ├── EmailComposer.jsx
│   │   ├── TemplateManager.jsx
│   │   ├── CampaignDashboard.jsx
│   │   ├── ListManager.jsx
│   │   ├── DuplicateWarning.jsx
│   │   └── BulkEmailSender.jsx
│   ├── sync/                # WordPress sync
│   │   ├── SyncDashboard.jsx
│   │   ├── SyncHistory.jsx
│   │   ├── ProductEventMapper.jsx
│   │   └── DataTransformationTool.jsx
│   └── admin/               # Admin tools
│       ├── UserManagement.jsx
│       ├── SystemSettings.jsx
│       ├── ThemeCustomizer.jsx
│       └── PermissionManager.jsx
├── hooks/                   # Custom React hooks
│   ├── useAuth.js
│   ├── useCustomers.js
│   ├── useEvents.js
│   ├── useEmailCampaigns.js
│   ├── useSync.js
│   └── useTheme.js
├── services/                # API services
│   ├── api.js
│   ├── authService.js
│   ├── customerService.js
│   ├── eventService.js
│   ├── emailService.js
│   └── syncService.js
├── store/                   # Redux store
│   ├── index.js
│   ├── authSlice.js
│   ├── customerSlice.js
│   ├── eventSlice.js
│   └── uiSlice.js
├── utils/                   # Utility functions
│   ├── dateHelpers.js
│   ├── emailValidation.js
│   ├── exportHelpers.js
│   └── formatters.js
└── types/                   # TypeScript definitions
    ├── customer.types.ts
    ├── event.types.ts
    ├── email.types.ts
    └── api.types.ts
```

## Backend Services Architecture

```
server/
├── routes/
│   ├── auth.js         # Authentication & authorization
│   ├── customers.js    # Customer CRUD operations
│   ├── events.js       # Event management
│   ├── emails.js       # Email operations
│   ├── sync.js         # WordPress synchronization
│   └── analytics.js    # Reporting & analytics
├── services/
│   ├── wordpressSync.js # WooCommerce API integration
│   ├── emailService.js  # Pluggable email providers
│   ├── templateEngine.js # Dynamic content processing
│   └── scheduler.js     # Cron jobs for sync
├── models/             # Database models (Sequelize ORM)
├── middleware/         # Auth, validation, logging
└── utils/             # Helper functions
```

## Data Transformation Logic

### WooCommerce → App Customer Transformation
```javascript
// Example transformation process
const transformWooCustomerData = (wooOrder) => {
  const appCustomers = [];
  
  wooOrder.line_items.forEach(item => {
    const participantEmails = item.meta_data.find(
      meta => meta.key === 'participant_emails'
    )?.value || [];
    
    const characterNames = item.meta_data.find(
      meta => meta.key === 'character_names'
    )?.value || [];
    
    participantEmails.forEach((email, index) => {
      appCustomers.push({
        email,
        name: extractNameFromEmail(email),
        characterName: characterNames[index] || null,
        sourceWooOrderItemId: item.id,
        eventId: findEventByProductId(item.product_id),
        createdFromMeta: {
          originalOrder: wooOrder.id,
          productId: item.product_id,
          productName: item.name
        }
      });
    });
  });
  
  return appCustomers;
};
```

### Email Duplicate Prevention Logic
```javascript
// services/emailService.js
const checkForDuplicates = async (templateId, eventId, customerIds) => {
  const duplicates = await EmailDeliveryLog.findAll({
    where: {
      template_id: templateId,
      event_id: eventId,
      app_customer_id: { [Op.in]: customerIds },
      delivery_status: { [Op.in]: ['sent', 'delivered'] }
    },
    include: [{ model: AppCustomer, attributes: ['name', 'email'] }]
  });
  
  return {
    hasDuplicates: duplicates.length > 0,
    duplicateCustomers: duplicates.map(d => d.AppCustomer),
    requiresConfirmation: duplicates.length > 0
  };
};
```

## Email Service Integration

### Pluggable Email Provider Architecture
```javascript
// services/emailProviders/
├── gmailProvider.js
├── sendgridProvider.js
├── mailgunProvider.js
└── baseProvider.js

// Abstract base class
class BaseEmailProvider {
  async sendEmail(to, subject, content) {
    throw new Error('sendEmail must be implemented');
  }
  
  async sendBulkEmail(recipients, subject, content) {
    throw new Error('sendBulkEmail must be implemented');
  }
  
  async trackDelivery(messageId) {
    throw new Error('trackDelivery must be implemented');
  }
}
```

## UI/UX Design Patterns

### Material Design Implementation
- **Color Scheme**: Customizable primary/secondary colors via admin settings
- **Typography**: Roboto font family with consistent heading hierarchy
- **Spacing**: 8px grid system for consistent layouts
- **Components**: Cards for data display, DataGrid for tables, Chips for status indicators

### Navigation Structure
```
Dashboard (/)
├── Customers (/customers)
│   ├── Customer List
│   └── Customer Detail (/customers/:id)
├── Events (/events)
│   ├── Event List
│   └── Event Detail (/events/:id)
├── Email Campaigns (/emails)
│   ├── Campaign List
│   ├── Template Manager
│   └── Compose Email
├── Sync Management (/sync)
│   ├── Sync Dashboard
│   └── Product-Event Mapping
├── Analytics (/analytics)
└── Settings (/settings)
    ├── User Management (Admin only)
    ├── Theme Customization
    └── System Configuration
```

### Responsive Design Breakpoints
- **Mobile**: < 600px (Stack components vertically)
- **Tablet**: 600px - 960px (Condensed sidebar, adjusted grid)
- **Desktop**: > 960px (Full sidebar, multi-column layouts)

## Email Template System

### Dynamic Tag System
```javascript
// Event-specific tags (only available for event-attached lists)
{{event.name}}, {{event.date}}, {{customer.character_name}}

// General tags (available for all templates)
{{customer.name}}, {{customer.email}}, {{system.date}}
```

### Template Features
- Rich HTML templates with CSS inlining for email compatibility
- Dynamic tag replacement system
- Template versioning and rollback
- A/B testing capabilities
- Mobile-responsive email templates

## API Endpoints Reference

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `POST /api/auth/refresh` - Refresh JWT token

### Customers
- `GET /api/customers` - List customers with filtering
- `GET /api/customers/:id` - Get customer details
- `PUT /api/customers/:id` - Update customer
- `DELETE /api/customers/:id` - Delete customer

### Events
- `GET /api/events` - List events
- `POST /api/events` - Create event
- `GET /api/events/:id` - Get event details
- `PUT /api/events/:id` - Update event
- `DELETE /api/events/:id` - Delete event

### Email Campaigns
- `GET /api/emails/templates` - List email templates
- `POST /api/emails/templates` - Create template
- `GET /api/emails/campaigns` - List campaigns
- `POST /api/emails/campaigns` - Create campaign
- `POST /api/emails/send` - Send email campaign

### Sync Operations
- `POST /api/sync/manual` - Trigger manual sync
- `GET /api/sync/status` - Get sync status
- `GET /api/sync/logs` - Get sync history

This technical reference document provides detailed implementation guidance for developers working on the LARP Mailing System.
