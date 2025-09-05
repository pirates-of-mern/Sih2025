# Project Title (Working): **Bharat Sanskriti Atlas**

*A modern, map‑first platform to discover, preserve, and promote India’s cultural heritage (arts, crafts, festivals, food, monuments, folk traditions) with an artisan directory and interactive storytelling.*

---

## 0) TL;DR – What we’ll build in the hackathon

**MVP (demo‑ready, 48–72 hrs):**

* 🌏 **Cultural Atlas:** India map → click a state → see cards for **Dance, Food, Crafts, Festivals, Monuments**.
* 🧑🏽‍🎨 **Artisan Directory:** searchable profiles with location, craft, images, and contact links.
* 📖 **Story Cards:** short, multimedia “Did you know?” snippets with audio narration.
* 🔎 **Search + Filters:** state, category, tribe/community, time period.
* 💬 **Feedback/Contribute:** simple form to submit corrections/new entries (goes to moderation).

**Stretch (if time permits):**

* 🗺️ **Heritage Trails:** curated routes (e.g., “Textile Trail of Kutch”).
* 🖼️ **AR Postcards (WebXR):** 3D artifact preview or monument mini‑model.
* 🤖 **AI Narrator:** text‑to‑speech + language switcher; recommendation feed by interests.
* 🔍 **Image Recognizer:** upload a monument/photo → suggest possible matches.

---

## 1) System Architecture (High Level)

**Frontend (Web):** React + Vite + Tailwind CSS, React Router, Zustand/Context for state, Map library (MapLibre GL JS or Leaflet), React Query for data fetching, i18next for multilingual (EN/HI at least).

**Backend (API):** Node.js + Express, REST APIs, JWT auth (users/admins), input validation (zod/joi), rate limiting (express-rate-limit), logging (morgan/winston).

**Database:** MongoDB Atlas (GeoJSON for map layers, Atlas Search for full-text), collections listed below.

**Storage/CDN:** Cloudinary **or** AWS S3 (images/audio). Use signed URLs; thumbnails via Cloudinary transformations.

**Optional CMS:** Strapi or Sanity (headless) to speed up content authoring. If not, build a minimal admin panel.

**Map Tiles/Data:** MapLibre/Leaflet with open vector tiles; state/district boundaries via GeoJSON (public datasets). Marker clustering for heritage sites/artisans.

**Deployment (hackathon grade):** Vercel/Netlify for frontend; Render/Railway/Heroku for backend; MongoDB Atlas free tier.

```
[ React (Map + UI) ]  <—HTTP/JSON—>  [ Express API ]  <—>  [ MongoDB Atlas ]
       |                                   |
   Cloudinary/S3 (media)               Atlas Search
```

---

## 2) Data Model (MongoDB – draft)

**users**

```json
{
  "_id": ObjectId,
  "name": "Admin/Curator",
  "email": "admin@…",
  "passwordHash": "…",
  "role": "admin|curator|viewer",
  "createdAt": ISODate
}
```

**states** (reference table)

```json
{
  "_id": ObjectId,
  "code": "GJ",
  "name": "Gujarat",
  "geometry": { "type": "MultiPolygon", "coordinates": [...] } // optional for state outline
}
```

**heritageItems** (generic for Dance/Food/Craft/Festival/Monument/Folklore)

```json
{
  "_id": ObjectId,
  "type": "dance|food|craft|festival|monument|folklore",
  "title": "Garba",
  "summary": "Folk dance from Gujarat performed during Navratri.",
  "details": "rich markdown/html",
  "stateCodes": ["GJ"],
  "community": ["Gujarati", "Kutchi"],
  "period": "Medieval|Modern|Ancient",
  "media": [{"url": "https://…", "kind": "image|audio|video", "credit": "…"}],
  "location": { "type": "Point", "coordinates": [72.5714, 23.0225] },
  "tags": ["dance", "navratri", "folk"],
  "sources": ["ASI", "Ministry of Culture", "Author Name"],
  "status": "published|pending|rejected",
  "createdBy": ObjectId,
  "createdAt": ISODate
}
```

**artisans**

