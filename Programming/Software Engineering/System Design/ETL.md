---
tags:
  - software-engineering
  - system-design
  - etl
  - data-engineering
created: 2026-01-02
status: 🔴
---
# 🔄 ETL (Extract, Transform, Load)

> *"Moving data from where it is to where it needs to be, in the format it needs to be in."*

## 🎯 What is ETL?

ETL es un proceso de integración de datos que consiste en:
- **Extract**: Obtener datos de múltiples fuentes
- **Transform**: Limpiar, validar y convertir los datos
- **Load**: Cargar los datos en el destino final

---

## 🏗️ ETL Pipeline Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        ETL PIPELINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   EXTRACT               TRANSFORM                LOAD           │
│   ┌─────────────┐      ┌───────────────┐      ┌───────────┐    │
│   │ Database    │──┐   │               │      │           │    │
│   └─────────────┘  │   │ • Clean       │      │  Data     │    │
│   ┌─────────────┐  │   │ • Validate    │      │  Warehouse│    │
│   │ API         │──┼──►│ • Transform   │─────►│           │    │
│   └─────────────┘  │   │ • Aggregate   │      │  (OLAP)   │    │
│   ┌─────────────┐  │   │ • Enrich      │      │           │    │
│   │ Files       │──┘   │               │      │           │    │
│   └─────────────┘      └───────────────┘      └───────────┘    │
│                                                                 │
│   Sources               Processing            Destination       │
│   (OLTP systems)                             (Analytics)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📤 Extract Phase

### Data Sources
```typescript
// Database extraction
async function extractFromDatabase() {
  const connection = await mysql.createConnection({
    host: 'source-db.example.com',
    database: 'production'
  });

  // Full extraction
  const [rows] = await connection.query('SELECT * FROM orders');

  // Incremental extraction (only new/changed records)
  const [newRows] = await connection.query(`
    SELECT * FROM orders 
    WHERE updated_at > ? 
    ORDER BY updated_at
  `, [lastExtractTime]);

  return rows;
}

// API extraction
async function extractFromAPI() {
  let allData = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`https://api.example.com/data?page=${page}`);
    const data = await response.json();
    
    allData = [...allData, ...data.items];
    hasMore = data.hasMore;
    page++;
  }

  return allData;
}

// File extraction (CSV, JSON, etc.)
async function extractFromFiles() {
  const files = await glob('data/*.csv');
  
  const allData = await Promise.all(files.map(async (file) => {
    const content = await fs.readFile(file, 'utf-8');
    return parseCSV(content);
  }));

  return allData.flat();
}
```

### Change Data Capture (CDC)
```
┌─────────────────────────────────────────────────────────────────┐
│               CHANGE DATA CAPTURE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Source DB              CDC Tool              Target           │
│   ┌─────────┐          ┌─────────┐          ┌─────────┐        │
│   │ Orders  │          │ Debezium│          │  Kafka  │        │
│   │ Table   │──────────│ (CDC)   │─────────►│  Topic  │        │
│   └─────────┘          └─────────┘          └─────────┘        │
│       │                                          │              │
│       │   Transaction Log                        │              │
│       └── Reading binlog/WAL                     │              │
│                                                  │              │
│   Events: INSERT, UPDATE, DELETE ◄───────────────┘              │
│   captured in real-time                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Transform Phase

### Common Transformations
```typescript
// Data cleaning
function cleanData(records: RawRecord[]): CleanedRecord[] {
  return records
    // Remove invalid records
    .filter(record => isValid(record))
    // Normalize values
    .map(record => ({
      ...record,
      email: record.email?.toLowerCase().trim(),
      phone: normalizePhoneNumber(record.phone),
      name: capitalizeWords(record.name),
      // Handle nulls
      country: record.country || 'Unknown',
    }))
    // Remove duplicates
    .filter((record, index, self) => 
      index === self.findIndex(r => r.id === record.id)
    );
}

// Data validation
function validateRecord(record: any): ValidationResult {
  const errors: string[] = [];

  if (!record.email || !isValidEmail(record.email)) {
    errors.push('Invalid email');
  }
  if (!record.amount || record.amount < 0) {
    errors.push('Invalid amount');
  }
  if (!record.date || !isValidDate(record.date)) {
    errors.push('Invalid date');
  }

  return {
    isValid: errors.length === 0,
    errors,
    record
  };
}

// Data enrichment
async function enrichData(records: Order[]): Promise<EnrichedOrder[]> {
  return Promise.all(records.map(async (order) => {
    // Fetch additional data
    const customer = await getCustomer(order.customerId);
    const product = await getProduct(order.productId);
    
    return {
      ...order,
      customerName: customer.name,
      customerSegment: customer.segment,
      productCategory: product.category,
      productBrand: product.brand,
      // Derived fields
      orderYear: new Date(order.date).getFullYear(),
      orderMonth: new Date(order.date).getMonth() + 1,
      isHighValue: order.amount > 1000,
    };
  }));
}

// Aggregation
function aggregateByDay(records: Order[]): DailyAggregate[] {
  const grouped = records.reduce((acc, order) => {
    const date = order.date.toISOString().split('T')[0];
    
    if (!acc[date]) {
      acc[date] = { date, count: 0, total: 0, orders: [] };
    }
    
    acc[date].count++;
    acc[date].total += order.amount;
    acc[date].orders.push(order.id);
    
    return acc;
  }, {} as Record<string, DailyAggregate>);

  return Object.values(grouped);
}

// Type conversion
function transformTypes(record: any): TransformedRecord {
  return {
    id: String(record.id),
    amount: parseFloat(record.amount),
    quantity: parseInt(record.quantity, 10),
    date: new Date(record.date),
    isActive: record.is_active === 'true' || record.is_active === 1,
    metadata: JSON.parse(record.metadata || '{}'),
  };
}
```

