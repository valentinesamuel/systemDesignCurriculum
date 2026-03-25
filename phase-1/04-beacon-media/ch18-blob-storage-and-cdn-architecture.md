---
**Title**: Beacon Media — Chapter 18: Fix the CDN — $2M/Month and Falling
**Level**: Senior
**Difficulty**: 7
**Tags**: #cdn #storage #performance #cost-optimization #distributed
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 19, Ch. 20
**Exercise Type**: Performance Optimization
---

### Story Context

**Grafana dashboard review — Day 1, 2:00 PM**

Fatima pulls up the CDN metrics. The numbers are not good.

```
CDN Cache Hit Rate:       54.2% (target: 92%)
CDN Egress Cost:          $68,420/day ($2.05M/month)
Origin Pull Requests:     2.3M/day (every cache miss hits your origin)
Avg Video Start Time:     4.2 seconds (target: < 1.5s)
Buffering Rate:           8.4% of sessions (target: < 2%)
Content Types:
  - VOD (video on demand): 60% of requests
  - Live streams:           25% of requests
  - Thumbnails/images:      12% of requests
  - API responses:           3% of requests
```

**Fatima**: A 54% hit rate means nearly half of all our CDN requests are going
back to origin. Every origin pull costs us money. Origin is in AWS S3 and our
transcoding servers. At 2.3 million origin pulls per day, we're hammering our
own infrastructure.

**You**: What's the CDN vendor?

**Fatima**: Cloudfront. But we're re-evaluating. The contract is up in 4 months.

**You**: Any idea why the hit rate is so low? Have you looked at the miss reasons?

**Fatima**: We have some data. Mostly it's cache misses on new content in the first
hour after upload, and also something weird with the URL structure.

---

**CDN miss analysis (you pull the logs Wednesday morning)**

After a few hours of log analysis, you identify three root causes:

**Cause 1: URL uniqueness explosion (41% of misses)**
The video player generates URLs with query parameters:
```
https://cdn.beaconmedia.io/v/movie-123/720p.m3u8?quality=auto&bitrate=2500&session=a7f3b2c1&ts=1704067200
```
The `session` and `ts` (timestamp) query parameters make every URL unique.
Even if 10,000 users are watching the same video, each gets a unique cache key.
The CDN never caches anything with these parameters.

**Cause 2: Missing cache headers on transcoded video (33% of misses)**
Video segments (`.ts` files) have `Cache-Control: no-cache` set by the transcoding
pipeline. Someone added this during debugging 18 months ago and it was never removed.

**Cause 3: Short TTL on thumbnail images (26% of misses)**
Thumbnail images have a 60-second TTL. They're regenerated every 60 seconds even
though they rarely change (maybe weekly when a show updates its artwork).

---

**Slack DM — Marcus Webb → You, Wednesday afternoon**

**Marcus Webb**
CDN hit rate at 54%. The session parameter URL problem is the funniest and
saddest thing I've seen today. Someone made every URL globally unique and then
wondered why the cache never works.

Two questions before you fix it:
1. Those session parameters — are they used for anything? Analytics? DRM? If
   you strip them from the cache key, you need to make sure you're not breaking
   something downstream.
