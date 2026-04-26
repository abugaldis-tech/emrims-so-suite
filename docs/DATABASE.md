# EMRims SO Suite - Database Schema

## Overview

EMRims SO Suite uses PostgreSQL 14+ with a normalized relational schema designed for:
- User authentication and authorization
- Lead and customer management
- Measurement tracking and EMR sizing
- Quote generation and tracking
- Activity audit trail

---

## Database Connection

### Connection Details
```
Host: localhost (or postgres in Docker)
Port: 5432
Database: emrims_db
User: emrims_user
Password: emrims_password_secure
```

### Connection String
```
postgresql://emrims_user:emrims_password_secure@localhost:5432/emrims_db
```

---

## Tables

### 1. users
Stores user account information and authentication details.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,  -- bcrypt hash
  email VARCHAR(255) UNIQUE NOT NULL,
  full_name VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL DEFAULT 'sales_officer',  -- admin, sales_officer, manager
  phone VARCHAR(20),
  is_active BOOLEAN DEFAULT true,
  last_login TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT role_check CHECK (role IN ('admin', 'sales_officer', 'manager'))
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `username`: Login username (must be unique)
- `password`: Bcrypt-hashed password
- `email`: User email (must be unique)
- `full_name`: User's full name
- `role`: User role (admin, sales_officer, manager)
- `phone`: Contact phone number
- `is_active`: Account status
- `last_login`: Timestamp of last login
- `created_at`: Account creation timestamp
- `updated_at`: Last update timestamp

**Default User:**
```
Username: admin
Password: Admin (hashed with bcrypt)
```

---

### 2. leads
Stores customer and prospect information.

```sql
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  phone VARCHAR(20) NOT NULL,
  company VARCHAR(255),
  position VARCHAR(255),
  status VARCHAR(50) DEFAULT 'new',  -- new, contacted, quoted, converted, lost
  source VARCHAR(100),  -- referral, website, social, event, cold_call
  notes TEXT,
  last_contacted TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT status_check CHECK (status IN ('new', 'contacted', 'quoted', 'converted', 'lost'))
);

CREATE INDEX idx_leads_user_id ON leads(user_id);
CREATE INDEX idx_leads_status ON leads(status);
CREATE INDEX idx_leads_email ON leads(email);
CREATE INDEX idx_leads_phone ON leads(phone);
CREATE INDEX idx_leads_created_at ON leads(created_at DESC);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `user_id`: FK to user who owns the lead
- `first_name`: Lead's first name
- `last_name`: Lead's last name
- `email`: Lead's email
- `phone`: Lead's phone (primary contact)
- `company`: Company/organization name
- `position`: Job position/title
- `status`: Lead status (new, contacted, quoted, converted, lost)
- `source`: How the lead was acquired
- `notes`: Custom notes about the lead
- `last_contacted`: When lead was last contacted
- `created_at`: When lead was created
- `updated_at`: When lead was last updated

---

### 3. measurements
Stores body measurements and EMR sizing information.

```sql
CREATE TABLE measurements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID NOT NULL REFERENCES leads(id) ON DELETE CASCADE,
  measurement_type VARCHAR(50) NOT NULL,  -- manual, emr_recommended, motion_capture
  data JSONB NOT NULL,  -- stores flexible measurement data
  recommended_size VARCHAR(10),  -- XS, S, M, L, XL, 2XL, 3XL, 4XL, 5XL
  fabric_tier VARCHAR(50),  -- Essential Shift, Regal Signature, Imperial Fit
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT measurement_type_check CHECK (measurement_type IN ('manual', 'emr_recommended', 'motion_capture')),
  CONSTRAINT size_check CHECK (recommended_size IN ('XS', 'S', 'M', 'L', 'XL', '2XL', '3XL', '4XL', '5XL'))
);

CREATE INDEX idx_measurements_lead_id ON measurements(lead_id);
CREATE INDEX idx_measurements_created_at ON measurements(created_at DESC);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `lead_id`: FK to the lead
- `measurement_type`: How measurement was taken
- `data`: JSONB containing flexible measurement data
  ```json
  {
    "chest": 38.5,
    "waist": 32.0,
    "length": 28.5,
    "sleeve_length": 24.0,
    "shoulder_width": 17.5,
    "unit": "inches"
  }
  ```
