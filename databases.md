
---
title: Databases and Data Structures
---

<div class="nav-bar">

[<a href="/CS499/">Home</a>] | 
[<a href="/CS499/code-review">Code Review</a>] | 
[<a href="/CS499/software-design">Software Design and Engineering</a>] | 
[<a href="/CS499/databases">Databases</a>] |
[<a href="/CS499/algorithms">Algorithms and Data Structures</a>]  | 


</div>

---

## Overview ##


The application that I chose to enhance for the next part of the capstone is my CS360 classes Android Application that is a Inventory Tracker app that would work on a phone. This particular applicaiton uses a simple database with only 2 tables and no relationships.

---

### What I reviewed

*1 AppDatabase.java*
    - The entire database configuration happens here. 
```java
@Database(
        entities = { InventoryItem.class, User.class },
        version = 1,
        exportSchema = false
)
```

    - Has 2 entity classes
      - InventoryItem and User
    - Has one database schema
    - Disallows schema export

```java
@Database(
        entities = { InventoryItem.class, User.class },
        version = 1,
        exportSchema = false
)
```
    - Room generates the implementation of these methods automatically 
    - DAO or Data Access Objects define how the database is queried
  - getInstance
    - Creates the actual database

  *Database Schema*
```
DATABASE: inventory.db
VERSION: 1
TABLES: 2
RELATIONSHIPS: 0 (ZERO!)

┌──────────────────────┐
│     inventory        │
├──────────────────────┤
│ id (PK)              │
│ name                 │
│ description          │
│ quantity             │
└──────────────────────┘

┌──────────────────────┐
│       users          │
├──────────────────────┤
│ id (PK)              │
│ username             │
│ passwordHash         │
└──────────────────────┘
```

*InventoryItem Entity*

```java
@Entity(tableName = "inventory")
```
- Creates a table named 'Inventory' based on the class

Fields are as follows:

```java
@PrimaryKey(autoGenerate = true)
private long id;

private final String name;
private final String description;
private int quantity;
```

SQL generates this SQL:

```sql
CREATE TABLE inventory (
    id          INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    name        TEXT,
    description TEXT,
    quantity    INTEGER NOT NULL
);
```
*User.java*

```java
@Entity(tableName = "users")
public class User {
    @PrimaryKey(autoGenerate = true)
    public long id;

    @NonNull
    public String username;

    @NonNull
    public String passwordHash;
```

  - Stores passwordHash not the actual password
  
*DAO Layer*

- Defines how to query the database

```java
@Dao
public interface InventoryDao {
    @Query("SELECT * FROM inventory ORDER BY id DESC")
    List<InventoryItem> getAll();

    @Query("SELECT * FROM inventory WHERE id = :id LIMIT 1")
    InventoryItem getById(Long id);

    @Insert
    long insert(InventoryItem item);

    @Update
    int update(InventoryItem item);

    @Delete
    int delete(InventoryItem item);
}
```

This DAO provides basic CRUD operations:
- **CREATE**: insert() method
- **READ**: getAll() and getById() methods
- **UPDATE**: update() method
- **DELETE**: delete() method

---

### Planned Enhancements

*Enhancement 1: Normalized Database Schema*

  - Redesigning the database to follow the Third Normal Form with propoer relationships

New Schema as follows:

