# UECFI — Scalability

How UECFI maintains high performance as the song catalog and metadata options expand, and the trade-offs taken along the way.

---

## Scaling Dimensions

UECFI scales along three independent axes:

| Axis | Challenge | Bottleneck |
|------|-----------|------------|
| **Catalog Volume** | Increasing numbers of hymnal packs and song sheets | Database search latency, cold load memory usage |
| **User Interaction** | Dynamic key transposition and font resizing | CPU overhead on render cycles, UI stuttering |
| **Sync Actions** | Multiple song revisions and edits by administrators | Sync queue transaction write conflicts |

Each axis has a distinct optimization strategy.

---

## Database Query Optimizations (SQLite)

When the song catalog grows from hundreds to thousands of records, naive SQL queries slow down. We implement strict optimization layers in local SQLite storage:

1. **Table Indexing** — We defined explicit database indexes on query filtering pathways (`idx_songs_title`, `idx_songs_pack_id`, `idx_songs_category`, `idx_favorites_song_id`, `idx_songs_language`). This ensures search execution operates in $O(\log n)$ time complexity rather than performing expensive full-table scans.
2. **Debounced Search Inputs** — Text inputs on the search bar use a 300ms debounce guard. Database queries run only after the user stops typing, preventing thread blocking from firing queries on every keystroke.
3. **Paginated Data Retrieval** — Rather than loading the entire catalog into mobile memory, the library screen implements paginated queries using SQL `LIMIT` and `OFFSET` operators. Data loads in segments of 50 songs, rendering additional items dynamically as the user scrolls.

---

## Render Cycle Performance (React Native)

Worship songs can be long, containing numerous verse and chorus blocks. Translating and rendering chord layers inline can degrade scroll performance.

- **FlatList Recyclers** — We utilize React Native's native `FlatList` component to recycle list cells, keeping memory utilization flat regardless of list length.
- **Transposition Caching** — The transposed chord sheet calculation is executed once per key change and cached in the parent screen state. Sub-elements (`LyricsView` stanzas) draw from this cache, preventing expensive pitch-shifting computations during layout scrolls.
- **Zustand Selector Isolation** — Store subscriptions are granularly selected:
  ```typescript
  const fontSize = useAppStore((s) => s.fontSize);
  ```
  This ensures that adjustments to font size only trigger re-renders in the song viewer panel, leaving the rest of the navigation layout untouched.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