```json
{
  "_id": ObjectId,
  "name": "Rameshbhai",
  "craft": "Bandhani tie‑dye",
  "bio": "3rd‑gen artisan from Jamnagar…",
  "contact": {"phone": "…", "instagram": "…", "website": "…"},
  "location": { "type": "Point", "coordinates": [70.07, 22.47] },
  "stateCode": "GJ",
  "images": ["https://…"],
  "verified": true,
  "createdAt": ISODate
}
```

**trails** (curated routes)

```json
{
  "_id": ObjectId,
  "title": "Textile Trail of Kutch",
  "stops": [ { "ref": ObjectId("heritageItem/ artisan"), "note": "…" } ],
  "distanceKm": 120,
  "stateCodes": ["GJ"],
  "createdBy": ObjectId
}
```

**contributions** (user submissions for moderation)

```json
{
  "_id": ObjectId,
  "type": "heritage|artisan|correction",
  "payload": { /* same shape as target entity */ },
  "submittedBy": { "name": "…", "email": "…" },
  "status": "pending|approved|rejected",
  "reviewerId": ObjectId,
  "createdAt": ISODate
}
```

Indexes:

* `heritageItems`: `type`, `stateCodes`, `tags`, `location` (2dsphere), Atlas Search on `title, summary, details`.
* `artisans`: `craft`, `stateCode`, `location` (2dsphere).

---

## 3) API Design (Express – sample)

**Auth**

* `POST /api/auth/register`
* `POST /api/auth/login`

**Public Data**

* `GET /api/states`
* `GET /api/heritage?state=GJ&type=dance&tag=navratri&near=lat,lng&radius=50`
* `GET /api/heritage/:id`
* `GET /api/artisans?state=GJ&craft=Bandhani&near=lat,lng`
* `GET /api/trails`

**Contributions**

* `POST /api/contributions` (public submit)
* `GET /api/contributions` (admin)
* `PATCH /api/contributions/:id` (approve → writes to main collections)

**Media Uploads**

* `POST /api/upload` → returns Cloudinary/S3 URL (admin/curator only)

**Admin/Curator**

* `POST /api/heritage`
* `PATCH /api/heritage/:id`
* `DELETE /api/heritage/:id`
* Similar for artisans/trails

Validation: zod/joi; Security: JWT + role guard; Rate limit on search endpoints.

---

## 4) Frontend – Pages & Components

**Pages**

* `/` Home (hero + search + featured states & categories)
* `/atlas` Map view with filters (state/category/tags)
* `/heritage/:id` Detail page (gallery, map pin, story, sources)
* `/artisans` Directory (list + map toggle)
* `/trails` Curated routes
* `/contribute` Submit form
* `/admin` Minimal CMS (only if not using Strapi/Sanity)

**Core Components**

* `MapView` (MapLibre/Leaflet + clustering + popups)
* `FilterBar` (state/type/tags/search)
* `CardGrid` (responsive cards)
* `DetailTabs` (Overview | Gallery | Map | Sources)
* `AudioNarrator` (TTS player)
* `ContributeForm`
* `Toast/Modal` system

**State/Data**

* React Query for server cache; Zustand/Context for UI state (filters, selections).

---

## 5) Implementation Plan (Day‑by‑Day)

**Day 0 (1–2 hrs):**

* Finalize MVP scope + dataset list (5–8 states, 3–4 categories each; 20–30 artisans).
* Create repo structure (frontend, backend) + env templates.

**Day 1:**

* Backend: Auth, states, heritage list/detail, artisans list; seed sample data.
* Frontend: Vite + Tailwind setup; Router; basic layout; Atlas page skeleton; CardGrid.
* Storage: Cloudinary/S3 configured.

**Day 2:**

* Map integration + filters + search.
* Detail page (gallery + map pin + sources).
* Contribution form + admin review flow (simple table).

**Day 3 (Polish + Pitch):**

* Accessibility pass, loading states, empty states.
* Seed data beautification; add 1 trail and 1 AR postcard (stretch if time).
* Prepare demo script & slides.

---

## 6) Task Breakdown (assign as per team)

**Frontend Lead**

* Layout, navigation, MapView, filters, cards, detail page, TTS.

**Backend Lead**

* Models, controllers, auth, search, geo queries, contributions workflow.

**Data/Content Lead**

* Curate statewise items, verify sources, upload media, prepare seed JSON.

**QA/Docs/Pitch**

