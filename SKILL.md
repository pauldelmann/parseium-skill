---
name: parseium-skill
description: Fetch structured data from social media and web platforms via Parseium APIs. Use when users want to extract profiles, posts, videos, comments, or other data from Instagram, TikTok, YouTube, Reddit, LinkedIn, Zillow, Indeed, or App Store.
license: MIT
---

# Parseium API Guide for AI Agents

Parseium provides pre-built APIs for extracting structured data from social media and web platforms. This guide explains how to discover, understand, and use these APIs.

**Base URL:** `https://api.parseium.com`

## Getting an API Key

An API key is required to use Parseium APIs. Users can obtain one at:

**https://www.parseium.com/api-key**

If the user hasn't provided an API key, prompt them to get one from the link above before making any data requests. The discovery endpoints (`/v1/apis`, `/v1/apis/{id}`) work without a key, so you can still help users explore available APIs while they set up their account.

## Quick Start

1. Get API catalog: `GET /v1/apis`
2. Get specific API docs: `GET /v1/apis/{id}`
3. Check credits: `GET /credits`
4. Call any API: `GET|POST /v1/{api-id}?api_key=YOUR_KEY&param=value`

## Authentication

All data-fetching endpoints require an API key. Pass it via:
- Header: `X-API-Key: your_api_key`
- Query param: `?api_key=your_api_key`

## Discovery Endpoints (No Auth Required)

### List All APIs
```
GET https://api.parseium.com/v1/apis
```

Returns JSON array of available APIs:
```json
[
  {
    "id": "instagram-profile",
    "name": "Instagram Profile",
    "url": "/v1/instagram-profile",
    "description": "Extract profile metadata, followers, posts from Instagram",
    "credits": 1
  },
  ...
]
```

### Get API Documentation
```
GET https://api.parseium.com/v1/apis/{id}
```

Returns markdown documentation with:
- Endpoint description
- Parameters (name, type, required, description)
- Request format (headers, query string, JSON body)
- Response schema (type-annotated JSON structure)
- Error codes

Example:
```
GET https://api.parseium.com/v1/apis/instagram-profile
```

## Account Endpoint

### Check Credits & Limits
```
GET https://api.parseium.com/credits
X-API-Key: your_api_key
```

Response:
```json
{
  "creditBalance": 1000,
  "concurrency": 5
}
```

- `creditBalance`: Remaining credits for API calls
- `concurrency`: Max simultaneous requests allowed

## Available APIs

| Platform | Endpoints |
|----------|-----------|
| Instagram | instagram-profile, instagram-post, instagram-posts, instagram-reels |
| TikTok | tiktok-profile, tiktok-video, tiktok-comments |
| YouTube | youtube-channel, youtube-video, youtube-transcript, youtube-comments, youtube-channel-videos, youtube-channel-shorts, youtube-search |
| Reddit | reddit-post, reddit-user |
| LinkedIn | linkedin-profile, linkedin-company, linkedin-post |
| Zillow | zillow-search, zillow-details |
| Other | indeed-job, app-store, app-store-search |

All endpoints cost 1 credit per request.

## Making Requests

### GET Request
```
GET https://api.parseium.com/v1/instagram-profile?api_key=YOUR_KEY&username=nasa
```

### POST Request
```
POST https://api.parseium.com/v1/instagram-profile
X-API-Key: YOUR_KEY
Content-Type: application/json

{"username": "nasa"}
```

## Response Headers

Successful responses include:
- `X-Credits-Used`: Credits consumed by this request
- `X-Credits-Remaining`: Remaining credit balance

## Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad request - missing/invalid parameters |
| 401 | Unauthorized - invalid/missing API key |
| 402 | Payment required - insufficient credits |
| 404 | Not found - resource doesn't exist |
| 429 | Too many requests - rate/concurrency limit exceeded |
| 502 | Bad gateway - upstream service error |

## Pagination