- `recommended_size`: EMR-recommended size (XS-5XL)
- `fabric_tier`: Which fabric tier (Essential, Regal, Imperial)
- `notes`: Additional measurement notes
- `created_at`: When measurement was recorded
- `updated_at`: When measurement was last updated

---

### 4. quotes
Stores sales quotations.

```sql
CREATE TABLE quotes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID NOT NULL REFERENCES leads(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  quote_number VARCHAR(50) UNIQUE NOT NULL,
  status VARCHAR(50) DEFAULT 'draft',  -- draft, sent, viewed, accepted, rejected, expired
  total_amount DECIMAL(12, 2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'NGN',
  tax_amount DECIMAL(12, 2),
  discount_amount DECIMAL(12, 2),
  notes TEXT,
  valid_until DATE,
  sent_at TIMESTAMP,
  viewed_at TIMESTAMP,
  accepted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT status_check CHECK (status IN ('draft', 'sent', 'viewed', 'accepted', 'rejected', 'expired'))
);

CREATE INDEX idx_quotes_lead_id ON quotes(lead_id);
CREATE INDEX idx_quotes_user_id ON quotes(user_id);
CREATE INDEX idx_quotes_status ON quotes(status);
CREATE INDEX idx_quotes_quote_number ON quotes(quote_number);
CREATE INDEX idx_quotes_created_at ON quotes(created_at DESC);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `lead_id`: FK to the lead
- `user_id`: FK to the sales officer who created the quote
- `quote_number`: Human-readable quote number (auto-generated)
- `status`: Quote status (draft, sent, viewed, accepted, rejected, expired)
- `total_amount`: Total quote amount (before tax)
- `currency`: Currency code (NGN, USD, etc.)
- `tax_amount`: Tax/VAT amount
- `discount_amount`: Applied discount
- `notes`: Quote notes/terms
- `valid_until`: Quote expiration date
- `sent_at`: When quote was sent to customer
- `viewed_at`: When customer viewed the quote
- `accepted_at`: When customer accepted the quote
- `created_at`: When quote was created
- `updated_at`: When quote was last updated

---

### 5. quote_items
Line items within a quote.

```sql
CREATE TABLE quote_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  quote_id UUID NOT NULL REFERENCES quotes(id) ON DELETE CASCADE,
  product_name VARCHAR(255) NOT NULL,
  product_category VARCHAR(100),  -- scrubs, lab_coat, accessories, etc.
  size VARCHAR(10),  -- XS, S, M, L, XL, 2XL, etc.
  fabric_tier VARCHAR(50),  -- Essential Shift, Regal Signature, Imperial Fit
  quantity INT NOT NULL,
  unit_price DECIMAL(12, 2) NOT NULL,
  total_price DECIMAL(12, 2) NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT quantity_check CHECK (quantity > 0),
  CONSTRAINT price_check CHECK (unit_price >= 0)
);

CREATE INDEX idx_quote_items_quote_id ON quote_items(quote_id);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `quote_id`: FK to the parent quote
- `product_name`: Name of the product
- `product_category`: Category (scrubs, lab_coat, etc.)
- `size`: Size (XS-5XL)
- `fabric_tier`: Which fabric tier
- `quantity`: Quantity ordered
- `unit_price`: Price per unit (in NGN)
- `total_price`: Quantity × Unit Price
- `description`: Product description
- `created_at`: When item was added

---

### 6. activities
Audit trail of all user actions.

```sql
CREATE TABLE activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID REFERENCES leads(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  activity_type VARCHAR(50) NOT NULL,  -- measurement, quote, call, email, message, note
  subject VARCHAR(255),
  description TEXT,
  metadata JSONB,  -- flexible data storage
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT activity_type_check CHECK (activity_type IN ('measurement', 'quote', 'call', 'email', 'message', 'note', 'update'))
);

CREATE INDEX idx_activities_lead_id ON activities(lead_id);
CREATE INDEX idx_activities_user_id ON activities(user_id);
CREATE INDEX idx_activities_activity_type ON activities(activity_type);
CREATE INDEX idx_activities_created_at ON activities(created_at DESC);
```