```
DATABASE: inventory.db
VERSION: 2 (migrated from version 1)
TABLES: 6 (was 2)
RELATIONSHIPS: 7 foreign keys

┌─────────────────────┐
│     categories      │
├─────────────────────┤
│ id (PK)             │
│ name                │
│ description         │
│ created_at          │
└─────────────────────┘
         ↓ 1:N
┌──────────────────────────────┐
│       inventory              │
├──────────────────────────────┤
│ id (PK)                      │
│ name                         │
│ description                  │
│ quantity                     │
│ price                        │ ← NEW
│ sku                          │ ← NEW
│ min_stock_level              │ ← NEW
│ category_id (FK) ────────────┘ ← NEW foreign key
│ supplier_id (FK) ────────┐    ← NEW foreign key
│ location_id (FK) ──────┐ │    ← NEW foreign key
│ created_at              │ │  ← NEW
│ updated_at              │ │  ← NEW
└─────────────────────────┼─┼──┘
                          │ │
         ┌────────────────┘ │
         ↓ N:1              │
┌─────────────────────┐     │
│     suppliers       │     │
├─────────────────────┤     │
│ id (PK)             │     │
│ name                │     │
│ contact_email       │     │
│ contact_phone       │     │
│ created_at          │     │
└─────────────────────┘     │
                            │
         ┌──────────────────┘
         ↓ N:1
┌─────────────────────┐
│     locations       │
├─────────────────────┤
│ id (PK)             │
│ name                │
│ warehouse           │
│ aisle               │
│ shelf               │
│ created_at          │
└─────────────────────┘

┌──────────────────────────────┐
│    inventory_history         │ ← NEW audit table
├──────────────────────────────┤
│ id (PK)                      │
│ inventory_id (FK) ───────────┤ → Points to inventory
│ user_id (FK) ────────────────┤ → Points to user
│ action                       │
│ old_quantity                 │
│ new_quantity                 │
│ timestamp                    │
└──────────────────────────────┘

┌─────────────────────┐
│       users         │
├─────────────────────┤
│ id (PK)             │
│ username (UNIQUE!)  │ ← NOW UNIQUE
│ passwordHash        │
│ created_at          │ ← NEW
└─────────────────────┘
```
- Organizes items into groups
- Query all items ina  category
- Generate reports by category
- No data duplication

*Suppliers Table*
  - Tracks where items come from:
    - Track which supplier provides which items
    - Reorder from correct supplier
    - Manage supplier contact information
    - Query all itesm from a supplier

*Location Table*

  - Tracks where items are stored:
    - Find items physically in warehouse
    - Organize warehouse layout
    - Optimize picking/packing
    - Track item movements

*Enhanced Inventory Table*

  - Adds foreign keys and additional fields
    - category_id: references categories table
    - supplier_id: references suppliers table
    - location_id: references location table
  - 'onDelete'
    - cannot delete category/supplier if items use it
    - if location is deleted, sets location_id to null

*Inventory History Table*

- Tracks all changes:
  - Full audit trail of all changes
  - See who changed what and when
  - Detect unauthorized modifications
  - Compliance for regulated industries
  - Undo capability

*Enhanced User Table*

- Added Unique contraints:
  - Username uniqueness enforced at database level
  - Cannot create duplicate users
  - Login works correctly
  - Data integrity maintained

---

### Enhancement 2: Safe Migration Strategy

- Instead of destructive migration. Use a proper migration object:
  - Creates new tables:
    - categories
    - suppliers
    - locations
    - inventory_history
  - Creates a default 'Uncategorized' category. 
  - Uses ALTER TABLE to add new columns to the existing inventory table
  - Checks for duplicate usernames
  - handles any conflicts in user name creation
  - add unique index
  - Creates indices on foreign key columns for query performance
  - User updates app from version 1 to version 2
  - Migration runs automatically
  - All exisitng data preserved
  - New features available
  - User never knows migration happened
  - No data loss

---

### Enhancement 3: Advanced Query Support ###

  - Adds new queries to support the new schema
  - JOIN queries:
    - Combine data from multiple tables:
    
```sql
SELECT i.*, c.name as categoryName
FROM inventory i
LEFT JOIN categories c ON i.category_id = c.id
```
  - Gets inventory items along with category name
  - Aggregation Queries: Calculate totals, counts, averages

```sql
SELECT SUM(price * quantity) FROM inventory
```

```sql
SELECT c.name, COUNT(i.id) as itemCount
FROM categories c
LEFT JOIN inventory i ON c.id = i.category_id
GROUP BY c.id, c.name
```

  - Calculates total inventory value.
  - Shows how many items in each category

  - Filtering Queries

```sql
SELECT * FROM inventory
WHERE quantity < min_stock_level
ORDER BY (min_stock_level - quantity) DESC
```

- Finds items that are below their minimum stock level, ordered by urgency.

  - Search Queries:

```sql
SELECT * FROM inventory
WHERE name LIKE '%' || :searchTerm || '%'
```
  - Allows users to search for items by name

---

### Enhancement 4: Transaction Support ###

*Example*:

