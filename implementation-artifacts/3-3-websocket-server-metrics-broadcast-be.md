# Story 3.3: WebSocket Server — Metrics Broadcast (BE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As the Portfolio FE,
I want to receive live health metric updates via WebSocket,
So that project cards update in real time without browser-side polling.

## Acceptance Criteria

**AC1:** Given a WebSocket client connects to `ws://{host}/ws/metrics`,
Then `MetricsWebSocketHandler` in `platform.websocket` accepts the connection and adds it to the active sessions list

**AC2:** Given a new `ProjectHealth` snapshot is produced (by scheduler or webhook trigger),
Then the updated metrics are broadcast to ALL connected WebSocket sessions within 3 seconds of detection (NFR-P5)

**AC3:** Given a WebSocket client disconnects,
Then the session is removed from the broadcast list — no further messages are sent to it

**AC4:** Given the WebSocket implementation,
Then it uses native WebSocket only — NOT STOMP, NOT SockJS — no `sockjs-client` or `@stomp/stompjs` dependencies anywhere

## Tasks / Subtasks

- [x] Task 1: Create WebSocket configuration (AC: #1, #4)
  - [x] 1.1 Create `src/main/java/dev/chinh/portfolio/platform/websocket/WebSocketConfig.java` — `@EnableWebSocket`, register `MetricsWebSocketHandler` at `/ws/metrics`
  - [x] 1.2 Ensure `spring-boot-starter-websocket` is in pom.xml (already present per Story 3.1 notes)

- [x] Task 2: Create MetricsWebSocketHandler (AC: #1, #2, #3)
  - [x] 2.1 Create `src/main/java/dev/chinh/portfolio/platform/websocket/MetricsWebSocketHandler.java` — extends `TextWebSocketHandler`
  - [x] 2.2 Implement `afterConnectionEstablished(WebSocketSession)` — add session to concurrent Set
  - [x] 2.3 Implement `afterConnectionClosed(WebSocketSession, CloseStatus)` — remove session from Set
  - [x] 2.4 Implement `handleTextMessage(WebSocketSession, TextMessage)` — log but ignore (one-way BE → FE)
  - [x] 2.5 Implement `broadcast(ProjectHealthDto)` — iterate sessions, send JSON via `session.sendMessage()`

- [x] Task 3: Wire MetricsAggregationService to broadcast (AC: #2)
  - [x] 3.1 Inject `MetricsWebSocketHandler` into `MetricsAggregationService`
  - [x] 3.2 Call `broadcast(dto)` after each successful poll in `pollApp()` and after webhook-triggered refresh
  - [x] 3.3 Handle case where no clients connected — skip broadcast silently (no exception)

- [x] Task 4: Add CORS configuration for WebSocket (AC: #1)
  - [x] 4.1 Configure WebSocket handshake CORS to allow FE origin
  - [x] 4.2 Allow FE domain in `WebSocketConfig` — use `setAllowedOrigins()`

- [x] Task 5: Write unit tests for MetricsWebSocketHandler (AC: #1, #2, #3)
  - [x] 5.1 Create `src/test/java/dev/chinh/portfolio/platform/websocket/MetricsWebSocketHandlerTest.java`
  - [x] 5.2 Test: afterConnectionEstablished adds session to Set
  - [x] 5.3 Test: afterConnectionClosed removes session from Set
  - [x] 5.4 Test: broadcast sends message to all connected sessions
  - [x] 5.5 Test: handleTextMessage logs but does not throw (one-way channel)

- [x] Task 6: Write integration test for broadcast flow (AC: #2)
  - [x] 6.1 Test: MetricsAggregationService poll triggers broadcast to connected handler
  - [x] 6.2 Test: no clients connected — poll completes without error

- [x] Task 7: Run full test suite and confirm build (AC: all)
  - [x] 7.1 Run `./mvnw.cmd clean test`
  - [x] 7.2 All previous tests pass (Story 3.1 + 3.2)
  - [x] 7.3 All new WebSocket tests pass

## Dev Notes

### CRITICAL: Files That ALREADY EXIST — Do NOT Recreate

| File | Location | Status |
|------|----------|--------|
| `ProjectHealthDto.java` | `platform/metrics/` | ✅ EXISTS — created in Story 3.1 |
| `MetricsMapper.java` | `platform/metrics/` | ✅ EXISTS — created in Story 3.1 |
| `MetricsAggregationService.java` | `platform/metrics/` | ✅ EXISTS — created in Story 3.1 |
| `ProjectHealth.java` | `platform/metrics/` | ✅ EXISTS — entity with all fields |
| `HealthStatus.java` | `platform/metrics/` | ✅ EXISTS — enum: UP, DOWN |
| `spring-boot-starter-websocket` | `pom.xml` | ✅ EXISTS — Story 3.1 confirmed |

---

### CRITICAL: Architecture Requirements

1. **Native WebSocket Only — NO STOMP, NO SockJS**:
   - Use `spring-boot-starter-websocket` (already in pom.xml)
   - Use `TextWebSocketHandler` (not Spring STOMP)
   - No `sockjs-client` or `@stomp/stompjs` in pom.xml or any FE deps
   - Endpoint: `ws://{host}/ws/metrics` (one-way: BE → FE)

2. **WebSocket Package Location**:
   - Path: `dev.chinh.portfolio.platform.websocket`
   - Files: `WebSocketConfig.java`, `MetricsWebSocketHandler.java`
   - This matches architecture.md: `platform/websocket/`

3. **Broadcast Timing**:
   - Must broadcast within 3 seconds of detection (NFR-P5)
   - Called from `MetricsAggregationService.pollApp()` after successful save
   - Also called from `triggerRefresh()` after webhook-triggered poll

---

### `WebSocketConfig.java` — Complete Implementation

```java
package dev.chinh.portfolio.platform.websocket;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    private final MetricsWebSocketHandler metricsWebSocketHandler;

    public WebSocketConfig(MetricsWebSocketHandler metricsWebSocketHandler) {
        this.metricsWebSocketHandler = metricsWebSocketHandler;
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(metricsWebSocketHandler, "/ws/metrics")
                .setAllowedOrigins("*"); // Configure for production: specific FE domain
    }
}
```

**Note on CORS:** For production, replace `setAllowedOrigins("*")` with specific FE domain(s). During local development, `*` is acceptable.

---

### `MetricsWebSocketHandler.java` — Complete Implementation

```java
package dev.chinh.portfolio.platform.websocket;

import dev.chinh.portfolio.platform.metrics.ProjectHealthDto;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.IOException;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class MetricsWebSocketHandler extends TextWebSocketHandler {

    private static final Logger log = LoggerFactory.getLogger(MetricsWebSocketHandler.class);

    private final Set<WebSocketSession> sessions = ConcurrentHashMap.newKeySet();
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        sessions.add(session);
        log.debug("WebSocket connected: {}", session.getId());
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        sessions.remove(session);
        log.debug("WebSocket disconnected: {}", session.getId());
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) {
        // One-way: BE → FE only. Incoming messages are ignored.
        log.debug("Received message from {} (ignored - one-way channel): {}", session.getId(), message.getPayload());
    }

    public void broadcast(ProjectHealthDto dto) {
        if (sessions.isEmpty()) {
            log.debug("No WebSocket clients connected, skipping broadcast");
            return;
        }

        String json;
        try {
            json = objectMapper.writeValueAsString(dto);
        } catch (IOException e) {
            log.error("Failed to serialize ProjectHealthDto for broadcast", e);
            return;
        }

        TextMessage message = new TextMessage(json);
        for (WebSocketSession session : sessions) {
            try {
                if (session.isOpen()) {
                    session.sendMessage(message);
                }
            } catch (IOException e) {
                log.warn("Failed to send WebSocket message to {}: {}", session.getId(), e.getMessage());
            }
        }
        log.debug("Broadcast metrics to {} clients: {}", sessions.size(), dto.projectSlug());
    }
}
```

---

### Integrating Broadcast into MetricsAggregationService

Modify `src/main/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationService.java`:

```java
@Service
public class MetricsAggregationService {

    private final ProjectHealthRepository repository;
    private final DemoAppRegistry registry;
    private final RestClient restClient;
    private final MetricsMapper mapper;
    private final MetricsWebSocketHandler webSocketHandler; // NEW

    public MetricsAggregationService(ProjectHealthRepository repository,
                                     DemoAppRegistry registry,
                                     RestClient restClient,
                                     MetricsMapper mapper,
                                     MetricsWebSocketHandler webSocketHandler) { // NEW
        this.repository = repository;
        this.registry = registry;
        this.restClient = restClient;
        this.mapper = mapper;
        this.webSocketHandler = webSocketHandler;
    }

    // In pollApp(), after successful save:
    ProjectHealth saved = repository.save(record);
    // Broadcast to all connected clients
    ProjectHealthDto dto = mapper.toDto(saved);
    webSocketHandler.broadcast(dto);

    // Same in triggerRefresh() after pollApp()
}
```

---

### Testing Pattern — WebSocket Handler Unit Tests

```java
@ExtendWith(MockitoExtension.class)
class MetricsWebSocketHandlerTest {

    @InjectMocks
    private MetricsWebSocketHandler handler;

    @Mock private WebSocketSession session;

    @Test
    void afterConnectionEstablished_addsSessionToSet() {
        handler.afterConnectionEstablished(session);
        // Verify session was added to internal Set
        // Use reflection or test package-private method
    }

    @Test
    void afterConnectionClosed_removesSessionFromSet() {
        handler.afterConnectionEstablished(session);
        handler.afterConnectionClosed(session, CloseStatus.NORMAL);
        // Verify session removed
    }

    @Test
    void broadcast_sendsMessageToAllSessions() throws IOException {
        WebSocketSession session1 = mock(WebSocketSession.class);
        WebSocketSession session2 = mock(WebSocketSession.class);
        when(session1.isOpen()).thenReturn(true);
        when(session2.isOpen()).thenReturn(true);

        handler.afterConnectionEstablished(session1);
        handler.afterConnectionEstablished(session2);

        ProjectHealthDto dto = new ProjectHealthDto("wallet-app", "UP", new BigDecimal("100.00"), 150, null, Instant.now());
        handler.broadcast(dto);

        verify(session1).sendMessage(any(TextMessage.class));
        verify(session2).sendMessage(any(TextMessage.class));
    }

    @Test
    void broadcast_noSessions_doesNotThrow() {
        ProjectHealthDto dto = new ProjectHealthDto("wallet-app", "UP", new BigDecimal("100.00"), 150, null, Instant.now());
        assertDoesNotThrow(() -> handler.broadcast(dto));
    }
}
```

---

### Project Structure — New Files to Create

**Repository root:** `I:/portfolio-v2/portfolio-platform/` (separate git root)

```
src/main/java/dev/chinh/portfolio/
└── platform/
    └── websocket/
        ├── WebSocketConfig.java           ← @EnableWebSocket, /ws/metrics endpoint
        └── MetricsWebSocketHandler.java   ← TextWebSocketHandler: sessions + broadcast

src/test/java/dev/chinh/portfolio/
└── platform/
    └── websocket/
        └── MetricsWebSocketHandlerTest.java ← Unit tests
```

**Files to modify:**
- `src/main/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationService.java` — inject `MetricsWebSocketHandler`, call `broadcast()` after save

**DO NOT create:**
- `WebSocketConfig` in any other package — only `platform.websocket`
- STOMP or SockJS configurations
- Any `*WebSocketClient` classes — that's Story 3.4 (FE)

---

### Architecture Compliance Guardrails

1. **Native WebSocket ONLY** — No STOMP, no SockJS. [Source: epics.md#Story-3.3 AC4]
2. **One-way broadcast** — BE → FE. `handleTextMessage` logs but ignores. [Source: architecture.md#WebSocket-public-]
3. **Broadcast within 3 seconds** — Called synchronously after DB save. [Source: epics.md#Story-3.3 AC2]
4. **Concurrent session management** — Use `ConcurrentHashMap.newKeySet()` for thread-safety. [Source: architecture.md#MetricsWebSocketHandler]
5. **Empty sessions handling** — Skip broadcast silently, no exception. [Source: epics.md#Story-3.3 AC3]
6. **CORS configuration** — Set allowed origins in `WebSocketConfig`, not `*` in production.
7. **DTO uses String status** — `ProjectHealthDto.status` is String, not enum (Story 3.1 design for easy Jackson serialization).

---

### What NOT to Implement in Story 3.3

- **No WebSocket client in FE** — That's Story 3.4 (WebSocket Client & Metrics Cards)
- **No STOMP or SockJS** — Explicitly forbidden by AC4
- **No authentication on WebSocket** — Stories 5.x handle auth
- **No heartbeat/ping-pong** — Optional future enhancement, not in AC
- **No reconnection logic in BE** — That's FE concern (Story 3.4)

---

### Previous Story Intelligence

| Observation from Story 3.1 | Impact on Story 3.3 |
|---|---|
| `ProjectHealthDto` created with String status field | Use same DTO for broadcast — no conversion needed |
| `MetricsMapper.toDto()` exists | Reuse for broadcast transformation |
| `MetricsAggregationService` is sole writer to `project_health` | Inject handler into this service to trigger broadcast after write |
| `spring-boot-starter-websocket` in pom.xml | Ready to use — no new dependency |
| `DemoAppRegistry` in `shared.config` | Already loaded and available for any config needs |
| 17 non-Docker tests pass | Add WebSocket tests to existing test suite |

---

### References

- [Source: epics.md#Story-3.3] — User story and acceptance criteria
- [Source: architecture.md#WebSocket-public-] — Endpoint: `WS /ws/metrics`
- [Source: architecture.md#MetricsWebSocketHandler] — `TextWebSocketHandler`, sessions + broadcast
- [Source: architecture.md#Complete-Project-Tree-portfolio-platform] — File locations: `platform/websocket/`
- [Source: architecture.md#Data-Flow-Metrics-Pipeline] — Broadcast flow: `@Scheduled → MetricsAggregationService → MetricsWebSocketHandler.broadcast()`
- [Source: 3-1-health-metrics-polling-pipeline-be.md] — `ProjectHealthDto`, `MetricsMapper` already exist
- [Source: epics.md#Story-3.3 AC4] — "uses native WebSocket only — NOT STOMP, NOT SockJS"

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (2026-03-12)

### Debug Log References

- WebSocket session management uses `ConcurrentHashMap.newKeySet()` for thread-safety
- Jackson serialization of `Instant` requires `jackson-datatype-jsr310` module (auto-configured in Spring Boot)

### Completion Notes List

- **Task 1:** Created `WebSocketConfig.java` with `@EnableWebSocket` and endpoint at `/ws/metrics`
- **Task 2:** Created `MetricsWebSocketHandler.java` with session management and broadcast
- **Task 3:** Wired `MetricsWebSocketHandler` into `MetricsAggregationService` - broadcasts on both success and failure
- **Task 4:** CORS configured with `setAllowedOrigins("*")` for development
- **Task 5:** Created unit tests for `MetricsWebSocketHandler` (4 tests)
- **Task 6:** Updated `MetricsAggregationServiceTest` to include WebSocket broadcast verification (10 tests total)
- **Task 7:** All 34 unit tests pass (excluding integration tests requiring Docker)

### File List

**New files:**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/websocket/WebSocketConfig.java`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/websocket/MetricsWebSocketHandler.java`
- `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/websocket/MetricsWebSocketHandlerTest.java`

**Modified files:**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationService.java` - injected WebSocket handler, added broadcast calls
- `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationServiceTest.java` - added mapper and WebSocket mocks

