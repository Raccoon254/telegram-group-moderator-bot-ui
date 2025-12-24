# Group Guard Dashboard

A modern, real-time web dashboard for monitoring and managing the Group Guard Telegram moderation bot. This application provides comprehensive insights into group moderation activities, violation tracking, and user management through an intuitive web interface.

## Overview

Group Guard is an AI-powered Telegram moderation bot that automatically detects and removes inappropriate content to maintain healthy group environments. This dashboard provides real-time visibility into the bot's operations, including violation detection, user warnings, and ban management.

## Features

### Real-time Monitoring
- Live Server-Sent Events (SSE) connection for instant updates
- Real-time activity banner showing recent moderation actions
- Auto-refresh of statistics when new violations occur
- Connection status indicator

### Statistics Dashboard
- Total users tracked
- Total warnings issued
- Total violations detected
- Total users banned
- Color-coded stat cards with trend indicators

### Violations Management
- Recent violations feed with detailed information
- User information with DiceBear avatars
- Violation type classification (sexual content, spam, phone numbers)
- Action taken indicators (warned, banned, kicked)
- Timestamp with relative time display
- Pagination support (10 violations per page)

### User Rankings
- Top violators leaderboard
- Warning count tracking
- User profile information
- Pagination support (10 users per page)

### Modern UI/UX
- Clean white theme with yellow accent colors
- Figtree font family for modern typography
- Smooth animations and transitions
- Fully responsive design
- Curved cards with subtle borders
- Hover effects and interactive elements
- Loading states

## Technology Stack

### Frontend
- **SvelteKit 2** - Modern web framework with SSR support
- **Svelte 5** - Reactive UI framework with runes API
- **TypeScript** - Type-safe JavaScript
- **Lucide Icons** - Beautiful, consistent icon set
- **CSS Custom Properties** - Theme management

### API Integration
- **REST API** - For fetching statistics, users, and violations
- **Server-Sent Events (SSE)** - For real-time updates
- **DiceBear API** - For generating user avatars

## Project Structure

```
telegram-bot-updates/
├── src/
│   ├── routes/
│   │   ├── +layout.svelte      # Root layout with global styles
│   │   ├── +page.svelte         # Main dashboard page
│   │   └── layout.css           # Global CSS with variables
│   └── lib/
├── static/
│   ├── bot.jpg                  # Bot logo/icon
│   └── robots.txt
├── package.json
├── svelte.config.js
├── tsconfig.json
└── vite.config.ts
```

## Installation

### Prerequisites
- Node.js 18 or higher
- npm or pnpm

### Setup Instructions

1. Clone the repository:
```bash
git clone <repository-url>
cd telegram-bot-updates
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Open your browser and navigate to:
```
http://localhost:5173
```

## API Integration

The dashboard connects to the Group Guard API at:
```
https://telegram-group-moderator-bot.onrender.com
```

### API Endpoints Used

#### GET /api/stats
Fetches overall moderation statistics.

**Response:**
```json
{
  "success": true,
  "data": {
    "totalUsers": 100,
    "totalWarnings": 45,
    "totalViolations": 60,
    "totalBans": 15,
    "totalWarningsIssued": 45
  }
}
```

#### GET /api/users
Fetches all users with warning counts.

**Response:**
```json
{
  "success": true,
  "count": 25,
  "data": [
    {
      "id": 1,
      "telegramId": "123456789",
      "username": "user123",
      "firstName": "John",
      "warningCount": 2
    }
  ]
}
```

#### GET /api/logs?limit=50
Fetches recent violations with user information.

**Query Parameters:**
- `limit` - Number of violations to fetch (default: 50)

**Response:**
```json
{
  "success": true,
  "count": 10,
  "data": [
    {
      "id": 1,
      "userId": 1,
      "message": "inappropriate content",
      "violationType": "sexual",
      "actionTaken": "warned",
      "createdAt": "2025-12-24T12:20:02.786Z",
      "user": {
        "telegramId": "123456789",
        "username": "user123",
        "firstName": "John"
      }
    }
  ]
}
```

#### GET /api/events (SSE)
Subscribes to real-time violation events.

**Event Types:**
- `connected` - Initial connection confirmation
- `warning` - Warning issued to user
- `ban` - User banned from group
- `violation` - New violation detected

**Example Event:**
```json
{
  "type": "warning",
  "data": {
    "user": {
      "telegramId": "123456789",
      "username": "user123",
      "firstName": "John"
    },
    "message": "violating message content",
    "violationType": "sexual",
    "action": "warned",
    "timestamp": "2025-12-24T12:20:00.000Z"
  }
}
```

## Development

### Available Scripts

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run type checking
npm run check

# Run linter
npm run lint
```

