Detailed notes on *12. Mastering Databases with Postgres* for backend engineering.

### 1. Fundamental Concepts
*   *Persistence:* The core purpose of a database is to ensure data survives after the program stops running.
*   *Disk vs. RAM:*
    *   *RAM (Primary Memory):* Very fast but volatile and expensive. Used for caching (e.g., Redis).
    *   *Disk (Secondary Memory):* Slower but cheaper and persistent. Databases (Postgres, Mongo) store data here to handle large capacities (terabytes) reliably.
*   *Why not Text Files?* Storing data in simple text files causes issues with parsing speed, lack of structure/schema, and *concurrency* (handling multiple users trying to update the same data simultaneously).

### 2. Choosing a Database
*   *Relational (SQL):* Organizes data in tables/rows/columns with a strict schema. Best for data integrity (e.g., CRM systems).
*   *Non-Relational (NoSQL):* Flexible schema (documents). Good for dynamic, unstructured content (e.g., Content Management Systems).
*   *Why Postgres?*
    *   *Open Source & Standard Compliant:* Free and sticks to SQL standards, making migration easier.
    *   *JSON Support:* The "killer feature." Postgres supports `JSON` and `JSONB` types, allowing you to store unstructured dynamic data (like MongoDB) inside a relational database.

### 3. Postgres Data Types & Best Practices
*   *Integers:*
    *   `Serial` / `BigSerial`: Auto-incrementing integers, usually used for IDs.
    *   `SmallInt`, `Integer`, `BigInt`: Choose based on the size of the number you need to store.
*   *Floats vs. Decimals:*
    *   *Decimal/Numeric:* Stores exact precision. *Always use this for money/prices* to avoid calculation errors.
    *   *Float/Real:* Floating-point numbers. Faster, but can have slight accuracy discrepancies. Use for scientific calculations.
*   *Strings:*
    *   `Char(n)`: Fixed length (pads with spaces). Avoid using this.
    *   `Varchar(n)`: Variable length with a limit. Note: `255` is often just a legacy habit from MySQL.
    *   *`Text`:* Variable length with no limit. *Recommendation:* Prefer `Text` over `Varchar` in Postgres. It is just as performant and avoids future migration headaches if you need to increase string length.
*   *Other Types:*
    *   *`UUID`:* Universally Unique Identifier. Safer and better for distributed systems than integer IDs.
    *   *`JSONB`:* Binary JSON. Always prefer this over standard `JSON` because Postgres can index it and query it faster.
    *   *`Enum`:* A custom type restricted to a specific set of values (e.g., Status: 'Pending', 'Completed'). Great for data integrity and self-documenting code.

### 4. Database Migrations
*   *Definition:* Migrations are version control for your database schema.
*   *Workflow:*
    *   *Up Migration:* Applies changes (e.g., Create Table).
    *   *Down Migration:* Reverts changes (e.g., Drop Table). Used to "roll back" if an update breaks the system.
*   *Why use them?* They track changes over time and ensure every developer/server has the exact same database structure.

### 5. Data Modeling (Relationships)
*   *Naming Conventions:* Use *plural* for table names (`users`, `projects`) and *snake_case* for columns (`full_name`) because Postgres is case-insensitive.
*   *One-to-One (User ↔ User Profile):*
    *   Split into two tables to keep the main user table lightweight.
    *   The Profile table uses the User ID as both its Primary Key and Foreign Key.
*   *One-to-Many (Project ↔ Tasks):*
    *   A Project has many Tasks.
    *   The `tasks` table contains a `project_id` foreign key.
*   *Many-to-Many (Users ↔ Projects):*
    *   A User can have many Projects; a Project can have many Users.
    *   Requires a *Linking Table* (e.g., `project_members`).
    *   Uses a *Composite Primary Key* (a combination of `user_id` and `project_id`).

### 6. Constraints & Integrity
*   *Primary Key:* Implicitly `Unique` and `Not Null`.
*   *Foreign Key:* Ensures you cannot reference a record that doesn't exist.
*   *Check Constraint:* Enforces custom logic at the database level (e.g., `CHECK priority BETWEEN 1 AND 5`).
*   *Referential Integrity (On Delete):*
    *   `Restrict`: Prevents deleting a User if they still own Projects.
    *   `Cascade`: If a Project is deleted, automatically delete all its Tasks.

### 7. Performance & Security
*   *Parameterized Queries (SQL Injection):*
    *   *Never* concatenate strings to build a query.
    *   Use placeholders (parameters). The database treats the input strictly as a string, preventing malicious code execution.
*   *Indexes:*
    *   Concept: Like a book index, it allows the DB to find a row without scanning every single item (Sequential Scan).
    *   When to Index: Create indexes on columns used in *`WHERE`* clauses, *`JOIN`* conditions, or *`ORDER BY`* sorting.
    *   Trade-off: Indexes speed up Reads but slightly slow down Writes (Insert/Update) because the index must be maintained.
*   *Triggers:*
    *   Used to automate tasks. A common use case is a trigger that automatically updates the `updated_at` timestamp whenever a row is modified, so the application code doesn't have to do it manually.

### 8. API Query Design
*   *Fetching Lists:* Always support *Pagination* (`LIMIT` and `OFFSET`) to avoid fetching too much data at once.
*   *Filtering:* Use `ILIKE` for case-insensitive pattern matching (e.g., searching for a name).
*   *Joins:* Use `LEFT JOIN` if you want to keep records from the main table even if the related table has no data (e.g., get Users even if they don't have a Profile).