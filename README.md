# GitHub Stats API

A Next.js 14 App Router API that provides comprehensive GitHub statistics including lifetime commit counts and detailed repository breakdowns for GitHub users.

## Features

- **Single Endpoint**: `GET /api/commits/{username}` returns comprehensive GitHub statistics
- **Detailed Repository Breakdown**: Shows owned repos, original repos (non-forks), and private repos
- **GitHub GraphQL Integration**: Fetches commits year-by-year and detailed repository statistics
- **Smart Caching**: 24-hour in-memory cache per user to avoid rate limits
- **CORS Support**: Ready for cross-origin requests (GitHub Pages, etc.)
- **Error Handling**: Comprehensive error responses with appropriate HTTP status codes
- **Vercel Ready**: Optimized for serverless deployment

## API Response

### Success Response
```json
{
  "commits": 1308,
  "total_repos": 42,
  "repos_count": {
    "owned": 28,
    "original": 35,
    "private": 12
  }
}
```

**Response Fields:**
- `commits`: Total lifetime commit count across all years
- `total_repos`: Total repository count (public + private)
- `repos_count.owned`: Repositories where user is the owner (excludes collaborations)
- `repos_count.original`: Non-forked repositories (original work)  
- `repos_count.private`: Private repositories

### Error Response
```json
{
  "error": "User not found"
}
```

## Setup

### 1. Clone and Install

```bash
git clone <repository-url>
cd git-stats-api
npm install
```

### 2. Environment Variables

Create a `.env.local` file in the root directory:

```bash
cp .env.local.example .env.local
```

Add your GitHub Personal Access Token:

```env
GITHUB_TOKEN=your_github_personal_access_token_here
```

#### Creating a GitHub Token

1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Click "Generate new token (classic)"
3. Give it a descriptive name like "Git Stats API"
4. Select scopes: `public_repo` (or `repo` if you need private repo access)
5. Click "Generate token"
6. Copy the token and add it to your `.env.local` file

### 3. Run Locally

```bash
npm run dev
```

The API will be available at `http://localhost:3000`

Test it: `http://localhost:3000/api/commits/octocat`

## Deployment

### Deploy to Vercel

1. **Via Vercel CLI** (recommended):
   ```bash
   npm i -g vercel
   vercel
   ```

2. **Via GitHub Integration**:
   - Push your code to GitHub
   - Connect your repository to Vercel
   - Add `GITHUB_TOKEN` to your Vercel environment variables

3. **Environment Variables**:
   - In your Vercel dashboard, go to Settings → Environment Variables
   - Add: `GITHUB_TOKEN` with your GitHub token value

### Manual Deploy
```bash
npm run build
npm start
```

## Usage Examples

### JavaScript/Fetch
```javascript
const response = await fetch('https://your-api.vercel.app/api/commits/octocat');
const data = await response.json();
console.log(`Total commits: ${data.commits}`);
console.log(`Total repos: ${data.total_repos}`);
console.log(`Owned: ${data.repos_count.owned}, Original: ${data.repos_count.original}, Private: ${data.repos_count.private}`);
```

### cURL
```bash
curl https://your-api.vercel.app/api/commits/octocat
```

### React Component
```tsx
import { useEffect, useState } from 'react';

function GitHubStats({ username }) {
  const [stats, setStats] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`https://your-api.vercel.app/api/commits/${username}`)
      .then(res => res.json())
      .then(data => {
        setStats(data);
        setLoading(false);
      });
  }, [username]);

  if (loading) return <div>Loading...</div>;
  return (
    <div>
      <div>{stats.commits} commits • {stats.total_repos} repositories</div>
      <div>
        Owned: {stats.repos_count.owned} | 
        Original: {stats.repos_count.original} | 
        Private: {stats.repos_count.private}
      </div>
    </div>
  );
}
```

## Architecture

```
lib/
├── github.ts       # GitHub GraphQL client
├── cache.ts        # In-memory caching system  
└── utils.ts        # Utility functions (CORS, number formatting)

app/api/commits/[username]/
└── route.ts        # Main API endpoint

types/
└── index.ts        # TypeScript type definitions
```

## How It Works

1. **User Request**: Client requests `/api/commits/{username}`
2. **Cache Check**: System checks if data is cached and fresh (< 24 hours)
3. **GitHub Query**: If not cached, queries GitHub GraphQL API:
   - Fetches user's account creation date
   - Loops through each year from creation to present
   - Sums `contributionCalendar.totalContributions` for each year
   - Fetches detailed repository statistics:
     - Total repositories (public + private)
     - Owned repositories (user is owner)
     - Original repositories (non-forks)
     - Private repositories
4. **Cache & Return**: Stores comprehensive stats in memory cache and returns detailed data

## Rate Limits

- **GitHub API**: 5000 requests/hour with personal access token
- **Caching**: 24-hour cache per user significantly reduces API calls
- **Optimization**: Year-by-year querying minimizes data transfer

## Error Handling

The API handles various error scenarios:

- **Missing Username**: 400 Bad Request
- **User Not Found**: 500 with descriptive error
- **Invalid Token**: 500 with GitHub API error
- **Network Issues**: 500 with connection error
- **Rate Limiting**: 500 with GitHub rate limit error

## CORS Configuration

The API includes CORS headers for cross-origin requests:

```javascript
'Access-Control-Allow-Origin': '*'
'Access-Control-Allow-Methods': 'GET, OPTIONS'
'Access-Control-Allow-Headers': 'Content-Type'
```

Perfect for integration with GitHub Pages, portfolios, and other static sites.

## Utilities

### Number Formatting
```javascript
import { formatNumber } from './lib/utils';

formatNumber(1308);     // "1.3k"
formatNumber(2500000);  // "2.5M"
formatNumber(42);       // "42"

// Example usage with API response
const { commits, total_repos, repos_count } = await fetch('/api/commits/username').then(r => r.json());
console.log(`${formatNumber(commits)} commits • ${formatNumber(total_repos)} total repos`);
console.log(`${formatNumber(repos_count.owned)} owned • ${formatNumber(repos_count.original)} original • ${formatNumber(repos_count.private)} private`);
```

## Development

### Project Structure
- Next.js 14 with App Router
- TypeScript for type safety
- No external dependencies beyond Next.js
- Serverless-ready architecture

### Local Development
```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

## License

MIT License - feel free to use in your projects!