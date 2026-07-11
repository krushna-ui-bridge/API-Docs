# EJET AI — Student Dashboard API Specification

**Base URL:** `https://api.ejetai.com/v1`
**Auth:** Bearer token (JWT) in `Authorization` header for all endpoints
**Content-Type:** `application/json`

---

## 1. Common / Shared

### 1.1 GET `/user/profile`
Used by the top-right avatar + semester selector on every page.

**Query params**
| Param | Type | Required | Description |
|---|---|---|---|
| — | — | — | none |

**Response 200**
```json
{
  "userId": "u_10293",
  "name": "Krish Malhotra",
  "avatarUrl": "https://cdn.ejetai.com/avatars/u_10293.jpg",
  "currentSemester": {
    "id": "sem_2024_1",
    "label": "Current Semester",
    "startDate": "2024-01-08",
    "endDate": "2024-05-17"
  },
  "availableSemesters": [
    { "id": "sem_2023_2", "label": "Fall 2023", "startDate": "2023-08-14", "endDate": "2023-12-15" },
    { "id": "sem_2024_1", "label": "Spring 2024", "startDate": "2024-01-08", "endDate": "2024-05-17" }
  ]
}
```

### 1.2 GET `/semesters/{semesterId}`
Called when the "Current Semester" dropdown is changed; every widget below refetches with this `semesterId` as a query param.

---

## 2. Dashboard Page (`/dashboard`)

### 2.1 GET `/dashboard/summary`

Powers the hero banner, aggregate score ring, and greeting text.

**Query params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Response 200**
```json
{
  "aggregateScore": {
    "value": 92.4,
    "label": "AGGREGATE SCORE",
    "scale": 100
  }
}
```

---

### 2.2 GET `/dashboard/placement-eligibility`

Powers the four "Placement Eligibility" cards (Marquee, Super Dream, Dream, Regular).

**Query params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Response 200**
```json
{
  "categories": [
    {
      "id": "marquee",
      "label": "Marquee",
      "icon": "star",
      "status": "ELIGIBLE",
      "reason": null
    },
    {
      "id": "super_dream",
      "label": "Super Dream",
      "icon": "trending_up",
      "status": "ELIGIBLE",
      "reason": null
    },
    {
      "id": "dream",
      "label": "Dream",
      "icon": "heart",
      "status": "ELIGIBLE",
      "reason": null
    },
    {
      "id": "regular",
      "label": "Regular",
      "icon": "warning",
      "status": "NOT_ELIGIBLE",
      "reason": "Minimum CGPA threshold not met"
    }
  ]
}
```

`status` enum: `ELIGIBLE`, `NOT_ELIGIBLE`, `PENDING_REVIEW`

---

### 2.3 GET `/dashboard/intellect-metrics`

Powers the four metric cards: Coding Hours, Attendance, Reviews Taken, Practice Subs. Each card links to a "View Details" page (see section 3 for the Coding Hours detail page).

**Query params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Response 200**
```json
{
  "metrics": [
    {
      "id": "coding_hours",
      "label": "CODING HOURS",
      "value": 240,
      "unit": "hrs"
    },
    {
      "id": "attendance",
      "label": "ATTENDANCE",
      "value": 92,
      "unit": "%",
      "trend": { "direction": null, "value": "Perfect streak: 14 days", "comparedTo": null },
      "detailRoute": "/coding-hours"
    },
    {
      "id": "reviews_taken",
      "label": "REVIEWS TAKEN",
      "value": 12,
      "unit": "qty",
      "trend": { "direction": null, "value": "Next review in 48h", "comparedTo": null },
      "detailRoute": "/reviews"
    },
    {
      "id": "practice_subs",
      "label": "PRACTICE SUBS",
      "value": 45,
      "unit": "items",
      "trend": { "direction": null, "value": "100% Acceptance rate", "comparedTo": null },
      "detailRoute": "/practice-submissions"
    }
  ]
}
```

---

## 3. Coding Hours Page (`/coding-hours`)

This page has two views driven by the same `semesterId` param:
- **All Semesters** (`semesterId=all`) — the landing view: aggregate stat cards, a per-semester trend chart, and a Semester Breakdown table.
- **Single semester** (`semesterId={id}`, e.g. `sem_4`) — reached by clicking a row's chevron in the Semester Breakdown table; shows that semester's stat cards and its full Session History.

