# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YoPrint Operations Dashboard - a Missive sidebar integration that displays quotes, sales orders, shipments, and purchase orders from the YoPrint API. Used by Hub City Design to manage apparel decoration operations directly from Missive.

## Architecture

**Single-file Missive integration** (`index.html`) with embedded CSS and JavaScript. No build process, no dependencies, no bundler.

- Connects to YoPrint external API at `https://app.yoprint.com/api/external`
- Uses Bearer token authentication (API key stored via `Missive.storeGet/storeSet`)
- Tabs: Quotes, Sales Orders, Shipments, Purchase Orders
- Client-side search and status filtering
- Uses Missive JS SDK for storage and forms (no localStorage/prompt/confirm)

## Missive Integration

**Required libraries** (loaded from CDN):
- `https://integrations.missiveapp.com/missive.css`
- `https://integrations.missiveapp.com/missive.js`

**Missive APIs used:**
- `Missive.storeGet(key)` - Retrieve stored values (replaces localStorage.getItem)
- `Missive.storeSet(key, value)` - Store values (replaces localStorage.setItem)
- `Missive.openForm({...})` - Display forms/confirmations (replaces prompt/confirm)

## Development

**To test in Missive:** Add as a sidebar integration in Missive settings.

**To test standalone:** Won't work - requires Missive JS SDK context.

## YoPrint API Endpoints Used

| Tab | Endpoint |
|-----|----------|
| Quotes | `/orders?type=quote&limit=50` |
| Sales Orders | `/orders?type=sales&limit=50` |
| Shipments | `/shipments?limit=50` |
| Purchase Orders | `/purchase-orders?limit=50` |

## Key Functions

- `loadData()` - Fetches data from YoPrint API based on current tab
- `renderData()` - Renders filtered items to DOM with search/status filtering
- `openItem(id)` - Opens item in YoPrint web app in new tab
- `resetApiKey()` - Clears stored API key and reloads

## YoPrint API Notes

Refer to the `/yoprint-api-knowledge` skill for comprehensive YoPrint API documentation, including:
- scoped_id_search requires both `type` and `scoped_id` fields
- v2 endpoints require UUIDs, not scoped_ids
- Empty arrays in request body cause 422 errors (omit them instead)
