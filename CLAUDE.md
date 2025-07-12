# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GeoguessMaster is a Vue 3 + TypeScript geolocation guessing game (similar to GeoGuessr) with real-time multiplayer support via Firebase. The project is no longer maintained - users are directed to https://geoguesscloud.com/.

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (Vite)
npm run dev

# Build for production (runs TypeScript check first)
npm run build

# Preview production build
npm run preview

# Run tests with coverage
npm test

# Run linting
npm run lint          # ESLint for JS/Vue files
npm run lint:style    # Stylelint for CSS/SCSS/Vue styles

# Firebase functions (in /functions directory)
cd functions
npm run build        # Build TypeScript functions
npm run lint         # Lint functions code
```

## Environment Setup

Create `.env.development.local` and `.env.production.local` files with:

```env
VITE_API_KEY=your_google_maps_api_key
VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
VITE_FIREBASE_DATABASE_URL=your_database_url
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FIREBASE_MEASUREMENT_ID=your_measurement_id
```

## Architecture Overview

### Frontend Architecture

The frontend uses Vue 3 with Composition API (`<script setup>` syntax) and is organized as follows:

- **State Management**: Pinia stores in `/src/stores/`
  - `gameSettings`: Game configuration (map, mode, player info, room settings)
  - `inGame`: Active game state (locations, scores, rounds, multiplayer sync)
  - `device`: Responsive device type tracking

- **Routing**: Vue Router with two main views:
  - `HomeView`: Game setup and room creation/joining
  - `GameView`: Active game with StreetView and map components

- **Key Components**:
  - `StreetView.vue`: Google Street View integration with countdown timer
  - `GameMap.vue`: Interactive map for placing guesses
  - `ScoreBoard.vue`: Results display after each round
  - `MultiPlayerList.vue`: Real-time player status in multiplayer games

### Multiplayer Architecture

1. **Room Creation**: Owner creates room with settings stored in Firebase Realtime Database at `/games/{roomId}`
2. **Player Joining**: Players join using 6-digit room code
3. **Real-time Sync**: Firebase listeners synchronize:
   - Street view locations for each round
   - Player guesses and scores
   - Round progression
   - Player connection status
4. **Cleanup**: Cloud Function `deleteExpiredRooms` runs weekly to remove rooms older than 24 hours

### Data Flow Patterns

- **Single Player**: Local state management with Pinia, no Firebase interaction
- **Multiplayer**: Firebase Realtime Database as source of truth, with local Pinia stores mirroring state
- **Precision Timing**: Uses `performance.now()` for accurate countdown timers
- **Error Handling**: Try-catch blocks around Firebase operations with console logging

## Testing

Tests use Vitest with Vue Test Utils:

```bash
# Run tests in watch mode
npm run test:unit

# Run tests once with coverage
npm test
```

Test files are located in `/src/__tests__/unit/components/` with `.test.ts` extension. The setup includes Google Maps API mocks.

## Code Style

- **TypeScript**: Full type coverage with strict mode
- **Vue**: Composition API with `<script setup>` syntax
- **Styling**: SCSS modules with BEM-like naming conventions
- **Pre-commit**: Husky runs ESLint and Stylelint via lint-staged

## Important Architectural Decisions

1. **Firebase Realtime Database** chosen for low-latency multiplayer requirements
2. **Vite** for fast development experience with HMR
3. **Progressive Web App** support for mobile users
4. **Code splitting** via dynamic imports for route components
5. **Custom Pinia reset plugin** using lodash cloneDeep for state management
6. **Device-aware UI** with responsive layouts for mobile/tablet/desktop