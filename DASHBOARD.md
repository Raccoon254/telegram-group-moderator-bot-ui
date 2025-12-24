# Group Guard Dashboard

A real-time dashboard for monitoring Telegram Group Guard moderation bot activity.

## Features

### Design
- **Pure Black & Yellow Theme** - Minimal color palette for maximum impact
- **Figtree Font** - Modern, clean typography
- **Lucide Icons** - Beautiful, consistent iconography
- **Curved Cards** - Smooth rounded corners (no shadows, no glass effects)
- **Smooth Animations** - Fade-in, slide-up, and slide-down effects

### Functionality
- **Real-time Updates** - Live SSE connection for instant violation notifications
- **Statistics Dashboard** - Total users, warnings, violations, and bans
- **Recent Violations** - Last 20 violations with user info and timestamps
- **Top Users** - Top 10 users ranked by warning count
- **Live Status Indicator** - Connection status with visual feedback
- **Responsive Design** - Works on all screen sizes

### Attribution
- Created by [Kentom](https://kentom.co.ke)
- Uses Kentom logo in header
- Group Guard bot branding

## Tech Stack
- SvelteKit 2
- Tailwind CSS 4
- Lucide Icons (Svelte)
- TypeScript
- Server-Sent Events (SSE)

## Development

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## API Endpoints Used

- `GET /api/stats` - Overall statistics
- `GET /api/users` - User list with warnings
- `GET /api/logs?limit=20` - Recent violations
- `GET /api/events` - SSE stream for real-time updates

## Color Palette

- Background: `#000000` (Pure Black)
- Primary: `#EAB308` (Yellow-500)
- Text: `#FFFFFF` (White)
- Accents: Yellow with various opacity levels
- Borders: Yellow with 20-40% opacity

## Key Components

### Stats Cards
- 4 cards showing key metrics
- Gradient backgrounds (yellow to black)
- Hover effects with border brightness
- Icons for visual context

### Violations List
- Recent violations in chronological order
- User information and action taken
- Violation type badges
- Relative timestamps

### Top Users
- Ranked list by warning count
- User position badges
- Warning count display

### Real-time Banner
- Appears on new events
- Auto-updates data
- Smooth slide-down animation

## Deployment

The dashboard is ready to deploy to any static hosting platform:
- Vercel
- Netlify
- Cloudflare Pages
- GitHub Pages

Just run `npm run build` and deploy the `.svelte-kit/output` directory.

---

**Created by [Kentom](https://kentom.co.ke)**
