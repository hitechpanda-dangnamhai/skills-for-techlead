# Mục J — Caching · Toàn bộ 77 đề bài nói
### Gom từ 11 bài · 25 đề dạng phản biện được đánh dấu `[phản biện]`

> **Luật chơi cho mỗi đề:** bật ghi âm · nói **60–90 giây** · **không nhìn tài liệu** · dùng ít nhất **sáu thuật ngữ** trong bảng tra. Nghe lại một lần, đánh dấu chỗ ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa. Lần hai luôn tốt hơn lần một, và chính lần hai mới là thứ bạn mang vào phòng phỏng vấn.

> **Vì sao đánh dấu phản biện:** những đề đó buộc bạn phải *không đồng ý* bằng tiếng Anh mà vẫn giữ được lịch sự và giữ được lập luận. Đây là dạng câu phân biệt senior với mid-level, và cũng là dạng người Việt yếu nhất vì nó cần cấu trúc nhượng bộ rồi phản bác. Nếu thời gian có hạn, ưu tiên 25 đề này.

---

## Lịch chạy gợi ý — bốn tuần

**Tuần 1 — nền tảng.** Bài 1 đến Bài 3, mỗi ngày ba đề. Mục tiêu là nói trôi phần *tại sao*, chưa cần bảo vệ quan điểm.

**Tuần 2 — chiều sâu.** Bài 4 đến Bài 6, mỗi ngày ba đề. Bắt đầu bấm giờ nghiêm, cắt đúng chín mươi giây kể cả khi chưa nói xong.

**Tuần 3 — Redis và mở rộng.** Bài 7 đến Bài 9, mỗi ngày ba đề, kèm hai đề cũ đã đạt để ôn ngắt quãng.

**Tuần 4 — mock thật.** Bài 10 và Bài 11, sau đó chạy liên tục 25 đề phản biện trong hai buổi, mỗi buổi khoảng bốn mươi phút không nghỉ giữa các câu.

**Tiêu chí đạt cho một đề:** nói hết trong 90 giây, không quay lại sửa câu quá hai lần, dùng đủ sáu thuật ngữ, và với đề phản biện thì phải có đủ ba phần — thừa nhận điều đối phương nói đúng, nêu chỗ sai, đề xuất phương án thay thế.

---


## Bài 1 — Vì sao cache · các tầng cache · khi nào KHÔNG cache
*3 đề phản biện trong 7 đề*

1. **Explain to a junior** why a cache is a bet on locality, and what exactly we pay in exchange for that bet.

2. **Describe what happens** when a request travels from the browser down to the database. Name the six cache layers in order and say what each one trades away.

3. `[phản biện]` **A colleague says:** *"Our hit ratio is ninety-nine percent, so our cache is clearly working well."* Explain what is wrong with that claim and what you would measure instead.

4. `[phản biện]` **Someone on your team wants** to cache the stock level of a product so that the product page loads faster. Explain why you would push back, and describe what you would do instead.

5. `[phản biện]` **A teammate proposes** raising the TTL of the L1 in-process cache from five seconds to ten minutes, because it improves latency in the load test. Explain why you would push back, and what failure you expect in production.

6. **When would you choose** a materialized view over a cache, and when would you choose the opposite? Describe the cost of each direction.

7. **Your team added Redis** in front of the database, but p99 latency did not improve at all. Describe how you would investigate, and give three possible reasons.



## Bài 2 — Các chiến lược cache
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** the read flow and the write flow of cache-aside, in your own words, and say why we delete the key instead of updating it.

2. **A colleague says:** *"Read-through and cache-aside are basically the same thing, so it does not matter which one we pick."* Explain what is actually different between them and when that difference matters.

3. `[phản biện]` **Someone on your team wants** to use write-back for the payment path, because it makes writes much faster in the load test. Explain why you would push back, and describe exactly what you would do instead.

4. `[phản biện]` **A teammate claims** that write-through gives absolutely strong consistency, so the cache can never be stale. Explain what is wrong with that claim.

5. **Explain why cache-aside** is the industrial default, even though it has a cold start and a stale window. Give three reasons and be ready to defend each one.

6. **Describe what happens** to your system in the first minute after a deploy, and explain what cache warming can and cannot do about it.

7. **When would you cache** a raw row, and when would you cache a rendered fragment? Give the trade-off in both directions and finish with a concrete example.



## Bài 3 — Invalidation · TTL · thiết kế key · negative caching
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** why cache invalidation is famously hard, using the difference between a single record and derived data.

2. `[phản biện]` **A colleague wants** to remove all TTLs from the cache, because the team already invalidates explicitly on every write. Explain why you would push back and what you would keep.

3. **You are reviewing a pull request** and the cache key is simply the name of the function. Explain to the author what is missing, what can go wrong, and what the key should contain.

