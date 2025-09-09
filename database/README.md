# 📊 Difference between MySQL and PostgreSQL

## 🟢 MySQL
- Relational Database for **read-heavy workloads**।  
- সহজ সেটআপ, দ্রুত read query।  
- বহুল ব্যবহৃত → WordPress, Shopify, Uber ইত্যাদি।  

### বৈশিষ্ট্য
1. **Performance (Read Heavy)** → `SELECT` কোয়েরি দ্রুত।  
2. **Replication** সহজ (master-slave architecture)।  
3. **JSON support আছে**, কিন্তু PostgreSQL এর মত advanced না।  
4. Stored procedure, Trigger → সীমিত ফিচার।  
5. শেখা সহজ এবং developer-friendly।  

---

## 🔵 PostgreSQL
- Object-Relational Database (RDBMS + কিছু NoSQL ফিচার)।  
- Complex data, transaction-heavy workload এর জন্য উপযোগী।  

### বৈশিষ্ট্য
1. **ACID compliant & Strong Transaction Support**।  
2. **Advanced JSON / JSONB support** → MongoDB এর মত আচরণ করতে পারে।  
3. **Indexes (GIN, GiST, BRIN)** → জটিল সার্চে ভালো।  
4. **CTE, Window Functions, Full-Text Search** সাপোর্ট করে।  
5. **Partitioning, Sharding** এর মাধ্যমে horizontal scalability।  

---

## ⚖️ প্রধান পার্থক্য

| বিষয় | MySQL | PostgreSQL |
|------|-------|------------|
| **ফোকাস** | সহজ, দ্রুত, Read-heavy | Complex queries, Transaction-heavy |
| **JSON Support** | Limited | Powerful (JSONB) |
| **Indexes** | Basic | Advanced (GIN, GiST, BRIN) |
| **Scalability** | Replication সহজ | Partitioning, Sharding, Extensions |
| **Use Case** | Web apps, CMS (WordPress, Joomla) | Fintech, Analytics, Data-heavy apps |
| **Performance** | Faster in simple queries | Better for complex queries |

---

## 🛠 প্রজেক্টে ব্যবহারের সিদ্ধান্ত
- ✅ **MySQL** → Read-heavy API, CMS, simple e-commerce।  
- ✅ **PostgreSQL** → Financial transactions, analytics, complex queries।  

---

## 🧑‍💻 Interview Questions & Answers

### 1. MySQL আর PostgreSQL এর প্রধান পার্থক্য কী?  
- MySQL → simple, read-heavy workload।  
- PostgreSQL → complex queries, transaction-heavy workload।  

---

### 2. JSONB কেন PostgreSQL এ ভালো?  
- JSONB binary format এ data রাখে → compressed + indexed।  
- Query faster এবং filtering efficient।  
- MySQL এর JSON এ এই advanced indexing নাই।  

---

### 3. কোন ক্ষেত্রে MySQL, কোন ক্ষেত্রে PostgreSQL?  
- **MySQL:** read-heavy apps (CMS, blogs, e-commerce)।  
- **PostgreSQL:** transaction-heavy apps (Banking, analytics, JSON-heavy workloads)।  

---

### 4. PostgreSQL এর CTE / Window Function Example

**CTE (Common Table Expression):**
```sql
WITH recent_orders AS (
    SELECT user_id, order_id
    FROM orders
    WHERE created_at > now() - interval '30 days'
)
SELECT user_id, COUNT(*) as total_orders
FROM recent_orders
GROUP BY user_id;
```

### Window Function

```sql
SELECT user_id,
       order_id,
       amount,
       RANK() OVER (PARTITION BY user_id ORDER BY amount DESC) as rank
FROM orders;
```
এখানে প্রতি user এর top spending order বের করা যায়।

### প্রশ্ন 5: Scaling এর জন্য MySQL এ কী করবে? PostgreSQL এ কী করবে?

উত্তর:

#### MySQL Scaling:

* Read Replicas (master-slave replication)।

* Sharding (range/hash based)।

* Query optimization + caching।