Both views reuse the **same `stats` array shape** from `/coding-hours/summary` so the frontend renders one card component regardless of scope — the fields differ, not the structure.

---

### 3.1 GET `/coding-hours/summary`

Powers the 4 stat cards at the top of the page. Response shape is identical whether scope is `all` or a single semester; only the `stats` contents change.

**Query params**
| Param | Type | Required | Notes |
|---|---|---|---|
| semesterId | string | no | `all` (default) or a specific semester id (e.g. `sem_4`) |

**Response 200 — `semesterId=all`** (matches "All Semesters" view)
```json
{
  "scope": "all",
  "stats": [
    {
      "id": "total_hours",
      "value": 1240,
      "unit": "hrs",
      "trend": { "direction": "up", "value": "+12%", "comparedTo": "last sem" }
    },
    {
      "id": "weekly_intensity",
      "value": "High",
      "subtext": "Avg. 42 hours/week"
    },
    {
      "id": "consistency_score",
      "value": 94,
      "unit": "%",
      "progressPercent": 94
    },
    {
      "id": "global_rank",
      "value": "Top 2%",
      "subtext": "Ranked #142 of 12k+"
    }
  ]
}
```

**Response 200 — `semesterId=sem_4`** (matches "Semester 4 Breakdown" view)
```json
{
  "scope": "sem_4",
  "stats": [
    {
      "id": "total_hours",
      "value": 380,
      "unit": "hrs",
      "trend": { "direction": "up", "value": "+12%", "comparedTo": "Semester 3" }
    },
    {
      "id": "avg_intensity",
      "value": "Ultra High",
      "progressPercent": 92
    },
    {
      "id": "consistency",
      "value": 98,
      "unit": "%",
      "progressPercent": 98,
      "segments": { "filled": 5, "total": 6 }
    },
    {
      "id": "top_topic",
      "value": "Advanced React",
      "subtext": "142 hours dedicated focus"
    }
  ]
}
```