2. Presigned URLs. You'll need them for premium content. How do you cache a
   presigned URL? (Hint: you don't. You separate the auth check from the content delivery.)

---

**Cost breakdown from finance (Fatima forwards it Thursday)**

```
CDN egress:              $1,850,000/month
S3 storage:                $124,000/month
S3 requests (API):          $38,000/month
Transcoding (EC2):         $210,000/month
Total infrastructure:    $2,222,000/month

Board target: reduce total to $1,400,000/month within 6 months
Engineering has been asked to find $822,000/month in savings.
CDN optimization is the primary lever.
```

---

### Problem Statement

Beacon Media is spending $2M/month on CDN egress with a 54% cache hit rate.
Three root causes have been identified: session parameters in URLs creating
unique cache keys, missing cache headers on video segments, and aggressive TTL
on thumbnail images. Additionally, the CDN architecture doesn't yet support
presigned URL authentication for premium content. You must redesign the CDN
and storage architecture to achieve 92%+ cache hit rate while implementing
secure content delivery.

### Explicit Requirements

1. Increase CDN cache hit rate from 54% to 92%+
2. Fix URL structure: strip or normalize session/timestamp query parameters
   from cache keys without breaking downstream analytics or DRM systems
3. Set correct cache headers on video segments (`.ts` files): at least 7 days
   (video segments are immutable — once transcoded, they never change)
4. Set appropriate TTL on thumbnail images: 24 hours with cache invalidation
   on artwork update
5. Implement presigned URL architecture for premium content access
6. Achieve at least $600,000/month reduction in CDN egress cost

### Hidden Requirements

- **Hint**: Marcus Webb asked whether session parameters are used for anything.
  If `session` is used for analytics (tracking who watched what), you cannot
  simply remove it from the URL. You need a different approach: strip it from
  the *cache key* but still pass it to origin when needed. CDNs support this
  natively. What is the CDN configuration change?
- **Hint**: Presigned URLs are time-limited signed URLs that grant access to
  private S3 content. The problem is: if you put a presigned URL through a CDN,
  the signature (which changes every time) becomes part of the cache key, making
  it uncacheable. The standard pattern separates the auth token from the content URL.
  How does this work? (Hint: CDN viewer request → Lambda@Edge/CloudFront Function
  → validate token → forward to origin without token)
- **Hint**: Video segments (`.ts` files) are *immutable*. Once a video is transcoded,
  the `.ts` files never change. What cache TTL is appropriate for immutable assets?
  And what happens to the cache when you re-transcode a video at higher quality?
  (Hint: URL versioning / content hash in filename)

### Constraints

- **CDN**: AWS CloudFront (contract up in 4 months — architecture should be
  portable to Fastly/Akamai if the contract switches)
- **Storage**: AWS S3, ~180TB total video storage
- **Content library**: 50,000 video titles, average 4GB each (multi-bitrate)
- **New content uploads**: ~200 titles/week
- **CDN nodes**: CloudFront edge locations in Singapore, Jakarta, Nairobi, Lagos,
  Cape Town, Mumbai (primary markets)
- **Budget target**: $600k/month CDN cost reduction minimum

### Your Task

Redesign Beacon Media's CDN and blob storage architecture to achieve 92%+ cache
hit rate and implement secure presigned URL content delivery.

### Deliverables

- [ ] **CDN architecture diagram** (Mermaid) — show: S3 origin → CloudFront
  distribution → edge cache → end user; and the presigned URL auth flow
- [ ] **Cache configuration specification** — for each content type (video segments,
  HLS manifests, thumbnails, API responses), specify: cache TTL, cache key
  normalization rules (which query params to strip), invalidation strategy
- [ ] **Presigned URL authentication design** — how do you protect premium content
  without making the CDN URLs uncacheable? Show the token validation flow.
- [ ] **Cost model** — at 92% cache hit rate, what is new daily origin pull
  count? At the same egress price per GB, what is new monthly CDN cost?
  Show the math.
- [ ] **Content versioning strategy** — for immutable video segments, how do
  you handle re-transcoding (e.g., upgrading quality)? URL scheme that forces
  cache bust without invalidating the entire distribution.
- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. CloudFront signed URLs vs token-based auth (Lambda@Edge)
  2. Per-title CDN invalidation vs cache TTL expiry for thumbnail updates
  3. Single CDN vendor (CloudFront) vs multi-CDN strategy for geographic coverage

### Diagram Format

```mermaid
graph LR
  U[User] --> CF[CloudFront Edge]
  CF -->|cache miss| S3[(S3 Origin)]
  CF -->|cache hit| U
  subgraph Auth ["Premium Content Auth"]
    CF --> LE[Lambda@Edge\nToken Validation]
    LE --> S3
  end
```