4. `[phản biện]` **Someone on your team proposes** deleting every affected key one by one whenever the data format changes. Explain why you would push back, and describe the approach you would use instead.

5. **Describe what happens** when a nightly batch job writes straight into the database while your invalidation lives in the application code. Then explain how you would fix it properly.

6. **Explain what negative caching is**, why we need it, and the one risk it brings with it. Say exactly how you control that risk.

7. **When would you use** stale-while-revalidate, and when would you refuse to use it? Give one example on each side.



## Bài 4 — Tính nhất quán giữa cache và DB
*3 đề phản biện trong 7 đề*

1. **Explain to a junior** why a cache and a database can never be perfectly consistent for free, and what our realistic goal is instead.

2. `[phản biện]` **A colleague's pull request** updates the cache with the new value on every write, because "it saves one database read later". Explain why you would push back.

3. **Walk an interviewer through** both orderings — delete the cache then write the database, and write the database then delete the cache. Describe the race in each one and say which you would ship.

4. `[phản biện]` **A teammate says** the delayed double delete with a one-millisecond delay closes the race completely. Explain what is wrong with that, and what you would use when you really need certainty.

5. `[phản biện]` **A colleague claims** that write-through gives perfect consistency, so it is safe to cache the account balance. Explain why you would push back, and describe what you would do for that path instead.

6. **Describe read-your-own-writes**: how caching breaks it, how the user experiences the bug, and two ways to preserve the guarantee.

7. **Explain how CDC or the outbox pattern** keeps a cache in sync, and why it is more reliable than deleting the cache inside the application code.



## Bài 5 — Penetration · Breakdown · Avalanche · hot key và big key
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** the difference between penetration, stampede and avalanche, giving the cause and the consequence of each one.

2. `[phản biện]` **A colleague says** that negative caching on its own is enough to stop penetration, so a Bloom filter is over-engineering. Explain why you would push back.

3. **Describe how single-flight works** when a hot key expires, and be honest about the two limits it brings with it.

4. **Someone on your team proposes** putting a per-key mutex on everything to protect the database. Explain why that will not save you when Redis restarts, and what you would add instead.

5. **Explain what TTL jitter is**, why a batch of keys with one fixed TTL is dangerous, and how much randomness you would add.

6. **Describe what is happening** when one Redis node is at ninety percent CPU while the others sit idle and your hit ratio is one hundred percent. Give three ways to reduce it and the price of each.

7. `[phản biện]` **A teammate wants** to store the whole product catalogue in one Redis key, to save network round trips. Explain why you would push back, and describe what you would do instead.



## Bài 6 — Eviction và bộ nhớ
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** the difference between expiration and eviction, and why a key with a one-hour TTL can disappear after ten minutes.

2. `[phản biện]` **A colleague says:** *"I set a one-hour TTL, so the value is guaranteed to be there for an hour."* Explain why you would push back, and what their code has to handle instead.

3. **Explain when you would choose LFU over LRU**, using a crawler sweeping every key as your example.

4. `[phản biện]` **Someone on your team wants** one Redis instance for the cache, the sessions and the job queue, all on allkeys-lru, because it is cheaper. Explain why you would push back and what you would set up instead.

5. **Your hit ratio dropped sharply** right after eviction started. Describe how you would diagnose it, which two metrics you would look at, and the options you would weigh.

6. **Explain why one gigabyte of data** takes more than one gigabyte of RAM, and say what maxmemory you would set on a machine and why.

7. **A teammate configured** the cache instance with noeviction so that nothing would ever be lost, and now the app gets out-of-memory errors on writes. Explain what happened and how you would fix it.



## Bài 7 — Redis làm cache: bản chất bên trong
*3 đề phản biện trong 7 đề*

1. **Explain to a junior** why Redis is fast even though it executes commands on a single thread, and what part of the system can still be slow.

2. **A colleague ran the command** that lists every key on production to find some keys, and latency spiked across the whole service. Explain what happened, why the dashboard still showed Redis as healthy, and what they should use instead.

3. `[phản biện]` **A teammate says** that MULTI and EXEC work like a SQL transaction, so a failed command will roll the whole block back. Explain why you would push back and what you would use for conditional atomic logic.

4. **When would you still choose** Memcached over Redis today? Answer in terms of conditions rather than preference.

5. `[phản biện]` **Someone on your team wants** to fetch ten thousand members and sort them in the application to build a top-ten leaderboard. Explain why you would push back and what you would do instead.

6. **Explain the difference** between RDB and AOF, and say what you would configure for a pure cache instance versus a session store.

7. `[phản biện]` **You are reviewing AI-generated caching code** that uses the same key for every user. Explain what is wrong, what the consequence is, and the four things you check in any Redis review.