* Test scenarios, demo script, slide deck, runbook for judges.

---

## 7) Sample Code (Kickstart)

**Express – heritage route (simplified)**

```js
// routes/heritage.js
import { Router } from 'express';
import Heritage from '../models/Heritage.js';
const r = Router();

r.get('/', async (req, res) => {
  const { state, type, q, near, radius = 50 } = req.query;
  const filter = {};
  if (state) filter.stateCodes = state;
  if (type) filter.type = type;
  if (q) filter.$text = { $search: q };
  if (near) {
    const [lat, lng] = near.split(',').map(Number);
    filter.location = { $near: { $geometry: { type: 'Point', coordinates: [lng, lat] }, $maxDistance: radius * 1000 } };
  }
  const data = await Heritage.find(filter).limit(50);
  res.json(data);
});

export default r;
```

**Mongoose – schema (excerpt)**

```js
// models/Heritage.js
import mongoose from 'mongoose';

const MediaSchema = new mongoose.Schema({
  url: String,
  kind: { type: String, enum: ['image', 'audio', 'video'] },
  credit: String,
});

const HeritageSchema = new mongoose.Schema({
  type: { type: String, enum: ['dance','food','craft','festival','monument','folklore'], index: true },
  title: { type: String, required: true },
  summary: String,
  details: String,
  stateCodes: { type: [String], index: true },
  community: [String],
  period: String,
  media: [MediaSchema],
  location: { type: { type: String, enum: ['Point'] }, coordinates: { type: [Number], index: '2dsphere' } },
  tags: [String],
  sources: [String],
  status: { type: String, enum: ['published','pending','rejected'], default: 'published' }
}, { timestamps: true });

HeritageSchema.index({ title: 'text', summary: 'text', details: 'text' });

export default mongoose.model('Heritage', HeritageSchema);
```

**React – Map skeleton (Leaflet)**

```jsx
// components/MapView.jsx
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';

export default function MapView({ items = [] }) {
  return (
    <div className="h-[70vh] w-full rounded-2xl overflow-hidden shadow">
      <MapContainer center={[22.97, 78.65]} zoom={5} className="h-full w-full">
        <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
        {items.map(i => (
          i.location?.coordinates ? (
            <Marker key={i._id} position={[i.location.coordinates[1], i.location.coordinates[0]]}>
              <Popup>
                <div className="font-semibold">{i.title}</div>
                <div className="text-sm">{i.type} • {i.stateCodes?.join(', ')}</div>
              </Popup>
            </Marker>
          ) : null
        ))}
      </MapContainer>
    </div>
  );
}
```

---

## 8) Demo Script (3–4 minutes)

1. **Hook:** “India has 1000+ living traditions, but discovery is fragmented. We built a map‑first way to explore them.”
2. **Live Demo:** Open atlas → pick a state → open a dance → play narrator → show artisan nearby.
3. **Impact:** cultural education, tourism discovery, artisan livelihoods.
4. **How it works:** tiny slide on stack + geo + Atlas Search.
5. **Scale & Future:** trails, AR postcards, AI recommender, school kits.

---

## 9) Risks & Mitigations

* **Data accuracy:** show sources & moderation queue; version entries.
* **Media rights:** prefer CC‑BY/own photos; store credits; allow takedown.
* **Performance:** CDN images, pagination, clustering.
* **I18N complexity:** start with EN/HI; add more later.

---

## 10) Ready‑to‑Use Checklists

**MVP Data Targets**

* 5–8 states × 3 categories × 3 items each → \~60 curated entries
* 20–30 artisan profiles with coordinates & 2–3 images each

**Tech Setup**

* [ ] Vite + Tailwind + Router
* [ ] React Query + Axios
* [ ] Leaflet + marker cluster
* [ ] Express + JWT + CORS
* [ ] Mongo Atlas + sample seed
* [ ] Cloudinary/S3 env keys
* [ ] Atlas Search index

**Pitch Assets**

* [ ] 6 slides (Problem, Solution, Demo, Tech, Impact, Roadmap)
* [ ] 60‑sec walkthrough video (screen capture)

---

### That’s it – iterate on this doc as we build. Next step below 👇

**Next Step:** Create repos, push boilerplates, and seed 1 state (e.g., Gujarat) with 5 items to wire end‑to‑end.