**Fields:**
- `id`: Unique identifier (UUID)
- `lead_id`: FK to the associated lead (optional)
- `user_id`: FK to the user who performed the action
- `activity_type`: Type of activity (measurement, quote, call, email, message, note, update)
- `subject`: Subject line (for emails/messages)
- `description`: Activity description
- `metadata`: JSONB for flexible data
- `created_at`: When activity occurred

**Example Metadata:**
```json
{
  "call_duration": "12 minutes",
  "email_subject": "Follow-up on your order",
  "message_platform": "whatsapp",
  "quote_id": "abc-123"
}
```

---

## Relationships Diagram

```
users (1) ──────────┐
                    ├──→ leads (1) ──┐
users (1) ──→ quotes      │          ├──→ measurements
users (1) ──→ activities  │          ├──→ activities
              ↓           │          └──→ quotes (1) ──→ quote_items
           leads (1) ─────┘
```

---

## Key Constraints & Indexes

### Foreign Keys
- `leads.user_id` → `users.id` (ON DELETE CASCADE)
- `measurements.lead_id` → `leads.id` (ON DELETE CASCADE)
- `quotes.lead_id` → `leads.id` (ON DELETE CASCADE)
- `quotes.user_id` → `users.id` (ON DELETE CASCADE)
- `quote_items.quote_id` → `quotes.id` (ON DELETE CASCADE)
- `activities.lead_id` → `leads.id` (ON DELETE CASCADE)
- `activities.user_id` → `users.id` (ON DELETE CASCADE)

### Indexes for Performance
- User lookups: `(username)`, `(email)`
- Lead searches: `(user_id)`, `(status)`, `(created_at DESC)`
- Quote searches: `(quote_number)`, `(status)`, `(created_at DESC)`
- Activity logs: `(lead_id)`, `(user_id)`, `(activity_type)`, `(created_at DESC)`

---

## Data Types

- **UUID**: For all primary keys (distributed systems friendly)
- **VARCHAR(n)**: For text fields with known max length
- **TEXT**: For longer text (notes, descriptions)
- **JSONB**: For flexible, schema-less data (measurements, metadata)
- **DECIMAL(12, 2)**: For monetary values (prevents rounding errors)
- **TIMESTAMP**: For all date/time values (includes time zone info)
- **DATE**: For dates without time
- **BOOLEAN**: For true/false flags
- **ENUM-like CONSTRAINT**: Check constraints for allowed values

---

## Query Examples

### Get all leads for a user with recent activity
```sql
SELECT l.*, 
       COUNT(a.id) as activity_count,
       MAX(a.created_at) as last_activity
FROM leads l
LEFT JOIN activities a ON l.id = a.lead_id
WHERE l.user_id = $1
GROUP BY l.id
ORDER BY l.created_at DESC;
```

### Get quotes for a lead with line items
```sql
SELECT q.*, 
       json_agg(json_build_object('product_name', qi.product_name, 'quantity', qi.quantity, 'unit_price', qi.unit_price)) as items
FROM quotes q
LEFT JOIN quote_items qi ON q.id = qi.quote_id
WHERE q.lead_id = $1
GROUP BY q.id
ORDER BY q.created_at DESC;
```

### Get lead activity timeline
```sql
SELECT user_id, activity_type, subject, description, created_at
FROM activities
WHERE lead_id = $1
ORDER BY created_at DESC
LIMIT 50;
```

---

## Migration Strategy

Migrations are run automatically on backend startup. New migrations are added as:
- `001_init.sql` - Initial schema
- `002_add_field.sql` - Add a column
- `003_modify_table.sql` - Modify existing structure

Each migration is idempotent and logs execution status.

---

## Backup & Recovery

### Backup (manual)
```bash
pg_dump -U emrims_user -d emrims_db > backup.sql
```

### Restore (manual)
```bash
psql -U emrims_user -d emrims_db < backup.sql
```

### Docker backup
```bash
docker exec emrims-postgres pg_dump -U emrims_user emrims_db > backup.sql
```
