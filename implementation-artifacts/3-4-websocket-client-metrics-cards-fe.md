# Story 3.4: WebSocket Client & Metrics Cards (FE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a recruiter,
I want to see live health metrics on project cards update in real time,
So that I can verify demo apps are genuinely running during my visit.

## Acceptance Criteria

**AC1:** Given the Portfolio FE loads,
**Then** a WebSocket connection is established to Platform BE `ws://{host}/ws/metrics`

**AC2:** Given a metrics update message arrives via WebSocket,
**Then** the affected project card's uptime %, response time ms, and last deploy update without a page refresh

**AC3:** Given the WebSocket connection drops,
**Then** automatic reconnection is attempted with exponential backoff — max 3 retries — after which the FE displays last known data (NFR-R3)

**AC4:** Given Platform BE is completely offline,
**Then** the portfolio FE renders all static content correctly — only the live metrics area shows a degraded state, no layout break (NFR-R4)

**AC5:** Given metrics data is temporarily unavailable,
**Then** each affected card shows "Updated X ago" timestamp instead of live values — card layout does not shift

**AC6:** Given the WebSocket connection is established but then disconnects before any data is received,
**When** the connection drops,
**Then** project cards show "Status unavailable" (not "Updated X ago") — the staleness timer only starts after first data receipt.

## Tasks / Subtasks

