# Financial Tools Hub

A collection of real-time financial visualization tools for analyzing Binance cryptocurrency markets.

## Overview

This project provides browser-based tools for visualizing order book data and market activity on Binance spot markets. All tools connect directly to Binance WebSocket streams for real-time data.

## Tools

### 1. Order Book Visualizer (`order_book_visualizer.html`)

Real-time heatmap visualization of a single trading pair's order book.

**Features:**
- High-frequency pixel-by-pixel order book history
- Two visualization modes:
  - **Book Mode**: Full order book heatmap showing all price levels
  - **Walls Mode**: Filtered view showing only large orders (10x average volume)
- Event detection in Walls mode:
  - **Eaten Walls**: Large orders consumed by trades (≥80% traded through)
  - **Support Bounces**: Price reversal at buy walls
  - **Resistance Bounces**: Price reversal at sell walls
- Interactive controls: zoom (mouse wheel), pan (click-drag)
- Symbol search with autocomplete

**How it works:**
1. Fetches symbol list from Binance API
2. Connects to `@depth@100ms` WebSocket for order book updates
3. Connects to `@aggTrade` WebSocket for trade data (Walls mode only)
4. Synchronizes snapshot with buffered updates
5. Renders each update as a vertical pixel column
6. Tracks wall lifecycle and price interactions

### 2. Multi-Pair Wall Tracker (`multipair_wall_tracker.html`)

Simultaneous visualization of multiple USDC-quoted trading pairs.

**Features:**
- Automatically discovers all USDC spot pairs with perpetual futures contracts
- Grid layout displaying 3 charts per row
- Per-symbol wall detection and event tracking
- Independent zoom/pan controls for each chart
- Connection status modal showing sync state for all symbols
- Same event detection as Order Book Visualizer

**How it works:**
1. Queries Binance API for spot and futures symbols
2. Filters for USDC-quoted pairs with futures contracts
3. Creates combined WebSocket stream for all pairs
4. Manages independent order book state per symbol
5. Renders each symbol in its own canvas with shared event logic

## Architecture

### Data Management

Both tools use a similar data manager pattern:

```javascript
createOrderBookManager()
  - Manages WebSocket connections
  - Maintains order book state (Map structures)
  - Handles snapshot synchronization
  - Tracks active walls and events
  - Provides state updates via callback
```

**Key algorithms:**
- **Wall Detection**: Volume > (average volume × 10)
- **Eaten Wall**: ≥80% of wall volume traded through before removal
- **Bounce Detection**: Price approaches within 5 ticks, reverses by 7 ticks

### Visualization

Canvas-based rendering with two layers:
- **Main canvas**: Order book heatmap (shifted left each frame)
- **Overlay canvas**: Price line, crosshair, event markers

**Color mapping**: HSL gradient from blue (low volume) to red (high volume) using logarithmic scaling

### Synchronization

Implements Binance's recommended sync procedure:
1. Buffer WebSocket updates
2. Fetch REST snapshot
3. Drop buffered updates older than snapshot
4. Apply remaining buffered updates
5. Process live updates

## Usage

1. Open any HTML file in a modern browser
2. Select a trading pair (Order Book Visualizer only)
3. Click "Connect"
4. Use mouse wheel to zoom, click-drag to pan
5. Click info button (ⓘ) for detailed help

## Technical Details

**Dependencies:**
- Tailwind CSS (CDN)
- Native WebSocket API
- Canvas 2D API

**Data sources:**
- `wss://stream.binance.com:9443` - WebSocket streams
- `https://api.binance.com/api/v3` - REST API

**Performance:**
- 100ms update frequency
- Hardware-accelerated canvas rendering
- Efficient Map-based order book storage
- Incremental rendering (shift + draw new column)

## File Structure

```
fin-tools-hub/
├── index.html                    # Landing page with tool links
├── order_book_visualizer.html    # Single-pair visualizer
├── multipair_wall_tracker.html   # Multi-pair visualizer
└── README.md                     # This file
```

## Browser Compatibility

Requires modern browser with support for:
- ES6+ JavaScript (Map, async/await, arrow functions)
- WebSocket API
- Canvas 2D API
- ResizeObserver API

Tested on Chrome, Firefox, Safari, Edge (latest versions).