```java
@Transaction
@Query("SELECT * FROM inventory WHERE id = :id")
InventoryItemComplete getCompleteItem(long id);

// In InventoryActivity or Repository:
public void transferItem(long itemId, long fromLocationId, long toLocationId, int quantity) {
    db.runInTransaction(() -> {
        // Step 1: Get current item
        InventoryItem item = dao.getById(itemId);

        // Step 2: Verify sufficient quantity at source
        if (item.getQuantity() < quantity) {
            throw new IllegalStateException("Insufficient quantity");
        }

        // Step 3: Update item location
        item.setLocationId(toLocationId);
        dao.update(item);

        // Step 4: Record in history
        InventoryHistory history = new InventoryHistory();
        history.setInventoryId(itemId);
        history.setAction("TRANSFERRED");
        history.setOldQuantity(item.getQuantity());
        history.setNewQuantity(quantity);
        history.setTimestamp(System.currentTimeMillis());
        historyDao.insert(history);

        // ALL steps succeed or ALL fail (atomicity)
    });
}
```

  - Transactions ensure ACID properties:
    - Automicity: All operations succed or all fail
    - Consistency: Database remains in vald state
    - Isolation: Concurrent transactions don't interfere
    - Durability: Committed changes are permanent

---


## Normalization Demonstration ##

### First Normal Form (1NF)

"**First Normal Form (1NF)**: No repeating groups, atomic values

**Before (Violates 1NF if we stored multiple suppliers):**
```
inventory:
id | name    | suppliers
1  | Laptop  | Dell, HP, Lenovo  ← Multiple values in one field!
```

**After (Meets 1NF):**
```
inventory:
id | name    | supplier_id
1  | Laptop  | 1              ← Single atomic value

suppliers:
id | name
1  | Dell
2  | HP
3  | Lenovo
```

### Second Normal Form (2NF)

"**Second Normal Form (2NF)**: Meets 1NF + no partial dependencies

**Before (If we had composite key):**
```
inventory:
(category_id, item_id) | item_name | category_name
(1, 1)                 | Laptop    | Electronics  ← category_name depends only on category_id
```

**After (Meets 2NF):**
```
inventory:
id | name    | category_id
1  | Laptop  | 1

categories:
id | name
1  | Electronics  ← Stored once
```

### Third Normal Form (3NF)

"**Third Normal Form (3NF)**: Meets 2NF + no transitive dependencies

**Before (Violates 3NF):**
```
inventory:
id | name    | supplier_id | supplier_name | supplier_email
1  | Laptop  | 1           | Dell          | dell@example.com
2  | Monitor | 1           | Dell          | dell@example.com
   ↑                        ↑ supplier_name depends on supplier_id (transitive!)
```

**After (Meets 3NF):**
```
inventory:
id | name    | supplier_id
1  | Laptop  | 1
2  | Monitor | 1

suppliers:
id | name | email
1  | Dell | dell@example.com  ← Stored once, no redundancy
```

**[Summary]**

My new schema meets Third Normal Form (3NF):
-  Atomic values (1NF)
-  No partial dependencies (2NF)
-  No transitive dependencies (3NF)
-  Minimal redundancy
-  Maximum data integrity

---

## SECTION 5: SKILLS DEMONSTRATION



This database enhancement demonstrates several important database and software engineering skills.

### Database Design

"**Database Design Skills**:

1. **Entity-Relationship Modeling**
   - Identifying entities (Category, Supplier, Location)
   - Defining relationships (1:N, N:1)
   - Drawing ER diagrams
   - Understanding cardinality

2. **Normalization**
   - Applying First, Second, Third Normal Forms
   - Eliminating data redundancy
   - Ensuring data integrity
   - Preventing update/insertion/deletion anomalies

3. **Schema Design**
   - Choosing appropriate data types
   - Defining primary and foreign keys
   - Creating indices for performance
   - Setting up constraints (UNIQUE, NOT NULL, CHECK)

### SQL & Query Optimization ###

"**SQL Skills**:

1. **DDL (Data Definition Language)**
   - CREATE TABLE statements
   - ALTER TABLE for migrations
   - CREATE INDEX for performance
   - Foreign key constraints

2. **DML (Data Manipulation Language)**
   - INSERT, UPDATE, DELETE
   - Complex WHERE clauses
   - Transaction control

3. **DQL (Data Query Language)**
   - JOIN queries (INNER, LEFT, RIGHT)
   - Aggregation (SUM, COUNT, AVG, MIN, MAX)
   - GROUP BY and HAVING
   - Subqueries
   - Performance optimization"

### ORM (Object-Relational Mapping)

"**Room/ORM Skills**:

1. **Entity Mapping**
   - @Entity annotations
   - @ColumnInfo for column names
   - @PrimaryKey and auto-generation
   - @ForeignKey with cascade rules