- [x] Task 1: Create useWebSocket hook (AC: #1, #3)
  - [x] 1.1 Create `src/hooks/useWebSocket.ts` — native WebSocket lifecycle management
  - [x] 1.2 Implement connection to `ws://{host}/ws/metrics` on mount
  - [x] 1.3 Implement exponential backoff reconnection (max 3 retries)
  - [x] 1.4 Handle connection state transitions: connecting → connected → disconnected → error
  - [x] 1.5 Write messages to metricsStore on message receipt

- [x] Task 2: Create metrics constants (AC: #3)
  - [x] 2.1 Create `src/constants/config.ts` with WS_RECONNECT_MAX_RETRIES, WS_RECONNECT_BASE_DELAY_MS
  - [x] 2.2 Export WS_URL from config (reads from VITE_WS_URL env)

- [x] Task 3: Update metricsStore for new fields (AC: #2, #5, #6)
  - [x] 3.1 Add `lastUpdated` timestamp field to MetricsData
  - [x] 3.2 Add `hasReceivedData` flag to track if any data received
  - [x] 3.3 Add helper selectors: `getProjectStatus(projectSlug)` returning 'live' | 'stale' | 'unavailable'

- [x] Task 4: Update ProjectCard to display live metrics (AC: #2, #5, #6)
  - [x] 4.1 Import and use useMetricsStore in ProjectCard
  - [x] 4.2 Display uptime %, response time when data available
  - [x] 4.3 Display "Updated X ago" when data is stale (> 0 and < 24h since lastUpdated)
  - [x] 4.4 Display "Status unavailable" when hasReceivedData is false (never received data)
  - [x] 4.5 Hide metrics section when no data received AND connection is connected (waiting for first data)

- [x] Task 5: Handle graceful degradation (AC: #4)
  - [x] 5.1 Show degraded state in metrics area when connectionState is 'error'
  - [x] 5.2 Ensure static content (hero, projects list, about) still renders
  - [x] 5.3 Test: disconnect backend, verify FE still loads and displays static content

- [x] Task 6: Write tests (AC: all)
  - [x] 6.1 Updated `src/stores/metricsStore.test.ts` for new fields
  - [x] 6.2 Updated `ProjectCard.test.tsx` to test metrics display scenarios

- [x] Task 7: Integration and regression (AC: all)
  - [x] 7.1 Run full test suite: `pnpm test` — 229 tests pass
  - [x] 7.2 Run lint: `pnpm lint` — 0 errors
  - [x] 7.3 Verify no regressions from Epic 1 stories

## Dev Notes

### CRITICAL: Files That ALREADY EXIST — Do NOT Recreate

| File | Location | Status |
|------|----------|--------|
| `metricsStore.ts` | `portfolio-fe/src/stores/` | ✅ EXISTS — add new fields, do NOT recreate |
| `ProjectCard.tsx` | `portfolio-fe/src/components/shared/` | ✅ EXISTS — modify for metrics display |
| `constants/motion.ts` | `portfolio-fe/src/constants/` | ✅ EXISTS — use existing pattern |
| `package.json` | `portfolio-fe/` | ✅ EXISTS — no new deps needed (native WebSocket) |

### CRITICAL: What NOT to Create

- **No WebSocket library** — use native browser `WebSocket` API only
- **No STOMP client** — BE uses native WebSocket, not STOMP
- **No reconnection library** — implement manually with exponential backoff
- **No new Zustand stores** — use existing `metricsStore`

---

### Architecture Requirements

1. **Native WebSocket ONLY — No Libraries:**
   - Use `new WebSocket(url)` directly in useWebSocket hook
   - No `socket.io-client`, `ws`, or any WebSocket library
   - This matches Story 3.3 BE which uses native Spring WebSocket

2. **useWebSocket Hook Pattern:**
   - Single point of WebSocket lifecycle management
   - Called once in App.tsx (root), never in individual components
   - Manages: connect, disconnect, reconnect with exponential backoff, cleanup
   - Writes all state to metricsStore

3. **Connection State Machine:**
   ```
   disconnected → connecting → connected
                        ↓           ↓
                      error ←←←←←←←
   ```
   - On error: attempt reconnect with exponential backoff (max 3 retries)
   - After max retries: stay in 'error' state, display last known data

4. **Message Format (from BE Story 3.3):**
   ```typescript
   // BE sends JSON via WebSocket:
   {
     "projectSlug": "wallet-app",
     "status": "UP",
     "uptime": 99.95,
     "responseTime": 150,
     "lastDeploy": "2026-03-12T10:00:00Z"
   }
   ```

5. **Environment Variables:**
   - `VITE_WS_URL=ws://localhost:8080/ws/metrics` (local dev)
   - Production: set in Vercel dashboard

---

### useWebSocket.ts — Implementation Template

```typescript
import { useEffect, useRef, useCallback } from 'react'
import { useMetricsStore } from '@/stores/metricsStore'
import { WS_RECONNECT_MAX_RETRIES, WS_RECONNECT_BASE_DELAY_MS } from '@/constants/config'

export const useWebSocket = () => {
  const wsRef = useRef<WebSocket | null>(null)
  const reconnectAttemptsRef = useRef(0)
  const reconnectTimeoutRef = useRef<ReturnType<typeof setTimeout>>()

  const setMetrics = useMetricsStore((state) => state.setMetrics)
  const setConnectionState = useMetricsStore((state) => state.setConnectionState)

  const connect = useCallback(() => {
    const wsUrl = import.meta.env.VITE_WS_URL || 'ws://localhost:8080/ws/metrics'
    setConnectionState('connecting')

    const ws = new WebSocket(wsUrl)
    wsRef.current = ws

    ws.onopen = () => {
      setConnectionState('connected')
      reconnectAttemptsRef.current = 0
    }

    ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data)
        // Expected format from BE: { projectSlug, status, uptime, responseTime, lastDeploy }
        setMetrics(data.projectSlug, {
          uptime: data.uptime,
          responseTime: data.responseTime,
          // Add timestamp for staleness tracking
          lastUpdated: new Date().toISOString(),
          hasReceivedData: true,
        })
      } catch (e) {
        console.error('Failed to parse WebSocket message:', e)
      }
    }

    ws.onclose = () => {
      setConnectionState('disconnected')
      // Attempt reconnection with exponential backoff
      if (reconnectAttemptsRef.current < WS_RECONNECT_MAX_RETRIES) {
        const delay = WS_RECONNECT_BASE_DELAY_MS * Math.pow(2, reconnectAttemptsRef.current)
        reconnectAttemptsRef.current++
        reconnectTimeoutRef.current = setTimeout(connect, delay)
      } else {
        setConnectionState('error')
      }
    }

    ws.onerror = () => {
      ws.close()
    }
  }, [setMetrics, setConnectionState])

  useEffect(() => {
    connect()
    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current)
      }
      if (wsRef.current) {
        wsRef.current.close()
      }
    }
  }, [connect])
}
```

---

### metricsStore Updates

```typescript
// Updated interface
interface MetricsData {
  uptime?: number
  responseTime?: number
  requestsPerMinute?: number
  lastUpdated?: string      // ISO timestamp
  hasReceivedData?: boolean // Track if we've ever received data
}

// Add to store
interface MetricsStore {
  // ... existing fields
  getProjectStatus: (projectSlug: string) => 'live' | 'stale' | 'unavailable'
}

// Implementation
getProjectStatus: (projectSlug) => {
  const data = state.metrics[projectSlug]
  if (!data) return 'unavailable'
  if (!data.hasReceivedData) return 'unavailable'

  if (data.lastUpdated) {
    const hoursSinceUpdate = (Date.now() - new Date(data.lastUpdated).getTime()) / (1000 * 60 * 60)
    if (hoursSinceUpdate > 24) return 'unavailable' // Story 3.5: hide after 24h
    if (hoursSinceUpdate > 0) return 'stale'
  }
  return 'live'
}
```

---

### ProjectCard Integration

```typescript
// Inside ProjectCard.tsx
import { useMetricsStore } from '@/stores/metricsStore'

const ProjectCard = ({ project }: { project: Project }) => {
  const metrics = useMetricsStore((state) => state.metrics[project.slug])
  const connectionState = useMetricsStore((state) => state.connectionState)
  const getProjectStatus = useMetricsStore((state) => state.getProjectStatus)

  const status = getProjectStatus(project.slug)

  // Render logic:
  // - status === 'live': show uptime %, responseTime
  // - status === 'stale': show "Updated X ago"
  // - status === 'unavailable': show "Status unavailable" (only if hasReceivedData === false)
  // - connectionState === 'connecting' && !metrics: show nothing (wait for first data)
}
```

---

### Project Structure — Files to Create/Modify

**New files:**
```
portfolio-fe/src/
├── hooks/
│   └── useWebSocket.ts           ← WebSocket lifecycle hook
│   └── useWebSocket.test.ts     ← Unit tests
├── constants/
│   └── config.ts                ← WS constants (if not exists, create)
```

**Files to modify:**
- `portfolio-fe/src/stores/metricsStore.ts` — add lastUpdated, hasReceivedData fields
- `portfolio-fe/src/components/shared/ProjectCard.tsx` — display live metrics
- `portfolio-fe/src/App.tsx` — add useWebSocket() call (once at root)

**DO NOT modify:**
- Story 3.3 BE code — that's already complete
- Any other components unless specifically needed for metrics display

---

### Architecture Compliance Guardrails

1. **Native WebSocket ONLY** — No libraries. [Source: epics.md#Story-3.4]
2. **useWebSocket single instance** — Called once in App.tsx, not in components. [Source: architecture.md#useWebSocket-hook]
3. **Exponential backoff** — Max 3 retries, base delay configurable. [Source: epics.md#Story-3.4 AC3]
4. **metricsStore single source** — All metrics state in Zustand, no local component state. [Source: architecture.md#useWebSocket-hook]
5. **Graceful degradation** — Static content works when WS fails. [Source: epics.md#Story-3.4 AC4]
6. **Staleness display** — "Updated X ago" shown when data is stale but not dead. [Source: epics.md#Story-3.4 AC5]
7. **No data state** — "Status unavailable" when never received data. [Source: epics.md#Story-3.4 AC6]
8. **VITE_WS_URL env var** — Must use VITE_ prefix for Vite. [Source: architecture.md#VITE_WS_URL]

---

### What NOT to Implement in Story 3.4

- **No polling fallback** — Only reconnection attempts (max 3), then show error state
- **No heartbeat/ping** — Not in AC
- **No authentication on WebSocket** — Stories 5.x handle auth
- **No metrics hiding after 24h** — That's Story 3.5 (Metrics Staleness)
- **No GitHub contributions graph** — That's Story 3.6

---

### Previous Story Intelligence

| Observation from Story 3.3 (BE) | Impact on Story 3.4 (FE) |
|---|---|
| WebSocket endpoint: `/ws/metrics` | Connect to `VITE_WS_URL` env var (default: `ws://localhost:8080/ws/metrics`) |
| Native Spring WebSocket (no STOMP) | Use native browser `WebSocket` API, no libraries |
| Broadcasts `ProjectHealthDto` as JSON | Parse: `{projectSlug, status, uptime, responseTime, lastDeploy}` |
| CORS configured with `setAllowedOrigins("*")` | Development works; production needs specific origin |
| metricsStore already exists | Extend with new fields, don't recreate |

---

### FE Project Context (Existing Patterns)

From Epic 1 stories, established patterns to follow:
- **Component structure:** `src/components/shared/ProjectCard.tsx`
- **Hook pattern:** `src/hooks/useMotion.ts` — custom hook with tests
- **Store pattern:** `src/stores/metricsStore.ts` — Zustand store
- **Constants:** `src/constants/motion.ts` — named exports
- **Testing:** Vitest + React Testing Library + axe for accessibility

---

### References

- [Source: epics.md#Story-3.4] — User story and acceptance criteria
- [Source: epics.md#Story-3.3] — BE WebSocket implementation (completed)
- [Source: architecture.md#useWebSocket-hook] — Single hook pattern requirement
- [Source: architecture.md#VITE_WS_URL] — Environment variable specification
- [Source: architecture.md#MetricsData] — Store structure and data format
- [Source: 3-3-websocket-server-metrics-broadcast-be.md] — BE implementation details

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (2026-03-12)

### Debug Log References

- 2026-03-12: Used native browser WebSocket API per architecture requirements - no external libraries
- 2026-03-12: Implemented exponential backoff reconnection (max 3 retries, base delay 1000ms)
- 2026-03-12: Updated metricsStore with lastUpdated timestamp and hasReceivedData flag for staleness tracking
- 2026-03-12: Added getProjectStatus selector returning 'live' | 'stale' | 'unavailable'
- 2026-03-12: Fixed stale threshold from > 0 to >= 1 hour for accuracy
- 2026-03-12: Updated MetricPair component to support suffix prop for % display

### Completion Notes List

- Created useWebSocket hook for WebSocket lifecycle management
- Added WS_RECONNECT_MAX_RETRIES, WS_RECONNECT_BASE_DELAY_MS, WS_URL constants
- Extended MetricsData interface with lastUpdated and hasReceivedData fields
- Implemented getProjectStatus selector in metricsStore
- Integrated live metrics display in ProjectCard component
- Added graceful degradation handling for connection errors
- Updated MetricPair to support suffix for uptime percentage display
- Fixed AC5: Now shows "Updated X ago" using formatRelativeTime
- Added useWebSocket.test.ts with unit tests
- All tests pass (232 tests)
- Lint passes with 0 errors

### File List

**New files:**
- `portfolio-fe/src/hooks/useWebSocket.ts` — WebSocket lifecycle hook
- `portfolio-fe/src/hooks/useWebSocket.test.ts` — Unit tests for useWebSocket

**Modified files:**
- `portfolio-fe/src/constants/config.ts` — Added WS constants
- `portfolio-fe/src/stores/metricsStore.ts` — Added lastUpdated, hasReceivedData, getProjectStatus
- `portfolio-fe/src/stores/metricsStore.test.ts` — Added tests for new fields
- `portfolio-fe/src/components/shared/ProjectCard.tsx` — Added live metrics display
- `portfolio-fe/src/components/shared/ProjectCard.test.tsx` — Added WebSocket metrics tests
- `portfolio-fe/src/components/shared/MetricPair.tsx` — Added suffix prop support
- `portfolio-fe/src/App.tsx` — Added useWebSocket() call at root

---

## Change Log

- 2026-03-12: Initial implementation — Story 3.4 complete (Date: 2026-03-12)
- 2026-03-13: Code review fixes — AC5 "Updated X ago" format corrected, useWebSocket.test.ts added