## Bài 8 — Redis ngoài vai trò cache
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** why one Redis instance can serve so many different purposes, and what the common caveat is across all of them.

2. **Explain why a sorted set** is the right structure for a realtime leaderboard, and then raise the follow-up risk yourself before the interviewer does.

3. `[phản biện]` **A colleague wants** to keep sessions in each instance's own memory and turn on sticky sessions at the load balancer. Explain why you would push back and what you would propose instead.

4. **Describe how you would build** a rate limiter at the storage level, and explain exactly what breaks if each instance counts on its own.

5. `[phản biện]` **A teammate says** they will write a simple distributed lock with set-if-not-exists and delete this afternoon. Explain why you would push back, naming two concrete risks.

6. **When would you use HyperLogLog** instead of an ordinary set, and when would you refuse to use it? Give one example on each side.

7. **Users are being logged out at random** and everything shares one Redis with an evict-all-keys policy. Walk through your diagnosis and the fix you would ship.



## Bài 9 — Mở rộng Redis: replica · cluster · sentinel
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** when one Redis node stops being enough, and the directions available to scale. Say what you would measure before choosing.

2. **Describe how Redis Cluster** maps a key to a node, and explain why the slot layer exists at all instead of hashing straight onto the node count.

3. `[phản biện]` **A colleague hit a cross-slot error** and wants to put one shared hash tag on every key so that the error goes away. Explain why you would push back and what you would do instead.

4. **Explain the difference** between Sentinel and Cluster by naming the one question each of them answers.

5. `[phản biện]` **A teammate says** that because Redis has replication, the replica always has the data, so a distributed lock stays safe across a failover. Explain what is wrong with that claim.

6. **When would you use RediSearch**, and when would you insist on a dedicated search engine? Answer in terms of a spectrum rather than a winner.

7. **You are reviewing code** for a service that is about to move onto Redis Cluster. Describe what you check, and what you would tell the author about multi-key commands.



## Bài 10 — Công việc chạy nền: BullMQ và Redis Streams
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** the three classes of problem a job queue solves, giving one concrete example for each.

2. **Explain the difference** between a list-based queue and Redis Streams, and say exactly what a consumer group gives you that a list cannot.

3. `[phản biện]` **A teammate wrote a consumer** that assumes each job runs exactly once, because the queue library "handles that". Explain why you would push back and what you would ask them to add.

4. **Describe what happens** when a worker dies in the middle of a job, and how the system gets that job back to another worker.

5. `[phản biện]` **Someone on your team wants** unlimited retries so that no job is ever lost. Explain why you would push back and what you would configure instead.

6. **When is BullMQ not enough** and you would move to Kafka? Include the price of that move in your answer.

7. **A colleague put the queue** on the same Redis as the cache with an evict-all-keys policy, to save on cost. Explain the two opposite risks and what you would set up instead.



## Bài 11 — Cache ở tầng HTTP và CDN
*2 đề phản biện trong 7 đề*

1. **Explain to a junior** the difference between max-age, s-maxage, private and public, giving one example of content for each.

2. `[phản biện]` **A colleague set no-cache** on the payment page, believing that nothing would then be stored anywhere. Explain why you would push back and what they should set instead.

3. **Explain how an ETag** and revalidation work, and say when they beat simply setting a long max-age.

4. **Explain what a CDN solves** that Redis cannot, by naming two different kinds of latency.

5. `[phản biện]` **Someone on your team wants** to purge the CDN on every deploy so that new assets go live. Explain why you would push back and what you would do instead.

6. **Why is GraphQL over POST** harder to cache than a REST GET endpoint, and how would you get that caching back?

7. **Users report seeing each other's profile data**, and the endpoint is marked public with a max-age of three hundred seconds. Walk through the diagnosis, the fix, and then list the five caching traps you check in any review.



---

## Ba mẫu câu dùng riêng cho đề phản biện

Với dạng *"a colleague says X, explain what is wrong"*, câu trả lời tốt luôn có ba phần theo đúng thứ tự này.

**Phần một — thừa nhận phần đúng.** *"I can see why that looks attractive — it does reduce one round trip."* Nếu bỏ phần này, bạn nghe như đang bác bỏ người khác chứ không phải đang phân tích.

**Phần hai — nêu chỗ sai kèm cơ chế.** *"The problem is that two writers can reach the cache in the reverse order, so the old value ends up overwriting the new one."* Luôn nói cơ chế, đừng chỉ nói rằng nó sai.

**Phần ba — đưa phương án và cái giá của nó.** *"I would delete the key instead. We pay for one extra read on the next request, but we remove the class of bug entirely."* Nêu cả cái giá cho thấy bạn không bán một giải pháp hoàn hảo.

---

*Tổng hợp từ 11 bài song ngữ của mục J — Caching.*