### Environment Variables

No environment variables are required. The API base URL is hardcoded in the application. To change it, update the `API_BASE` constant in `src/routes/+page.svelte`:

```typescript
const API_BASE = 'https://telegram-group-moderator-bot.onrender.com';
```

## Design System

### Color Palette

```css
--color-primary: #f59e0b;        /* Amber-500 */
--color-primary-dark: #d97706;   /* Amber-600 */
--color-primary-light: #fbbf24;  /* Amber-400 */
--color-bg: #fafafa;             /* Gray-50 */
--color-bg-alt: #ffffff;         /* White */
--color-surface: #ffffff;        /* White */
--color-border: #e5e7eb;         /* Gray-200 */
--color-text: #111827;           /* Gray-900 */
--color-text-muted: #6b7280;     /* Gray-500 */
--color-text-light: #9ca3af;     /* Gray-400 */
--color-success: #10b981;        /* Green-500 */
--color-warning: #f59e0b;        /* Amber-500 */
--color-danger: #ef4444;         /* Red-500 */
```

### Typography

- **Font Family:** Figtree (Google Fonts)
- **Weights:** 300, 400, 500, 600, 700, 800, 900

### Border Radius

- Standard: 16px
- Large: 24px
- Pills/Badges: 100px

## Deployment

### Build for Production

```bash
npm run build
```

The built files will be in the `.svelte-kit/output` directory.

### Deployment Platforms

This SvelteKit application can be deployed to:

- **Vercel** - Zero-config deployment with automatic adapter
- **Netlify** - SvelteKit adapter available
- **Cloudflare Pages** - Edge deployment
- **Node.js** - Using adapter-node
- **Static Hosting** - Using adapter-static (requires configuration)

### Example: Deploy to Vercel

1. Install Vercel CLI:
```bash
npm i -g vercel
```

2. Deploy:
```bash
vercel
```

## Features in Detail

### Pagination

Both violations and top users lists support pagination:
- 10 items per page
- Previous/Next navigation buttons
- Current page indicator
- Disabled state for boundary buttons
- Auto-reset to page 1 on new data

### Avatar Generation

User avatars are generated using DiceBear's Avataaars style:
```
https://api.dicebear.com/7.x/avataaars/svg?seed={username}
```

### Relative Time Display

Timestamps are displayed in relative format:
- "Just now" - Less than 1 minute
- "Xm ago" - Minutes
- "Xh ago" - Hours
- "Xd ago" - Days

### Error Handling

- Graceful fallbacks for missing user data
- Auto-reconnect for SSE disconnections
- Console error logging for debugging

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Modern mobile browsers

## Performance

- Optimized bundle size with code splitting
- Lazy loading of components
- Efficient re-rendering with Svelte reactivity
- Minimal JavaScript footprint

## License

This project is created by Kentom (https://kentom.co.ke).

## Support

For issues or questions, visit https://kentom.co.ke

## Acknowledgments

- Group Guard API for providing moderation data
- DiceBear for avatar generation
- Lucide for icon library
- Google Fonts for Figtree typeface
