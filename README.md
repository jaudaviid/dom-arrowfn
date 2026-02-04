# DOM -Depth Market Display-

High-performance React trading interface displaying real-time order book depth with optimized rendering.

## Architecture

### State Management
**Primary State:** `DOMSnapshot` object (immutable pattern)
- `levels`: Price level array with bid/ask sizes and counts
- `lastPrice`, `lastSize`: Most recent trade info
- `timestamp`: Update marker for change detection

**Dual State:** Current + Previous snapshot enable flash detection on size changes without expensive diffing.

### Data Flow
1. **useMockWebSocket** → emits DOMSnapshot every 100–250ms (configurable)
2. **TradingLadder** (main) → consumes snapshot, derives sorted levels & max size
3. **PriceRow** → renders individual levels, detects changes via prevLevel
4. **PriceCell** → displays size with proportional bar, count badge, flash animation

### Rendering Optimization

**3-Level Memoization:**
- `PriceCell`: memo with 4-prop comparator (value, flash, count, maxSize)
- `PriceRow`: memo on price, bidSize, askSize only
- `TradingLadder`: useMemo for sortedLevels & maxLevelSize

Shallow comparisons prevent unnecessary re-renders; only changed levels update.

## Scaling to 10+ Workspaces

### WebSocket Channels Strategy
**Single connection, multiplexed channels** instead of N separate connections:
- **Subscribe/unsubscribe:** `{ type: 'subscribe', channel: 'ES', }` → server routes depth updates to listener
- **Fan-in pattern:** All instruments stream on one connection; client demultiplexes by channel ID
- **Bandwidth savings:** ~80% reduction vs. per-workspace connections; latency < 1ms per channel

### Additional Patterns
- **Virtualization:** Replace fixed height with react-window for 100+ price levels
- **Web Workers:** Offload sorting & bar calculations to prevent main-thread blocking
- **Tab management:** Use Context to switch active workspace without re-mounting components

## Error Handling & Data Lag

**Desync Detection:**
- Track `timestamp` delta; flag if > 1s delay
- Add "STALE DATA" warning badge when feed lags

**Recovery:**
- Automatic snapshot reset if lastPrice diverges > 2σ from recent trades
- Exponential backoff for reconnection logic (mock → real WebSocket)

**User Feedback:**
- Toast/banner: "Connection lost" → "Reconnecting..." → "Synced"
- Dim table + disable clicks during desync

## Implementation Highlights

- **Flash animations** on bid/ask size changes (300ms cyan/red pulse)
- **Proportional bars** scaled by maxLevelSize for visual depth ranking
- **Trade counts** (bidCount/askCount) render as badges for order fragmentation insight

## Developer Guidelines

✓ **Constants** at top (COLORS, ANIMATION_DURATION_MS, BASE_PRICE)  
✓ **JSDoc** on all exported functions & hooks  
✓ **Clear naming** (sortedLevels, maxLevelSize, detectChange)  
✓ **Modern patterns** (optional chaining, arrow functions, semantic HTML)  
✓ **Consistent formatting** (2-space indent, no trailing commas on JSX)  

## Usage

```bash
open dom.html
```

Edit `MAX_PRICE_LEVELS`, `BASE_PRICE`, `UPDATE_INTERVAL_MS` for tuning. Extend `useMockWebSocket` to accept real WebSocket URL for production.