### Data Quality Rules
```typescript
interface DataQualityRule {
  name: string;
  check: (record: any) => boolean;
  severity: 'error' | 'warning';
}

const qualityRules: DataQualityRule[] = [
  {
    name: 'email_not_null',
    check: (r) => r.email != null,
    severity: 'error'
  },
  {
    name: 'amount_positive',
    check: (r) => r.amount > 0,
    severity: 'error'
  },
  {
    name: 'date_not_future',
    check: (r) => new Date(r.date) <= new Date(),
    severity: 'warning'
  },
];

function applyQualityRules(records: any[]): QualityReport {
  const issues: QualityIssue[] = [];

  for (const record of records) {
    for (const rule of qualityRules) {
      if (!rule.check(record)) {
        issues.push({
          recordId: record.id,
          rule: rule.name,
          severity: rule.severity
        });
      }
    }
  }

  return {
    totalRecords: records.length,
    passedRecords: records.filter(r => 
      qualityRules.every(rule => rule.check(r))
    ).length,
    issues
  };
}
```

---

## 📥 Load Phase

### Loading Strategies

```typescript
// Full load (replace all data)
async function fullLoad(data: any[], tableName: string) {
  await db.query(`TRUNCATE TABLE ${tableName}`);
  await bulkInsert(tableName, data);
}

// Incremental load (append new data)
async function incrementalLoad(data: any[], tableName: string) {
  await bulkInsert(tableName, data);
}

// Upsert (insert or update)
async function upsertLoad(data: any[], tableName: string) {
  for (const record of data) {
    await db.query(`
      INSERT INTO ${tableName} (id, name, value, updated_at)
      VALUES (?, ?, ?, NOW())
      ON DUPLICATE KEY UPDATE
        name = VALUES(name),
        value = VALUES(value),
        updated_at = NOW()
    `, [record.id, record.name, record.value]);
  }
}

// SCD Type 2 (Slowly Changing Dimensions)
async function scdType2Load(data: any[], tableName: string) {
  for (const record of data) {
    // Close current record
    await db.query(`
      UPDATE ${tableName}
      SET end_date = NOW(), is_current = false
      WHERE id = ? AND is_current = true
    `, [record.id]);

    // Insert new record
    await db.query(`
      INSERT INTO ${tableName} 
        (id, name, value, start_date, end_date, is_current)
      VALUES 
        (?, ?, ?, NOW(), NULL, true)
    `, [record.id, record.name, record.value]);
  }
}
```

### Bulk Loading
```sql
-- PostgreSQL COPY command (fastest)
COPY orders FROM '/data/orders.csv' 
WITH (FORMAT csv, HEADER true);

-- MySQL LOAD DATA
LOAD DATA INFILE '/data/orders.csv'
INTO TABLE orders
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

-- Using staging tables
CREATE TEMP TABLE staging_orders (LIKE orders);

COPY staging_orders FROM '/data/orders.csv';

INSERT INTO orders 
SELECT * FROM staging_orders
ON CONFLICT (id) DO UPDATE SET ...;

DROP TABLE staging_orders;
```

---

## 🔀 ETL vs ELT

```
┌─────────────────────────────────────────────────────────────────┐
│                    ETL vs ELT                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ETL (Traditional)                                             │
│   ┌────────┐    ┌───────────┐    ┌────────┐                    │
│   │Extract │───►│ Transform │───►│  Load  │                    │
│   │        │    │(ETL Server)│   │        │                    │
│   └────────┘    └───────────┘    └────────┘                    │
│                                                                 │
│   • Transform before loading                                    │
│   • Good for complex transformations                            │
│   • Requires separate compute resources                         │
│                                                                 │
│   ELT (Modern)                                                  │
│   ┌────────┐    ┌────────┐    ┌───────────┐                    │
│   │Extract │───►│  Load  │───►│ Transform │                    │
│   │        │    │        │    │(in DW)    │                    │
│   └────────┘    └────────┘    └───────────┘                    │
│                                                                 │
│   • Load raw data first                                         │
│   • Transform using warehouse compute                           │
│   • Good for cloud data warehouses                              │
│   • BigQuery, Snowflake, Redshift                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ ETL Tools Comparison

| Tool | Type | Best For |
|------|------|----------|
| **Apache Airflow** | Orchestration | Complex workflows |
| **dbt** | Transform | SQL transformations |
| **Apache Spark** | Processing | Big data |
| **Fivetran** | EL | Quick setup, SaaS |
| **Airbyte** | EL | Open source, connectors |
| **AWS Glue** | ETL | AWS ecosystem |
| **Talend** | ETL | Enterprise |

---

## 📋 ETL Best Practices

> [!check] Design
> - [ ] Document data lineage
> - [ ] Implement idempotent operations
> - [ ] Use incremental loads when possible
> - [ ] Design for failure recovery (checkpoints)

> [!check] Quality
> - [ ] Validate data at each stage
> - [ ] Log rejected records
> - [ ] Monitor data quality metrics
> - [ ] Alert on anomalies

> [!check] Performance
> - [ ] Parallelize where possible
> - [ ] Use bulk operations
> - [ ] Optimize transformation logic
> - [ ] Schedule during low-traffic periods

---

## 📚 Related

- [[Programming/Software Engineering/Database Design/_Index|Database Design]]
- [[Programming/Software Engineering/System Design/Scalability|Scalability]]

---

← [[Programming/Software Engineering/System Design/_Index|Back to System Design]]