`stats[].id` enum (card identity, drives icon + label on the frontend — the API doesn't send display labels, same convention as the rest of this spec): `total_hours`, `weekly_intensity`, `consistency_score`, `global_rank`, `avg_intensity`, `consistency`, `top_topic`.

---

### 3.2 GET `/coding-hours/trend`

Powers the "Coding Hours Trend" bar chart. Only meaningful for `semesterId=all` — not called on the single-semester view.

**Query params**
| Param | Type | Required |
|---|---|---|
| — | — | none |

**Response 200**
```json
{
  "seriesLabel": "Current Semester",
  "comparisonLabel": "Class Avg",
  "semesters": [
    { "semesterId": "sem_1", "name": "Semester 1", "hours": 250, "classAvgHours": 230, "isActive": false },
    { "semesterId": "sem_2", "name": "Semester 2", "hours": 290, "classAvgHours": 260, "isActive": false },
    { "semesterId": "sem_3", "name": "Semester 3", "hours": 320, "classAvgHours": 300, "isActive": false },
    { "semesterId": "sem_4", "name": "Semester 4", "hours": 380, "classAvgHours": 310, "isActive": true }
  ]
}
```

> Note: the API returns semesters in chronological order (`sem_1` → `sem_4`). The mockup renders the bars as Sem 2, Sem 3, Sem 1, Sem 4 — that reads as a shuffled/placeholder ordering in the design rather than an intentional layout, so flag it with design before building the frontend sort. If a non-chronological order genuinely is intended, the frontend should do that reordering, not the API.

---

### 3.3 GET `/coding-hours/semesters`

Powers the "Semester Breakdown" table on the All Semesters view.

**Query params**
| Param | Type | Required |
|---|---|---|
| — | — | none |

**Response 200**
```json
{
  "semesters": [
    {
      "semesterId": "sem_4",
      "name": "Semester 4",
      "isActive": true,
      "totalHours": 380,
      "avgIntensity": "Ultra High",
      "topDomain": "AI/ML",
      "status": "ONGOING"
    },
    {
      "semesterId": "sem_3",
      "name": "Semester 3",
      "isActive": false,
      "totalHours": 320,
      "avgIntensity": "High",
      "topDomain": "Frontend",
      "status": "COMPLETED"
    },
    {
      "semesterId": "sem_2",
      "name": "Semester 2",
      "isActive": false,
      "totalHours": 290,
      "avgIntensity": "Moderate",
      "topDomain": "Backend",
      "status": "COMPLETED"
    },
    {
      "semesterId": "sem_1",
      "name": "Semester 1",
      "isActive": false,
      "totalHours": 250,
      "avgIntensity": "Moderate",
      "topDomain": "Fundamentals",
      "status": "COMPLETED"
    }
  ]
}
```

`status` enum: `ONGOING`, `COMPLETED`
`isActive` drives the "(Active)" suffix shown next to the semester name in the UI — not sent as literal text from the API.

---

### 3.4 GET `/coding-hours/semesters/{semesterId}/sessions`

Powers the "Session History" table on a single semester's breakdown page (e.g. Semester 4 Breakdown), including breadcrumb context, filter, and pagination.

**Path params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Query params**
| Param | Type | Required | Notes |
|---|---|---|---|
| page | integer | no | default `1` |
| pageSize | integer | no | default `4` |
| intensity | string | no | `Moderate` \| `High` \| `Ultra High` — powers the "Filter" button |
| focusArea | string | no | e.g. `Custom Hooks` |
| dateFrom | string (`YYYY-MM-DD`) | no | |
| dateTo | string (`YYYY-MM-DD`) | no | |

**Response 200**
```json
{
  "semesterId": "sem_4",
  "semesterName": "Semester 4",
  "isCurrentSemester": true,
  "breadcrumb": ["Dashboard", "Coding Hours", "Semester 4 Breakdown"],
  "description": "Detailed analysis of your technical progression and cognitive endurance throughout the fourth academic module.",
  "page": 1,
  "pageSize": 4,
  "totalItems": 24,
  "totalPages": 6,
  "sessions": [
    {
      "sessionId": "sess_a101",
      "date": "2023-10-24",
      "sessionLabel": "MORNING SPRINT",
      "durationMinutes": 260,
      "intensity": { "level": "High", "percent": 70 },
      "focusArea": "Custom Hooks",
      "status": "COMPLETED"
    },
    {
      "sessionId": "sess_a100",
      "date": "2023-10-23",
      "sessionLabel": "EVENING DEEP WORK",
      "durationMinutes": 345,
      "intensity": { "level": "High", "percent": 70 },
      "focusArea": "State Management",
      "status": "COMPLETED"
    },
    {
      "sessionId": "sess_a099",
      "date": "2023-10-22",
      "sessionLabel": "REVISION BLOCK",
      "durationMinutes": 120,
      "intensity": { "level": "Moderate", "percent": 40 },
      "focusArea": "System Architecture",
      "status": "COMPLETED"
    },
    {
      "sessionId": "sess_a098",
      "date": "2023-10-20",
      "sessionLabel": "LATE NIGHT HACK",
      "durationMinutes": 195,
      "intensity": { "level": "High", "percent": 55 },
      "focusArea": "API Integration",
      "status": "COMPLETED"
    }
  ]
}
```

`status` enum: `COMPLETED`, `IN_PROGRESS`, `FLAGGED`
`durationMinutes` is raw — the frontend formats it to `"4h 20m"` rather than the API sending a pre-formatted string, consistent with how the rest of this spec treats display formatting as a UI concern.

---

### 3.5 GET `/coding-hours/semesters/{semesterId}/sessions/export`

Backs the "Export" button. Streams a file rather than JSON.

**Path params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Query params**
| Param | Type | Required | Notes |
|---|---|---|---|
| format | string | no | `csv` (default) \| `xlsx` |
| *(same filters as 3.4)* | | no | `intensity`, `focusArea`, `dateFrom`, `dateTo` apply to the exported set too |

**Response 200**
`Content-Type: text/csv` (or the appropriate `xlsx` MIME type), `Content-Disposition: attachment; filename="semester-4-sessions.csv"` — binary/text body, not JSON.

---

## 4. Attendance Page (`/attendance`)

This is a distinct page from Coding Hours — it tracks **class attendance** (calendar-based), not deep-focus work sessions. Shares the `semesterId` query param convention used elsewhere.

### 4.1 GET `/attendance/summary`

Powers the hero card ("Your streak is exceptional", current sem attendance %, target goal + progress bar) and the two side cards (Total Classes, Late Arrivals).

**Query params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | yes |

**Response 200**
```json
{
  "attendancePercent": {
    "value": 92,
    "unit": "%"
  },
  "targetGoal": {
    "value": 95,
    "unit": "%"
  },
  "totalClasses": {
    "value": 124,
    "unit": "Sessions"
  },
  "lateArrivals": {
    "value": 4,
    "unit": "Times"
  }
}
```

> Note: `attendancePercent.value / targetGoal.value` drives the progress bar fill on the frontend — no separate `progressPercent` field is sent to avoid drift between the two numbers. All display copy (tag, headline, card labels) is owned by the UI, not the API.

---

### 4.2 GET `/attendance/calendar`

Powers the monthly calendar grid ("Attendance Records"), including the prev/next month navigation.

**Query params**
| Param | Type | Required | Notes |
|---|---|---|---|
| semesterId | string | yes | |
| month | integer | yes | `1`–`12` |
| year | integer | yes | e.g. `2023` |

**Response 200**
```json
{
  "month": 9,
  "year": 2023,
  "label": "September 2023",
  "days": [
    { "date": "2023-08-28", "inCurrentMonth": false, "status": null },
    { "date": "2023-08-29", "inCurrentMonth": false, "status": null },
    { "date": "2023-08-30", "inCurrentMonth": false, "status": null },
    { "date": "2023-08-31", "inCurrentMonth": false, "status": null },
    { "date": "2023-09-01", "inCurrentMonth": true, "status": "PRESENT" },
    { "date": "2023-09-02", "inCurrentMonth": true, "status": "PRESENT" },
    { "date": "2023-09-03", "inCurrentMonth": true, "status": "NO_CLASS" },
    { "date": "2023-09-04", "inCurrentMonth": true, "status": "PRESENT" },
    { "date": "2023-09-20", "inCurrentMonth": true, "status": "ABSENT" },
    { "date": "2023-09-29", "inCurrentMonth": true, "status": "NO_CLASS" },
    { "date": "2023-09-30", "inCurrentMonth": true, "status": "NO_CLASS" }
  ]
}
```

`status` enum: `PRESENT` (teal dot), `ABSENT` (red date/dot), `LATE`, `NO_CLASS` (no dot, greyed if outside current month)

> Only a representative slice of `days` is shown above — the real response includes one entry per calendar cell (typically 35 or 42 to fill a full grid).

---

### 4.3 GET `/attendance/calendar/{date}`

Detail for a single day cell, for when a date is clicked (e.g. to see which class/session drove a PRESENT/ABSENT/LATE mark).

**Path params**
| Param | Type | Required |
|---|---|---|
| date | string (`YYYY-MM-DD`) | yes |

**Response 200**
```json
{
  "date": "2023-09-20",
  "status": "ABSENT",
  "classes": [
    {
      "classId": "cls_4471",
      "subject": "Data Structures",
      "scheduledTime": "09:00",
      "status": "ABSENT"
    }
  ]
}
```

---

## 5. Review Taken Page (`/reviews`)

Lists mentor/panel reviews the student has been through — filterable by name, sortable, paginated.

### 5.1 GET `/reviews/summary`

Powers the "Total Reviews" and "Avg. Score" cards at the top.

**Query params**
| Param | Type | Required |
|---|---|---|
| semesterId | string | no — omit for all-time totals |

**Response 200**
```json
{
  "totalReviews": {
    "value": 128,
    "trend": { "direction": "up", "value": "+12%", "comparedTo": "last semester" }
  },
  "avgScore": {
    "value": 84,
    "unit": "%"
  }
}
```

---

### 5.2 GET `/reviews`

Powers the filter input, sort dropdown, review list, and pagination.

**Query params**
| Param | Type | Required | Notes |
|---|---|---|---|
| search | string | no | matches against review `title` (the "Filter reviews by name…" box) |
| sort | string | no | `latest` (default), `oldest`, `highest_score`, `lowest_score` |
| status | string | no | `COMPLETED` \| `REVIEWING` \| `ARCHIVED` — filter chip if added later |
| page | integer | no | default `1` |
| pageSize | integer | no | default `10` |

**Response 200**
```json
{
  "page": 1,
  "pageSize": 10,
  "totalItems": 128,
  "totalPages": 13,
  "reviews": [
    {
      "reviewId": "rev_5521",
      "title": "Neural Architecture Search (NAS) Fundamentals",
      "description": "Detailed evaluation of cell-based search spaces and reinforcement learning agents.",
      "level": "ADVANCED",
      "date": "2023-10-24",
      "status": "COMPLETED"
    },
    {
      "reviewId": "rev_5518",
      "title": "Distributed Ledger Consistency Models",
      "description": "Review of CAP theorem applications in modern NoSQL clusters.",
      "level": "BASIC",
      "date": "2023-10-22",
      "status": "REVIEWING"
    },
    {
      "reviewId": "rev_5510",
      "title": "Graph Theory: Max-Flow Min-Cut",
      "description": "Historical archival of the Ford-Fulkerson algorithm application review.",
      "level": "ADVANCED",
      "date": "2023-10-19",
      "status": "ARCHIVED"
    },
    {
      "reviewId": "rev_5502",
      "title": "Transformer Attention Mechanisms",
      "description": "Cross-examination of multi-head attention versus sparse attention variants.",
      "level": "INTERMEDIATE",
      "date": "2023-10-15",
      "status": "COMPLETED"
    }
  ]
}
```

`level` enum: `BASIC`, `INTERMEDIATE`, `ADVANCED`
`status` enum: `COMPLETED`, `REVIEWING`, `ARCHIVED`

> Note: the design shows `"BAISIC"` as a badge label — that's a copy typo in the UI layer, not a value the API should emit. The API returns the corrected `BASIC` and the frontend is responsible for display text/casing (e.g. uppercasing badges).

---

### 5.3 GET `/reviews/{reviewId}`

Detail view when a row's chevron (`>`) is clicked.

**Response 200**
```json
{
  "reviewId": "rev_5518",
  "title": "Distributed Ledger Consistency Models",
  "description": "Review of CAP theorem applications in modern NoSQL clusters.",
  "level": "BASIC",
  "date": "2023-10-22",
  "status": "REVIEWING",
  "reviewer": {
    "name": "Dr. Ananya Rao",
    "role": "Mentor"
  },
  "score": null,
  "feedback": null
}
```

`score` and `feedback` are `null` until `status` becomes `COMPLETED` or `ARCHIVED`.

---

## 6. Error Responses (all endpoints)

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired token"
  }
}
```

| HTTP Status | Code | Meaning |
|---|---|---|
| 400 | `BAD_REQUEST` | Invalid query params |
| 401 | `UNAUTHORIZED` | Missing/invalid token |
| 403 | `FORBIDDEN` | No access to this semester's data |
| 404 | `NOT_FOUND` | Resource (e.g. sessionId) doesn't exist |
| 500 | `INTERNAL_ERROR` | Server error |

---

## 7. Page → Endpoint Map

| Page | UI Element | Endpoint |
|---|---|---|
| Dashboard | Avatar / semester dropdown | `GET /user/profile` |
| Dashboard | Welcome banner + score ring | `GET /dashboard/summary` |
| Dashboard | Placement Eligibility cards | `GET /dashboard/placement-eligibility` |
| Dashboard | Intellect Metrics cards | `GET /dashboard/intellect-metrics` |
| Coding Hours | Top stat cards (all or single semester) | `GET /coding-hours/summary?semesterId=all` |
| Coding Hours | Coding Hours Trend chart | `GET /coding-hours/trend` |
| Coding Hours | Semester Breakdown table | `GET /coding-hours/semesters` |
| Coding Hours | Row chevron → Semester Breakdown page | `GET /coding-hours/summary?semesterId=sem_4` |
| Coding Hours | Session History table | `GET /coding-hours/semesters/sem_4/sessions` |
| Coding Hours | Filter button | `GET /coding-hours/semesters/sem_4/sessions?intensity=High` |
| Coding Hours | Export button | `GET /coding-hours/semesters/sem_4/sessions/export` |
| Attendance | Hero card + side stat cards | `GET /attendance/summary` |
| Attendance | Calendar grid | `GET /attendance/calendar?month=9&year=2023` |
| Attendance | Prev/next month nav | `GET /attendance/calendar?month=8\|10&year=2023` |
| Attendance | Day cell click | `GET /attendance/calendar/{date}` |
| Review Taken | Total Reviews / Avg. Score cards | `GET /reviews/summary` |
| Review Taken | Filter reviews by name | `GET /reviews?search=neural` |
| Review Taken | Sort dropdown ("Latest") | `GET /reviews?sort=latest` |
| Review Taken | Review list + pagination | `GET /reviews?page=2` |
| Review Taken | Row chevron click | `GET /reviews/{reviewId}` |
