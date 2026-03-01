# FLUYE – Technical and Product Vision

**Version:** 1.0  
**Date:** March 2026  
**Author:** FLUYE Team  
**Repository:** [github.com/fitnessfluye](https://github.com/fitnessfluye)

---

## 1. Introduction

FLUYE is a SaaS platform designed to **increase member retention in gyms** through a combination of:

- **Real-time smart capacity tracking**
- **AI-driven footfall prediction**
- **Gamified community and social events**

Our mission is to transform traditional gyms into **active communities**, reducing churn (currently 63% in the first 3 months) and increasing customer lifetime value.

---

## 2. The Problem (With Data)

| Problem | Impact |
| :--- | :--- |
| Overcrowding | 40% of members quit due to saturation and lack of equipment |
| Loneliness / Gymtimidation | 35–40% feel intimidated; lack of community accelerates churn |
| Lack of motivation | Without social stimuli, consistency drops by 45% after the first few weeks |
| Cold apps | Current apps are mere administrative tools; they don't create emotional bonds |

FLUYE tackles these causes **with technology and behavioral psychology**.

---

## 3. Product Overview

FLUYE consists of:

- **Central backend** (REST API + AI engine)
- **Embeddable components** (webviews to integrate into existing apps)
- **Optional SDK** for native integration
- **Dashboard** for gym managers

The entire system is designed to **integrate with leading management software** (Virtuagym, CrossHero, etc.) by leveraging their APIs and webhooks.

---

## 4. Technical Architecture

[Gym's existing app] ←→ [FLUYE SDK / Webviews] ←→ [FLUYE API]
↑
[FLUYE Backend]
↑
[Database]
↑
[Virtuagym Webhooks]
↑
[Turnstiles / Check-ins]

### 4.1 High-Level Diagram
### 4.2 Detailed Components

| Component | Technology | Description |
| :--- | :--- | :--- |
| **REST API** | Node.js / Python FastAPI | Secure endpoints to query capacity, events, streaks, users |
| **Webhook Receiver** | Express / Django | Endpoint that receives real-time check-ins/outs from Virtuagym |
| **Capacity Engine** | In-memory logic + Redis | Calculates current occupancy = entries - exits |
| **Prediction Engine** | Python + scikit-learn / TensorFlow Lite | Model based on historical data (hour, day, weather, events) with confidence % |
| **Database** | PostgreSQL + Redis | Persistent data (users, events, streaks) + session cache |
| **SSO Authentication** | JWT + OAuth2 | Validation of Virtuagym tokens for unified login |
| **Web Components** | React / Vue.js | Embeddable webviews (capacity, feed, events, profile) |
| **SDK (future)** | Kotlin / Swift | Native libraries for deeper integration |

### 4.3 Integration with Virtuagym

Virtuagym exposes its **Club Member API (V1)** which allows:

- Retrieving the member list and basic data.
- Receiving **webhooks** for each check-in/out (via the "watch visits" module).
- Querying memberships and statuses.

**Integration Flow:**

1. The gym authorizes FLUYE as an external application.
2. Virtuagym sends a webhook to our endpoint for each access.
3. FLUYE updates real-time capacity.
4. The gym's app (via webview) queries our endpoint `/api/capacity/{gym_id}`.
5. Users can create events, view their streak, etc., all stored in our DB.

---

## 5. Key Features (Technical Detail)

### 5.1 Real-Time Capacity

- **Endpoint:** `GET /api/gym/{id}/capacity`
- **Response:** `{ "current": 47, "max": 120, "percentage": 39, "status": "low" }`
- **Source:** Check-in/out webhooks + Redis cache.

### 5.2 AI-Powered Prediction

- **Model:** LSTM network (time series) or statistical model with moving averages.
- **Inputs:** Historical capacity (last 3–6 months), day of week, hour, holidays, weather (external API).
- **Output:** Prediction by time slot with confidence %.
- **Endpoint:** `GET /api/gym/{id}/prediction?date=2026-03-10`
- **Response:** `[ { "hour": "16:00", "occupancy": 45, "confidence": 78 }, ... ]`

### 5.3 Gamified Community (The Heart of FLUYE)

#### 5.3.1 Visual Streaks

- **Mechanic:** User accumulates consecutive days of visits.
- **Visual:** Flame that grows and changes color (orange → red → crown).
- **Logic:** Updated daily via cron job that queries previous day's attendances.
- **Special badges:** "Eternal Flame" (60 days), "Phoenix" (recover streak after losing it).

#### 5.3.2 User-Created Events

- **Data model:** `Event { id, gym_id, creator_id, title, description, date, time, location, max_attendees, attendees[] }`
- **Flow:**
  1. User creates event via webview.
  2. Saved in DB and appears in feed.
  3. Other users join ("Attend" button).
  4. Creator can edit/cancel.
- **Event types:**
  - **Internal challenge:** "Bench press PR", "1000 team abs"
  - **Social:** "Saturday 9am running meetup", "Post-workout beers"
  - **Educational:** "Mobility workshop with trainer (paid)"

#### 5.3.4 Ranking System (Levels)

- **Metrics:** Points for attendance, events created, event attendance, interactions.
- **Levels:** Beginner → Explorer → Warrior → Hero → Legend.
- **Benefits:** Unlock special badges, appear in rankings, discounts (if configured by the gym).

#### 5.3.5 Social Feed

- **Endpoint:** `GET /api/gym/{id}/feed`
- **Content:**
  - Achievements: "María just got a 14-day streak 🔥"
  - Upcoming events: "Running meetup - Saturday 9am (8 attendees)"
  - Challenges: "This week: Squat challenge. Join in!"
- **Interaction:** Likes, comments (future).

#### 5.3.6 Gym Buddies (User Matching)

- **Function:** Optional system that, based on similar schedules and goals, suggests potential workout partners.
- **Privacy:** Visible only if both accept.
- **Match:** "Carlos and you both like to train on Mondays at 6pm. Would you like to connect?"

#### 5.3.7 Weekly Ranking

- **Categories:**
  - Attendance: days per week
  - Progress: improvement in personal records (if integrated with equipment)
  - Community: events created + attendances
- **Visibility:** Weekly top 10, with medal for the winner.

---

## 6. Development Roadmap

| Phase | Duration | Technical Milestones |
| :--- | :--- | :--- |
| **Phase 1** | Months 1–3 | Core backend: API, DB, webhook receiver, basic capacity engine |
| **Phase 2** | Months 3–5 | Community MVP: profiles, streaks, event creation, basic feed |
| **Phase 3** | Months 5–7 | AI prediction (statistical model), embeddable web components |
| **Phase 4** | Months 7–9 | Gym dashboard, advanced gamification (ranks, badges) |
| **Phase 5** | Months 9–12 | Native SDK (iOS/Android), gym buddies, wearable integration |

---

## 7. Virtuagym Integration Model (Detailed)

### 7.1 API Requirements from Virtuagym

Based on their public documentation and other developers' experiences, we need:

| Resource | Availability |
| :--- | :--- |
| Member list | ✅ Yes (Club Member API) |
| Real-time check-ins | ✅ Yes (via webhooks / "watch visits") |
| Membership data | ✅ Yes |
| Event creation | ❌ Not confirmed (we can store events in our DB without syncing) |

### 7.2 Proposed Data Flow

1. **Gym onboarding:**
   - Gym provides us with `API Key` and `Club Secret` (Virtuagym credentials).
   - FLUYE configures a webhook in Virtuagym to receive check-ins.

2. **Initial sync:**
   - Query `GET /members` to obtain the member list.
   - Create a profile in our DB for each member (with their Virtuagym `member_id`).

3. **Real-time:**
   - For each check-in, Virtuagym sends a POST to our endpoint.
   - FLUYE updates the capacity counter and logs the visit in the member's history.

4. **SSO Authentication:**
   - When a member opens the gym's app and accesses a FLUYE webview, the app sends us a JWT token signed by Virtuagym.
   - FLUYE validates the token and authenticates the member without asking for credentials.

### 7.3 Security

- All communications over HTTPS.
- Secure storage of credentials (encrypted).
- JWT tokens with expiration time.
- GDPR compliance (minimal personal data, ability to export/delete).

---

## 8. Key Differentiators (What NO ONE Else Has)

| Feature | FLUYE | Competition (Virtuagym, etc.) |
| :--- | :--- | :--- |
| Member‑visible capacity | ✅ Yes | ❌ Internal only |
| AI prediction with confidence % | ✅ Yes | ⚠️ Only "popular hours" (Basic-Fit) |
| User‑created events | ✅ Yes | ❌ Only club news |
| Advanced gamification (visual streaks, ranks) | ✅ Yes | ⚠️ Basic achievements |
| Social feed with interactions | ✅ Yes | ❌ |
| Gym buddies (member matching) | ✅ Yes | ❌ |

---

## 9. Conclusion

FLUYE is not a management app. It is the **experience and community layer** that gyms need to retain members in an increasingly competitive market.

Our integration with Virtuagym allows us to:
- Leverage their check-in infrastructure and member data.
- Immediately add value to their 1,000+ clients in Spain.
- Create an ecosystem where **everyone wins**: the gym retains members, Virtuagym enhances its offering, and FLUYE becomes the go‑to partner for engagement.

**We are building the future of the gym experience. Join us?**