#### PostgreSQL Scaling:

* Partitioning (range/list/hash)।

* Logical replication (publish/subscribe)।

* Connection pooler (PgBouncer)।

* Sharding via Citus (extension)।


# PostgreSQL basics (DDL, DML)

1. DDL (Data Definition Language)

    👉 ডাটাবেজে structure define বা change করার জন্য ব্যবহার হয়।
    উদাহরণ: CREATE, ALTER, DROP, TRUNCATE

2. DML (Data Manipulation Language)

    👉 ডাটার সাথে কাজ করার জন্য ব্যবহার হয়।
    উদাহরণ: INSERT, UPDATE, DELETE, SELECT

## Best Practices

* Table design করার সময় primary key অবশ্যই রাখতে হবে।

* Unique fields (যেমন email, username) এর জন্য UNIQUE constraint ব্যবহার করো।

* Date/time এর জন্য TIMESTAMP ব্যবহার করো DEFAULT NOW() দিয়ে।

* সবসময় WHERE clause ছাড়া UPDATE বা DELETE লিখো না (না হলে সব ডাটা চলে যাবে 😅)।

* Index ব্যবহার করো বেশি query হওয়া columns এর জন্য।


## PostgreSQL এ Primary key আর Unique key এর মধ্যে পার্থক্য কী?
🔑 মূল পার্থক্য

| Feature                | Primary Key                | Unique Key                      |
|------------------------|----------------------------|----------------------------------|
| সংখ্যা (Number)        | প্রতি টেবিলে ১টা           | একাধিক থাকতে পারে               |
| NULL allow করে?       | ❌ না                       | ✅ হ্যাঁ (একটা বা একাধিক NULL)   |
| উদ্দেশ্য (Purpose)     | Row uniquely identify করে  | Duplicate value আটকায়           |
| Implicit Index         | Auto create হয়             | Auto create হয়                   |



# Database Performance

Database Performance সিস্টেম ডিজাইনে খুবই গুরুত্বপূর্ণ বিষয়।

## Database Indexing

সাধারণত ডেটা Disk-এ সংরক্ষন হয়ে থাকে। যখন ডেটা বেড়ে যায় তখন সেই ডেটাগুলো থেকে Query করতে অনেক সময় লাগে, এই সময় কমানোর জন্য আরেকটি টেবিল Disk-এ তৈরী হয় যাকে Index Table বলে। এই Index Table-এ মূলত আমাদের মূল টেবিল এর row(s) এর সাথে একটি লিংক করা থাকে, সেটি key-value আকারেও থাকতে পারে। যখন নতুন row কিংবা entry ডেটাবেস টেবিলে insert হয়, Index Table-এ সেই নতুন ডেটার সাথে একটি লিংক তৈরী হয় (সেজন্য আমাদের write operation slow হয়ে যেতে পারে আর read operation fast হয়)।

পরবর্তী সময়ে যখন কেউ নির্দিষ্ট ডেটা query করবে, তখন Index Table বলে দিবে কোন এড্ড্রেসে বা কোন ব্লকে ডেটা আছে।

## Indexing কীভাবে Query দ্রুত করে?

Index মূলত ডেটার জন্য একটি lookup table তৈরি করে। যখন কোনো column-এ index থাকে, তখন ডেটাবেস engine পুরো টেবিল না ঘেঁটে index table-এ খুঁজে দ্রুত row-এর location বের করতে পারে।  
এতে করে:

- **Full Table Scan** এড়ানো যায় (সব row পড়তে হয় না)।
- **Search, JOIN, WHERE, ORDER BY** query-তে দ্রুত ফলাফল পাওয়া যায়।
- Index tree (যেমন B-Tree) ব্যবহার করে log(n) টাইমে ডেটা খুঁজে পাওয়া যায়।

**উদাহরণ:**  
যদি ১০ লাখ row-এর টেবিলের email column-এ index থাকে,  
```sql
SELECT * FROM users WHERE email = 'abc@example.com';
```
এখানে ডেটাবেস পুরো টেবিল না পড়ে index থেকে দ্রুত row খুঁজে বের করবে।