Many endpoints support pagination via `cursor`, `continuation`, or `next_url` parameters. Check the specific API docs for details:
```
GET https://api.parseium.com/v1/apis/{id}
```

## Workflow for AI Agents

1. **Discover**: Call `/v1/apis` to see available endpoints
2. **Learn**: Call `/v1/apis/{id}` for specific endpoint docs
3. **Check**: Call `/credits` to verify sufficient balance
4. **Execute**: Call the target API with required parameters
5. **Paginate**: Use returned cursor/continuation for more results

## Example: Fetch Instagram Profile

```bash
# 1. Check available credits
curl -H "X-API-Key: YOUR_KEY" https://api.parseium.com/credits

# 2. Fetch profile
curl -H "X-API-Key: YOUR_KEY" "https://api.parseium.com/v1/instagram-profile?username=nasa"
```

Response includes identity, metrics, account type, business info, recent posts, and pagination cursor.

## Rate Limits

- Discovery endpoints (`/v1/apis`, `/v1/apis/{id}`): 30 requests/minute per IP
- Data endpoints: Based on your plan's concurrency limit (check `/credits`)

## Common Use Cases

| Task | Endpoint | Key Parameter |
|------|----------|---------------|
| Get someone's follower count | instagram-profile, tiktok-profile, youtube-channel | username/handle |
| Extract video transcript for summarization | youtube-transcript | video_id or url |
| Get post engagement metrics | instagram-post, tiktok-video, reddit-post | url |
| Research a company | linkedin-company | url or slug |
| Search for properties | zillow-search | query (city/zip) |
| Get job posting details | indeed-job | job_key |
| Find apps | app-store-search | query |

For detailed parameters and response schemas, always fetch `/v1/apis/{id}`.

## Chaining Endpoints

Some workflows require multiple API calls:

**Instagram pagination:**
1. Call `instagram-profile?username=X` → returns `user_id` in identity
2. Call `instagram-posts?user_id=Y&cursor=Z` → use user_id for pagination

**YouTube deep dive:**
1. Call `youtube-channel?handle=X` → get channel_id
2. Call `youtube-channel-videos?channel_id=Y` → list videos
3. Call `youtube-video?video_id=Z` → get specific video details
4. Call `youtube-transcript?video_id=Z` → get transcript for summarization

**Zillow property research:**
1. Call `zillow-search?query=Austin TX` → get listing URLs
2. Call `zillow-details?url=Y` → get full property details

## Error Handling

| Error | Action |
|-------|--------|
| 401 Unauthorized | API key missing/invalid. Ask user to provide key or get one at https://www.parseium.com/api-key |
| 402 Payment Required | Insufficient credits. Ask user to add credits at https://www.parseium.com |
| 404 Not Found | Resource doesn't exist (deleted account, private profile, etc). Inform user. |
| 429 Too Many Requests | Rate/concurrency limit hit. Wait a few seconds and retry, or reduce parallel requests. |
| 502 Bad Gateway | Upstream platform issue. Retry once after brief delay. If persistent, the platform may be blocking. |

## Best Practices

1. **Check credits first** - Call `/credits` before batch operations to ensure sufficient balance
2. **Handle pagination** - Many endpoints return partial results; use cursor/continuation tokens for complete data
3. **Respect concurrency** - Don't exceed your concurrency limit with parallel requests
4. **Cache when appropriate** - Profile data doesn't change frequently; avoid redundant calls
5. **Use POST for complex params** - Easier to structure JSON body than encode complex query strings

## Response Format

All data endpoints return JSON. The `/v1/apis/{id}` endpoint returns markdown documentation. Successful responses include `X-Credits-Used` and `X-Credits-Remaining` headers for tracking usage.

## Timeouts

Some endpoints may take 5-15 seconds due to upstream fetching. This is normal for:
- First request to a new profile (no cache)
- Platforms with anti-bot measures (LinkedIn, Instagram)
- Transcript extraction (YouTube)

Don't timeout requests prematurely. If building user-facing features, show a loading state.
