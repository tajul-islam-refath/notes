# Rate Limiter কী?

Rate Limiter হলো একটা নিয়ন্ত্রণ ব্যবস্থা, যা বলে দেয় একজন ইউজার বা সিস্টেম কতগুলো রিকোয়েস্ট নির্দিষ্ট সময়ের মধ্যে করতে পারবে।

👉 কেন দরকার?

- অপব্যবহার রোধ (যেমন বট দিয়ে হাজার রিকোয়েস্ট পাঠানো)।

- ফেয়ারনেস নিশ্চিত করা (একজন ইউজার যেন সার্ভারের সব রিসোর্স খেয়ে না ফেলে)।

- সার্ভার সুরক্ষা (হঠাৎ ট্রাফিক স্পাইক এলে সার্ভার ক্র্যাশ না করা)।

উদাহরণ:
একটা API তে প্রতি ইউজারের জন্য প্রতি মিনিটে ১০০ রিকোয়েস্ট অনুমোদন করা হলো।
যদি কেউ বেশি পাঠায় → সে 429 Too Many Requests রেসপন্স পাবে।

# Rate Limiting এর অ্যালগরিদম

### Fixed Window Counter

 * নির্দিষ্ট সময়ে কাউন্টার চালায় (যেমন ১ মিনিটে সর্বোচ্চ ১০০ রিকোয়েস্ট)।

 * সমস্যা: মিনিটের শেষে ও শুরুতে বার্স্ট (burst) হতে পারে।

### Sliding Window Log

 * প্রতিটা রিকোয়েস্টের টাইমস্ট্যাম্প রেখে দেয়।

 * খুব এক্স্যাক্ট, কিন্তু মেমরি বেশি খায়।

### Sliding Window Counter

 * আগের উইন্ডোর আংশিক কাউন্ট + বর্তমান কাউন্ট মিশিয়ে নেয়।

 * ব্যালান্সড উপায়।

### Token Bucket (সবচেয়ে জনপ্রিয়)

 * একটা বালতিতে টোকেন থাকে। নির্দিষ্ট হারে টোকেন যোগ হয়।

 * প্রতিটি রিকোয়েস্টে ১টা টোকেন খরচ হয়। টোকেন না থাকলে রিকোয়েস্ট ব্লক হয়।

 * বার্স্ট (burst) সামলাতে ভালো।

### Leaky Bucket

 * কিউতে পানি (রিকোয়েস্ট) ঢোকে, নির্দিষ্ট হারে বের হয়।

 * হঠাৎ স্পাইক স্মুথ করে দেয়।

# Best Practices

* Short-term (burst) + Long-term (sustained) লিমিট একসাথে ব্যবহার করো।
* Redis / Memcached এর মতো fast datastore ব্যবহার করো।
* ইউজারকে সঠিক এরর কোড ফেরত দাও → 429 Too Many Requests।
* Abuse detection এর জন্য লগ রাখো।
* Premium আর Free ইউজারের জন্য আলাদা রেট লিমিট সেট করো।

# বাস্তব Use Case

* Login API → brute force attack ঠেকাতে।

* চ্যাট অ্যাপ (WhatsApp, Messenger) → ইউজার যেন একসাথে হাজার মেসেজ না পাঠাতে পারে।

* Payment API → ফ্রড এড়াতে দ্রুত একাধিক ট্রানজেকশন ব্লক করা।

* Public API (GitHub, Twitter) → free quota কন্ট্রোল করা।

# ইন্টারভিউ প্রশ্ন

### বেসিক

1. **Rate Limiter কী এবং কেন দরকার?**  
    Rate Limiter হলো এমন একটি কৌশল, যা নির্দিষ্ট সময়ের মধ্যে একজন ইউজার বা সিস্টেম কতগুলো রিকোয়েস্ট পাঠাতে পারবে তা নিয়ন্ত্রণ করে। এটি অপব্যবহার রোধ, ফেয়ারনেস নিশ্চিতকরণ এবং সার্ভারকে অতিরিক্ত লোড থেকে রক্ষা করতে ব্যবহৃত হয়।

2. **Token Bucket আর Leaky Bucket এর মধ্যে পার্থক্য কী?**  
    - **Token Bucket:** নির্দিষ্ট হারে টোকেন যোগ হয়, প্রতিটি রিকোয়েস্টে ১টি টোকেন খরচ হয়। টোকেন থাকলে বার্স্ট রিকোয়েস্টও অনুমোদন করা যায়।  
    - **Leaky Bucket:** রিকোয়েস্টগুলো একটি কিউতে জমা হয় এবং নির্দিষ্ট হারে বের হয়। এটি ইনকামিং স্পাইক স্মুথ করে দেয়, কিন্তু বার্স্ট হ্যান্ডলিং কম।

3. **Rate Limit ছাড়ালে কোন HTTP status ফেরত দেওয়া হয়?**  
    সাধারণত 429 Too Many Requests HTTP status code ফেরত দেওয়া হয়, যা নির্দেশ করে ইউজার নির্ধারিত সীমা অতিক্রম করেছে।

### 4. Distributed environment এ কিভাবে Rate Limiting করবে?