**সতর্কতা:**  
- Index বেশি দিলে write operation (INSERT/UPDATE/DELETE) ধীর হতে পারে।
- সব column-এ index না দিয়ে, query-তে বেশি ব্যবহৃত column-এ index দাও।

* [https://codemacaw.com/database-indexing-makes-db-query-faster/](https://codemacaw.com/database-indexing-makes-db-query-faster/)
* [https://codemacaw.com/what-is-b-tree-b-tree-in-dbms/](https://codemacaw.com/what-is-b-tree-b-tree-in-dbms/)

## Indexing Strategies & Types

### 1. Single-Column Index
- শুধু একটি column-এ index।
- সাধারণত WHERE, ORDER BY, JOIN-এ বেশি ব্যবহৃত column-এ দেওয়া হয়।

### 2. Composite (Multi-Column) Index
- একাধিক column নিয়ে তৈরি।
- Query-তে একাধিক column একসাথে filter করলে দ্রুত ফলাফল দেয়।
- Column order গুরুত্বপূর্ণ (index(left, right) → left/right; index(right, left) → right/left)।

### 3. Unique Index
- Duplicate value আটকায়।
- Primary/Unique key constraint দিলে অটো unique index হয়।

### 4. Partial Index (PostgreSQL)
- নির্দিষ্ট condition-এ index তৈরি (যেমন: active=true)।
- কম ডেটা, কম storage, targeted query দ্রুত।

### 5. Covering Index (Index Only Scan)
- Index-এ query-র সব column থাকলে, মূল টেবিল না পড়ে শুধু index থেকেই ফলাফল দেয়।

---

## Index Types & Their Differences

| Index Type      | MySQL Support | PostgreSQL Support | Use Case / Advantage                |
|-----------------|--------------|-------------------|-------------------------------------|
| **B-Tree**      | ✅            | ✅                 | Default, range/search/query         |
| **Hash**        | ✅ (limited)  | ✅                 | Exact match (PostgreSQL only)       |
| **GIN**         | ❌            | ✅                 | Array, JSONB, Full-text search      |
| **GiST**        | ❌            | ✅                 | Geospatial, custom data types       |
| **BRIN**        | ❌            | ✅                 | Very large, sequential data         |
| **Full-Text**   | ✅            | ✅                 | Text search                         |
| **Spatial**     | ✅            | ✅                 | Geo data (GIS)                      |

---

## PostgreSQL Indexing Strategies & Types: Examples & In-Depth Explanation

### 1. Single-Column Index

**Explanation:**  
একটি নির্দিষ্ট column-এ index তৈরি করলে, সেই column-এ দ্রুত search, filter, বা sort করা যায়।

**Example:**
```sql
CREATE INDEX idx_users_email ON users(email);
```
এখন `WHERE email = ...` বা `ORDER BY email` query-তে দ্রুত ফলাফল পাওয়া যাবে।

---

### 2. Composite (Multi-Column) Index

**Explanation:**  
একাধিক column-এ একসাথে filter বা sort করলে composite index দ্রুত ফলাফল দেয়। Column order গুরুত্বপূর্ণ—index `(a, b)` দিলে `WHERE a=... AND b=...` দ্রুত, কিন্তু শুধু `b=...` ততটা দ্রুত না।

**Example:**
```sql
CREATE INDEX idx_orders_userid_status ON orders(user_id, status);
```
এখন `WHERE user_id = ... AND status = ...` query দ্রুত হবে।

---

### 3. Unique Index

**Explanation:**  
একটি column বা column set-এ duplicate value আটকাতে unique index ব্যবহার হয়। Primary/Unique key দিলে অটো unique index হয়।

**Example:**
```sql
CREATE UNIQUE INDEX idx_users_username ON users(username);
```
এখন একই username দুইবার insert করা যাবে না।

---

### 4. Partial Index (PostgreSQL Only)

**Explanation:**  
শুধু নির্দিষ্ট condition-এ index তৈরি হয়। যেমন, active user-দের জন্য index।

**Example:**
```sql
CREATE INDEX idx_users_active ON users(email) WHERE active = true;
```
এখন `WHERE active = true AND email = ...` query-তে index ব্যবহার হবে, inactive user-দের জন্য হবে না।

---

### 5. Covering Index (Index Only Scan)

**Explanation:**  
যদি query-তে যেসব column লাগে, সব index-এ থাকে, তাহলে PostgreSQL শুধু index থেকেই ফলাফল দিতে পারে—মূল টেবিল পড়তে হয় না।

**Example:**
```sql
CREATE INDEX idx_orders_userid_amount ON orders(user_id, amount);
```
এখন নিচের query-তে index only scan হতে পারে:
```sql
SELECT user_id, amount FROM orders WHERE user_id = 123;
```

---

## Index Types: Examples & Use Cases

### B-Tree Index (Default)

**Explanation:**  
সবচেয়ে সাধারণ index; range query, equality, sorting—সবকিছুতে দ্রুত।

**Example:**
```sql
CREATE INDEX idx_products_price ON products(price);
```

---

### Hash Index

**Explanation:**  
শুধু exact match query-তে দ্রুত (range query-তে না)। PostgreSQL-এ কম ব্যবহৃত।

**Example:**
```sql
CREATE INDEX idx_users_hash_email ON users USING hash(email);
```
`WHERE email = ...` query-তে দ্রুত, কিন্তু `ORDER BY` বা range query-তে নয়।

---

### GIN (Generalized Inverted Index)

**Explanation:**  
Array, JSONB, full-text search-এর জন্য। Nested/compound value-তে দ্রুত search।

**Example:**
```sql
CREATE INDEX idx_posts_tags_gin ON posts USING gin(tags);
-- tags: text[] array

CREATE INDEX idx_data_json_gin ON data USING gin(info jsonb);
-- info: jsonb column
```
এখন `WHERE tags @> ARRAY['postgres']` বা `WHERE info @> '{"active": true}'` query-তে দ্রুত ফলাফল।

---

### GiST (Generalized Search Tree)

**Explanation:**  
Geospatial, custom data type, range query-তে ব্যবহৃত।

**Example:**
```sql
CREATE INDEX idx_locations_geom_gist ON locations USING gist(geom);
-- geom: geometry type (PostGIS)
```
এখন spatial query (যেমন, কোন point কোন area-তে আছে) দ্রুত হবে।

---

### BRIN (Block Range Index)

**Explanation:**  
Huge, sequential data (যেমন time-series) এর জন্য। কম storage, দ্রুত range scan।

**Example:**
```sql
CREATE INDEX idx_logs_created_brin ON logs USING brin(created_at);
```
এখন `WHERE created_at BETWEEN ...` query-তে দ্রুত ফলাফল, বিশেষ করে বড় টেবিলে।

---

### Full-Text Index

**Explanation:**  
Text search-এর জন্য; natural language query-তে দ্রুত ফলাফল।

**Example:**
```sql
CREATE INDEX idx_articles_body_fts ON articles USING gin(to_tsvector('english', body));
```
এখন `WHERE to_tsvector('english', body) @@ plainto_tsquery('search term')` query-তে দ্রুত ফলাফল।

---

### Spatial Index

**Explanation:**  
Geospatial data (GIS) query-তে দ্রুত ফলাফল।

**Example:**
```sql
CREATE INDEX idx_places_location_gist ON places USING gist(location);
-- location: geography/geometry type
```

---

## Indexing Under the Hood: Visualization

- **B-Tree:**  
    ```
    [root]
        /    \
 [node] [node]
    ```
    Balanced tree, fast search/insert/delete (O(log n)).

- **GIN:**  
    Inverted index—value থেকে row pointer।

- **BRIN:**  
    Block-wise min/max value store—range scan দ্রুত।

---

**Tip:**  
`EXPLAIN ANALYZE` দিয়ে query-র index usage দেখতে পারো।
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'abc@example.com';
```