2. **DAO Design**
   - @Query with SQL
   - @Insert, @Update, @Delete
   - @Transaction for ACID
   - LiveData and Flow integration

3. **Migration Strategy**
   - Version management
   - Safe data migration
   - Schema evolution
   - Backward compatibility"

### Database Best Practices

"**Professional Practices**:

1. **Data Integrity**
   - Referential integrity with foreign keys
   - Unique constraints
   - NOT NULL constraints
   - Check constraints

2. **Performance**
   - Index strategic columns
   - Avoid N+1 query problem
   - Use proper JOIN types
   - Pagination for large datasets

3. **Security & Audit**
   - Password hashing (already implemented)
   - Audit trail (history table)
   - Track who changed what when
   - Data validation

4. **Scalability**
   - Normalized structure
   - Efficient queries
   - Proper indexing
   - Transaction management"

---


## SECTION 6: COURSE OUTCOMES ALIGNMENT

### Outcome 4: Well-Founded Techniques (PRIMARY)

"**Primary Alignment**: Outcome 4 - Demonstrate an ability to use well-founded techniques

This enhancement demonstrates:

1. **Industry-Standard Database Design**
   - Third Normal Form (3NF) - taught in every database course
   - Foreign key constraints - fundamental database concept
   - ACID transactions - core database principle
   - ER modeling - standard design methodology

2. **Professional Migration Practices**
   - Safe data migration preserving user data
   - Version control with schema export
   - Backward compatibility considerations
   - Used in all professional database applications

3. **SQL Best Practices**
   - JOIN queries for related data
   - Aggregation for reporting
   - Indices for performance
   - Constraints for data integrity

4. **Android/Room Best Practices**
   - Proper entity annotations
   - Background thread operations (remove allowMainThreadQueries)
   - DAO pattern
   - Migration objects"

### Outcome 3: Algorithmic Principles ###

"**Secondary Alignment**: Outcome 3 - Algorithmic principles

Database operations involve algorithms:

- **Query optimization**: Database uses B-tree (O(log n)) for indexed lookups
- **JOIN algorithms**: Hash join, sort-merge join
- **Sorting**: Database ORDER BY uses efficient sorting algorithms
- **Search**: Index lookups are O(log n) vs O(n) table scans"

### Outcome 5: Security Mindset ###

"**Alignment**: Outcome 5 - Security mindset

- Password hashing (already implemented, maintained)
- Audit trail (track all changes)
- Referential integrity (prevent invalid data)
- Data validation constraints
- Transaction ACID properties prevent corruption"

### Outcome 2: Professional Communication ###

"**Alignment**: Outcome 2 - Professional communication

- This comprehensive 50+ page code review document
- ER diagrams and schema documentation
- Migration documentation
- Professional technical writing"

---
### Summary ###

"**Current State Issues**:
- Destructive migration DELETES all user data on updates
- Only 2 tables with zero relationships
- Denormalized flat structure
- No username uniqueness
- Missing critical fields
- Main thread database queries
- No advanced query capabilities

**Enhanced Solution**:
- Safe migration preserving ALL data
- 6 tables with proper relationships
- Third Normal Form (3NF) normalization
- Foreign keys for referential integrity
- Complete field set (price, SKU, timestamps, etc.)
- Background database operations
- Advanced JOINs, aggregation, searching
- Full audit trail
- Transaction support for ACID properties"

### Expected Results

"When this enhancement is complete:

**For Users**:
- Zero data loss on app updates
- Organize items by category
- Track suppliers and locations
- Calculate total inventory value
- See full change history
- Search and filter capabilities
- Better performance

**For Business**:
- Professional database design
- Scalable architecture
- Regulatory compliance (audit trail)
- Reporting capabilities
- Data integrity guaranteed

**For My Skills Demonstration**:
- Database design expertise (ER modeling, normalization)
- SQL proficiency (DDL, DML, DQL)
- ORM mastery (Room/Android)
- Migration strategies
- Best practices implementation
- Professional-quality code"

---

### Conclusion ###

This database enhancement transforms my simple 2-table structure into a professional, normalized, relational database that follows industry best practices. It demonstrates comprehensive understanding of database design principles, SQL, ORMs, and data integrity - all while ensuring zero data loss for users. Thank you for watching this presentation on my Database Design and Implementation enhancement.