Distributed environment-এ rate limiting করতে হলে, সব instance-এ একই লিমিট enforce করতে হয়। এজন্য সাধারণত Redis, Memcached, বা অন্য কোনো centralized fast datastore ব্যবহার করা হয়। এতে সব instance একই ডেটা শেয়ার করে এবং সঠিকভাবে লিমিটিং হয়।

**উদাহরণ (NestJS + Redis):**

```typescript
// npm install rate-limiter-flexible ioredis
import { RateLimiterRedis } from 'rate-limiter-flexible';
import * as Redis from 'ioredis';

const redisClient = new Redis();

const rateLimiter = new RateLimiterRedis({
    storeClient: redisClient,
    points: 100, // ১০০ রিকোয়েস্ট
    duration: 60, // প্রতি ৬০ সেকেন্ডে
});

async function rateLimitMiddleware(req, res, next) {
    try {
        await rateLimiter.consume(req.ip);
        next();
    } catch {
        res.status(429).send('Too Many Requests');
    }
}
```

---

### 5. Fixed Window vs Sliding Window এর সুবিধা/অসুবিধা কী?

| Fixed Window | Sliding Window |
|--------------|---------------|
| সহজ ইমপ্লিমেন্টেশন | একটু জটিল ইমপ্লিমেন্টেশন |
| মিনিটের শেষে ও শুরুতে burst হতে পারে | burst কম হয়, smoother |
| কম মেমরি লাগে | বেশি মেমরি লাগে (log হলে) |
| উদাহরণ: প্রতি মিনিটে ১০০ রিকোয়েস্ট | প্রতি ৬০ সেকেন্ডে ১০০ রিকোয়েস্ট, যেকোনো সময় থেকে গণনা |

---

### 6. কিভাবে Free vs Premium ইউজারের জন্য আলাদা লিমিট সেট করবে?

ইউজারের টাইপ (free/premium) দেখে আলাদা rate limiter কনফিগার করা যায়।

**উদাহরণ (NestJS):**

```typescript
const freeLimiter = new RateLimiterRedis({ storeClient: redisClient, points: 100, duration: 60 });
const premiumLimiter = new RateLimiterRedis({ storeClient: redisClient, points: 1000, duration: 60 });

async function rateLimitMiddleware(req, res, next) {
    const userType = req.user.type; // ধরো user object আছে
    const limiter = userType === 'premium' ? premiumLimiter : freeLimiter;
    try {
        await limiter.consume(req.ip);
        next();
    } catch {
        res.status(429).send('Too Many Requests');
    }
}
```

---

### 7. হাজারো microservices instance এ Rate Limiting ডিজাইন করবে কিভাবে?

- Centralized datastore (Redis) ব্যবহার করে সব instance-এ একই লিমিট enforce করা।
- API Gateway-এ rate limiting implement করা যায়, যাতে backend instance-এ চাপ কমে।
- Distributed rate limiter library (যেমন rate-limiter-flexible) ব্যবহার করা।

---

### 8. একাধিক API Gateway থাকলে ফেয়ার লিমিট কিভাবে নিশ্চিত করবে?

- সব gateway-কে একই centralized datastore (Redis) ব্যবহার করাতে হবে।
- তাহলে যেকোনো gateway-তে রিকোয়েস্ট আসুক, একই লিমিট শেয়ার হবে।
- Redis cluster/scaling ব্যবহার করে performance বাড়ানো যায়।

---

### 9. একসাথে per-IP + per-user + global limit কিভাবে ম্যানেজ করবে?

- আলাদা আলাদা limiter কনফিগার করে, প্রতিটা রিকোয়েস্টে সবগুলো চেক করতে হবে।
- উদাহরণ: প্রথমে per-user, তারপর per-IP, তারপর global limiter চেক করা।

**উদাহরণ (NestJS):**

```typescript
async function rateLimitMiddleware(req, res, next) {
    try {
        await Promise.all([
            userLimiter.consume(req.user.id),
            ipLimiter.consume(req.ip),
            globalLimiter.consume('global'),
        ]);
        next();
    } catch {
        res.status(429).send('Too Many Requests');
    }
}
```

### Rate Limiter Guard তৈরি করা

```typescript
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  TooManyRequestsException,
  Inject,
} from '@nestjs/common';
import Redis from 'ioredis';

@Injectable()
export class RateLimitGuard implements CanActivate {
  constructor(@Inject('REDIS_CLIENT') private readonly redis: Redis) {}

  private limit = 10;       // প্রতি মিনিটে ১০টা রিকোয়েস্ট
  private ttl = 60;         // ৬০ সেকেন্ডের জন্য উইন্ডো

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const userIp = request.ip || 'global';

    const key = `rate-limit:${userIp}`;

    // Redis এ কাউন্টার বাড়ানো
    const current = await this.redis.incr(key);

    if (current === 1) {
      // প্রথমবার সেট করলে TTL বসানো
      await this.redis.expire(key, this.ttl);
    }

    if (current > this.limit) {
      throw new TooManyRequestsException('Too Many Requests');
    }

    return true;
  }
}
```