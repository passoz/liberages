# Decision Log

> Decisions recorded during strategy session — Jul 2026

| Decision | Value |
|----------|-------|
| **Scope** | All 71 features (grouped/trimmed as needed). No stripped-down MVP. |
| **PSP** | Interface Mercado Pago (swappable). Concrete integration TBD. |
| **Moderation Gate** | Feature #7 (Moderated Entry) is the first filter. No open access like Sexlog. |
| **Backup/DR** | Sidecar container copying SQLite to S3-compatible storage (MinIO?). |
| **Database** | SQLite in v1. Migrate when multiple containers needed. |
| **Storage** | Local filesystem first. S3 adapter later, same as DB. |
| **Auth** | Local bcrypt+JWT first. Keycloak adapter when needed. |
| **Sexlog defense** | Features are the hook for a multi-network community, not direct feature parity. |
| **Location model** | 3 modes: (1) Check-in at catalog locations, (2) Destination broadcast, (3) "Tô Querendo" — temporary GPS with error margin, auto-off near home (client-side). Server aggregates by location, never exposes raw coordinates. Triangulation risk minimal due to short window + error margin + anonymity. |
| **Implementation plan** | Founder will define separately. |
| **User Identity Model** | One User = one unfoldable Profile. Account can be single, couple, or throuple (needs prior moderation). Anonymity is a mode for community discussions with pseudonyms — not a separate identity. |
| **Profile Architecture** | Profile is not a separate entity from User; it's the same entity with different facets/modes. No multiple profiles per user. |
| **Matching & Friendship** | Match = two positive swipes → suggests Friendship. Friendship has category: `real` (know in person) or `virtual` (online only). No "Encontro Surpresa" feature. |
| **Check-in & Radar** | Check-in expires (TTL). Radar = active checkins per location, always anonymous. User notified that "X people are at Y" — never who. Identity never revealed via radar; only through chat after establishing contact. |
| **Identity Policy** | Real identities discouraged. Pseudonyms/nicknames only. No face photos unless user explicitly opts in per-profile. |
| **Content Types** | Fotolog (daily photo, 24h feed), Album (curated event gallery, no expiry), Contos (text), Live (streaming), Marketplace (packs). No Story feature. Each is a separate entity — not polymorphic ContentItem. |
| **Gamification — Core** | Levels unlock features. XP from interactions. Badges for milestones. Not cosmetic — gating mechanism. |
| **Verified Badge** | Social verification (web of trust): 4 existing verified users with `real` friendship must certify the profile. Not a platform-issued verification. |
| **Anonymous Communities** | User chooses a pseudonym per community. Moderators/owners can see real identity (for moderation). Other members see only the pseudonym. No anonymous participation outside communities. |
| **Moderation Flow** | Report → AI initial verification (triage) → human reviews and decides. AI never acts alone — always human-in-the-loop. Actions: ban, suspension, content removal, warning — depends on case. |
| **Júri Popular** | Only for broader/complex issues (not misconduct reports). Only Angels vote. Not a first-level moderation tool. |
| **Anjo da Comunidade** | Badge granted by platform admin to verified users. Angels can participate in Júri Popular. Not a moderator role — honorific + voting power. |
| **Push Notifications** | Only 4 types: (1) new chat message, (2) friend nearby (1km radius), (3) check-in nearby (shows location name), (4) like on profile/fotolog/album (batched — "João e +3"). No quiet hours. User can configure per type. |
| **Notification Delivery** | Hybrid: Web Push API for all 4 types + WebSocket for chat real-time when both online. |
| **Notification Persistence** | Table `notifications` in SQLite with 90-day TTL. Cleanup job. Powers bell/feed UI. |
| **Push Permission Timing** | Contextual: ask at first notifiable event (e.g., first like received), not at install. |
| **Ghost Mode** | Visibility states: `visible`, `invisible` (only friends), `ghost` (completely hidden). Optional timer (auto-disable). **Premium feature.** Both Invisible and Ghost are premium — "Complete Privacy" package. |
| **Likes Received Visibility** | Free: see who liked within 2 hours of the like. Premium: unlimited time to see who liked you. Drives daily engagement. |
| **Match Boost** | Hybrid: premium subscribers get 1 free boost/week. Non-subscribers can buy avulso (R$2-5). Maximizes both subscription + microtransaction revenue. |
| **Albums Limit** | Free: up to 3 albums. Premium: unlimited. |
| **Contos Publishing** | Free: read only. Premium can publish. Alternative gate: badges/XP can unlock publishing rights too. |
| **Badges as Feature Gates** | Badges can unlock various rights (more contos, more album photos, more fotologs per day, etc.). Not just cosmetic — functional. |
| **Live Streaming** | Out of scope entirely. High infra/moderation cost. |
| **Marketplace (Content Sales)** | Out of scope entirely. Not a feature. |
| **Ads Model** | Free tier: display + native B2B ads (relevant to audience). Premium: ad-free. |
| **Economy — Single Currency** | One platform currency for everything. Earned via: checkins, interactions, achievements, challenges, purchases. Spent on: premium features, boosts, partner discounts, virtual gifts. Unifies XP, fidelity points, and microtransactions. |
| **B2B Features** | Cartão Fidelidade (checkin → points → partner discounts) + Caça ao Tesouro (collect → vouchers). In scope for MVP. Powered by single currency. |
| **Modo Falso (Emergency Button)** | Free for all users. Safety feature — cannot be paywalled. |
| **Profile Visits** | Same model as likes: free sees visits within time window (drives engagement), premium sees full history. |
| **Gift Subscription** | In scope. User can purchase Premium for another user. Revenue vector + interest gesture without requiring match. |
| **Currency — Purchasable** | Platform currency is purchasable with real money (R$). Not transferable between users (no P2P transfer). Gifts (cost currency) are transferable. Prevents parallel market. |
| **Premium Payment** | Hybrid: real money OR partial currency + top-up. Currency never covers 100% of subscription. Protects revenue while valuing grind. |
| **Location Catalog** | Curated initial set (50-100 per city at launch). Then community submissions with moderation. |
| **Location Data Model** | Regular: name, address, coordinates, category, photos, hours, rating, tags (B). Partners (B2B): full C — plus events, live status, avg price, social links, report closed. |
| **Check-in TTL** | User-configurable (1h, 2h, 4h) with a maximum cap. User chooses at check-in time based on intention. |
| **Destination Broadcast** | Not a check-in. A one-time broadcast: "going to X tonight, who else?" Interested users reply anonymously and can arrange. Disappears after event. |
| **"Tô Querendo" Mode** | Cut from MVP. Potential post-MVP merge with check-in at generic areas (neighborhoods, regions) rather than specific venues. |
| **DMs Encryption** | HTTPS-only for MVP (server can read — enables moderation). E2E (Web Crypto + ECDH) planned post-MVP. Architecture documented upfront for smooth migration. |
| **Age Verification** | Hybrid: auto-declaration (cookie) for initial access. Anonymous document verification to unlock adult content/interactions. No real name stored. |
| **Forum & Communities** | Unified structure. Forum = open space (anyone reads/posts). Community = same base + anonymity (pseudonyms) + moderator visibility. Both share: categories, topics, posts, voting, FTS5 search. |
| **Screenshot Protection** | Every photo rendered with dynamic watermark (viewer's user ID hash). Light heuristic detection (Page Visibility + keyboard shortcuts) as bonus. Watermark is the real deterrent — traceable leaks. |
| **Moderated Entry (Onboarding)** | Layered access: (1) register + auto-declaration → browse only (profiles, forum read). (2) anonymous document verification → unlock interactions (check-in, fotolog, contos, chat). Verification async — notified when approved. |
| **Couple/Throuple Account** | One shared account with one shared profile. Both members administer it. One login (shared credentials or PIN), one public identity. Single account = single profile per person. |
| **Login with PIN** | Primary login method for day-to-day. PIN (4 digits) on registered device. Email+password only for first login or forced logout. |
| **Blur Facial** | Automatic on all photos by default. User must explicitly enable "show face" per post. Privacy-first design. |
| **Selfie Destrutível** | Chat-only feature: send photo that auto-deletes after being opened once. Intimate, ephemeral, disposable. No feed logic needed. |
| **Admin Dashboard** | Medium scope for MVP: user management (ban, verification approval), location management, moderation queue, basic metrics (DAU, matches, check-ins/day, revenue). Advanced analytics post-MVP. |
| **Fetish/Interest Catalog** | Hybrid approach: fixed taxonomy (curated list) for calculating match %, plus 3 free-text tags for personality. |
| **Match Algorithm** | Funnel: Location first (distance radius) → Compatibility second (% match on fixed taxonomy). |
| **Match Calculation** | Simple intersection for MVP (Jaccard similarity or simple common count). Directional/role-based matching deferred to post-MVP. |
| **Bucket List** | Acts as post-match icebreaker. If two users match and share a location in their bucket list, app suggests it for the date. |
| **Swipe Limits** | Free tier has a daily limit on likes (e.g., 30/day). Premium tier has unlimited swipes. Standard dating app monetization loop. |
| **Global Profile Search** | Broad FTS5 search enabled (city, bio, tags, pseudonym). The app is a social network first, swipe is just a supplementary feature. Privacy is maintained via pseudonyms, face blur, and Ghost Mode. |
| **Fotolog Feed Algorithm** | Single continuous infinite feed (no tabs). Priority sorting: 1st) Friends, 2nd) Nearby strangers (location), 3rd) Distant strangers with high compatibility (interests). |
| **Economy — Cash-out Policy** | Closed-loop economy. No cash-out (Moeda → R$) for users. Eliminates banking compliance/KYC complexity. B2B partner compensation handled out-of-band via separate contracts. |
| **B2B Venue Management** | Admin-managed (concierge approach) for MVP to save dev time. |
| **Events Creation** | UGC (User Generated Content). Any user can create an event, not just B2B partners. |
| **Events Visibility** | Verified users can create public events (visible on map/agenda). Unverified users can only create private events (accessible via invite/PIN). Uses Web of Trust as a safety gate. |
| **Carona Solidária** | Out of scope. Removed due to physical safety liability and scope creep. |
