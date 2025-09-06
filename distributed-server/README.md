# Distributed Server কী?

একটা Distributed Server/System মানে হলো —

    অনেকগুলো সার্ভার একসাথে কাজ করে যেন ইউজারের কাছে পুরো সিস্টেমটাকে একটা ইউনিফাইড সার্ভার মনে হয়।

অর্থাৎ, কাজগুলো ভাগ হয়ে যায় (distribution হয়) একাধিক সার্ভারে, কিন্তু ইউজারের কাছে সেটা অদৃশ্য থাকে।

# কেন দরকার?

### একটা সার্ভার (single server) সীমিত:

* CPU, RAM, Storage সীমাবদ্ধ

* একসাথে অনেক ইউজার এলে লোড সামলাতে পারে না

* Fail হলে পুরো সিস্টেম বন্ধ হয়ে যায়

### 👉 সমাধান = Distributed Server

* অনেক সার্ভার একসাথে কাজ করে Scalability আনে

* Fault tolerance হয় (একটা সার্ভার নষ্ট হলেও অন্যগুলো কাজ চালিয়ে যায়)

* High availability (২৪/৭ সার্ভিস)

# Distributed Server এর Components

- Load Balancer → রিকোয়েস্ট কোন সার্ভারে যাবে ঠিক করে।

- Application Servers → কোড রান করে (NestJS, Django, etc)।

- Database Cluster → ডেটা ভাগ হয়ে store হয় (sharding, replication)।

- Cache (Redis) → দ্রুত রেসপন্স দেয়।

- Message Queue (Kafka, RabbitMQ) → async communication এর জন্য।

# Interview Questions

- Distributed system কী?

- Distributed server আর single server এর মধ্যে পার্থক্য কী?

- CAP theorem কী?

- Distributed system এ ডেটা consistency কিভাবে maintain করবে?
- Fault tolerance কিভাবে আনা যায়?
- Load balancing এর ভূমিকা কী?

- যদি তোমার ১০০ সার্ভার থাকে, ইউজার রিকোয়েস্ট কিভাবে সঠিক সার্ভারে পাঠাবে?
- Leader election বলতে কী বোঝায় distributed system এ?
- Latency কমাতে worldwide distributed server deploy করলে কী কী সমস্যা হতে পারে?

# Voice Chat App – Distributed Architecture

```
 ┌───────────────┐        ┌───────────────────────────┐
 │   React Native │  ---> │     API Gateway / LB      │
 │   (Mobile App) │       │ (NGINX / AWS ALB / Kong)  │
 └───────────────┘        └───────────────┬───────────┘
                                          │
                   ┌──────────────────────┼───────────────────────┐
                   │                      │                       │
        ┌──────────▼──────────┐ ┌─────────▼───────────┐ ┌────────▼─────────┐
        │ Chat Service (Nest) │ │ Audio Service (Django│ │   Auth Service   │
        │  - WebSocket/RTC    │ │   - Upload/Storage   │ │  - JWT / OAuth   │
        └──────────┬──────────┘ └─────────┬───────────┘ └────────┬─────────┘
                   │                      │                       │
                   │                      │                       │
        ┌──────────▼─────────┐   ┌────────▼─────────┐    ┌────────▼─────────┐
        │   Redis Cluster    │   │  Object Storage   │    │   Database (SQL/ │
        │ - Rate Limiting    │   │  (S3, MinIO)      │    │    NoSQL, Shard) │
        │ - Session Store    │   │  - Audio files    │    │ - User data      │
        │ - Pub/Sub (Chat)   │   │                   │    │ - Chat metadata  │
        └──────────┬─────────┘   └────────┬─────────┘    └────────┬─────────┘
                   │                      │                       │
        ┌──────────▼─────────┐   ┌────────▼─────────┐    ┌────────▼─────────┐
        │  Kafka / RabbitMQ  │   │  CDN (CloudFront │    │ Monitoring/Logs  │
        │  - Async tasks     │   │   Akamai, etc.)  │    │ (ELK, Grafana)   │
        │  - Event streaming │   │  - Fast delivery │    │                  │
        └────────────────────┘   └──────────────────┘    └──────────────────┘
```

## কীভাবে কাজ করে

### Client (React Native App)

ইউজার ভয়েস মেসেজ পাঠায় বা শোনে।

সব রিকোয়েস্ট প্রথমে API Gateway / Load Balancer এর কাছে যায়।

### API Gateway / Load Balancer

ট্রাফিক সঠিক সার্ভারে রাউট করে (chat, auth, audio)।

এখানে প্রাথমিক Rate Limiting বসানো যায়।

### Chat Service (NestJS)

WebSocket/RTC হ্যান্ডল করে।

Redis Pub/Sub ব্যবহার করে রিয়েলটাইম মেসেজ সব সার্ভারে সিঙ্ক করে।

যদি এক ইউজার সার্ভার A তে আরেক ইউজার সার্ভার B তে থাকে, Redis এর মাধ্যমে মেসেজ শেয়ার হয়।

### Audio Service (Django)

ভয়েস ফাইল আপলোড নেয়।

ফাইলগুলো Object Storage (S3/MinIO) এ সেভ হয়।

Metadata Database এ যায়।

### Auth Service

JWT / OAuth2 দিয়ে ইউজার অথেনটিকেট করে।

Token validation সবার জন্য একই থাকে।

## Redis Cluster

Rate limiter store.

WebSocket সেশনের ডাটা।

Pub/Sub চ্যানেল (রিয়েলটাইম মেসেজ সিঙ্ক)।

### Kafka / RabbitMQ

Async কাজ যেমন → ভয়েস ফাইল প্রসেসিং, নোটিফিকেশন পাঠানো, analytics data সংগ্রহ।

### CDN

রেকর্ড করা ভয়েস ফাইল সার্ভ করতে।

Latency কমায়।

### Database (SQL + NoSQL)

SQL → User, auth, billing ইত্যাদি।

NoSQL (MongoDB/Cassandra) → Chat message, history।

Sharding/replication করে distributed data storage।

### Monitoring / Logging

Prometheus + Grafana → সার্ভারের health check।

ELK Stack (ElasticSearch, Logstash, Kibana) → Error tracking।

## Benefits

* Scalability → নতুন সার্ভার যোগ করলে load balancer ভাগ করে দেবে।
* High Availability → একটা সার্ভার ডাউন হলেও বাকি সার্ভার কাজ চালাবে।
* Low Latency → CDN ও Redis caching ইউজারকে দ্রুত রেসপন্স দেবে।
* Fault Tolerance → Kafka / RabbitMQ async টাস্ক ব্যাকআপ রাখে।

## Interview Style Questions

#### যদি Chat Server ডাউন হয়ে যায়, অন্য ইউজারের মেসেজ কি হারাবে? কিভাবে নিশ্চিত করবে?

#### Audio ফাইল সেভ করার সময় তুমি Database এ রাখবে না Object Storage এ রাখবে? কেন?

#### Redis Pub/Sub এর উপর নির্ভর করলে কী সমস্যা হতে পারে? Kafka দিয়ে কি সমাধান করা যায়?

#### তুমি কীভাবে Global Rate Limit আর Per-user Rate Limit একসাথে ম্যানেজ করবে?

#### যদি ১ মিলিয়ন ইউজার একসাথে অনলাইন থাকে, তখন সিস্টেমে কোন bottleneck আসতে পারে?