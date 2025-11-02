# Comprehensive KDP Research Tool - Coding Plan

## Executive Summary

This document outlines a complete coding plan for building an all-in-one Amazon KDP (Kindle Direct Publishing) research application that combines the functionality of Publisher Rocket, KDP Spy, Titans Pro, BookBeam, KindleTrends, and other market research tools. The application will be available in two versions:

1. **Web-Based Version** - Free hosted solution using modern web technologies
2. **Desktop Version** - Rust + Tauri offline application for enhanced privacy and performance

---

## 1. Core Feature Set

### 1.1 Keyword Research Module
**Features:**
- Amazon autocomplete suggestion expansion (A-Z, 0-9 prefixes/suffixes)
- Estimated search volume calculation
- Keyword competition scoring (0-100)
- Keyword difficulty analysis (SERP intensity, usage frequency)
- Reverse ASIN keyword extraction
- Related keywords and long-tail variations
- Amazon Ads (AMS) keyword generation
- Keyword clustering and grouping
- Export to CSV/Excel

**Data Sources:**
- Amazon autocomplete API (unofficial)
- Web scraping Amazon search results
- Optional: DataForSEO API, Google Keyword Planner API
- Keepa API for historical data

### 1.2 Category Analysis Module
**Features:**
- Complete Amazon category database (19,000+)
- Category search and filtering
- Category hierarchy visualization
- Bestseller rank requirements by category
- Daily sales estimates for Top 10, Top 50, Top 100
- Category difficulty scoring algorithm
- Ghost category detection (categories that don't award badges)
- Duplicate category identification
- Seasonal trend analysis (12-month historical data)
- Optimal launch timing recommendations
- Category comparison tool

**Metrics Tracked:**
- #1 and #100 book BSR in category
- Number of books published (last 30/90 days)
- Average price in category
- Average review count
- Category growth trend (%)
- Market saturation index

### 1.3 Competitor Intelligence Module
**Features:**
- Book data extraction (ASIN, title, author, price, BSR, ratings)
- Review analysis and sentiment scoring
- Review text mining (common phrases, complaints, praise)
- Competitor keyword extraction
- Listing quality scoring
- Cover image analysis and trends
- Price positioning analysis
- Market gap identification
- Top performers tracking

**Analysis Capabilities:**
- SERP (Search Engine Results Page) analysis
- Top 10/20/50/100 analysis for any search term
- Competitor monitoring dashboard
- Historical data tracking

### 1.4 BSR to Sales Conversion
**Features:**
- Configurable power-law conversion formulas
- Format-specific calculations (Kindle, Paperback, Hardcover, Audio)
- Multi-marketplace support (US, UK, DE, FR, ES, IT, CA, AU)
- Revenue estimation based on price and royalty rate
- Historical sales trend visualization
- Daily/weekly/monthly sales projections

**Default Formulas:**
```
Kindle: sales = 5500 × BSR^(-0.83)
Paperback: sales = 2600 × BSR^(-0.75)
Hardcover: sales = 1800 × BSR^(-0.70)
Audiobook: sales = 1200 × BSR^(-0.65)
```

### 1.5 Market Intelligence & Niche Discovery
**Features:**
- Niche discovery wizard
- Demand vs. opportunity matrix
- Market validation scoring (0-100)
- Success probability calculator
- Niche stress testing
- Competitive landscape mapping
- Trend detection (growth, stable, declining)
- Seasonality analysis
- Market saturation warnings

**Scoring Algorithm:**
- Search volume weight: 25%
- Competition level: 30%
- Average BSR: 20%
- Price point viability: 15%
- Review velocity: 10%

### 1.6 Listing Optimization
**Features:**
- Title optimizer (keyword density, character limits)
- Subtitle/series name suggestions
- Description builder with keyword tracking
- 7-keyword backend optimizer
- Author name optimization
- A+ Content suggestions
- Bullet point generator
- Listing score analysis (0-100)
- Keyword usage heatmap
- Character counter for all fields

### 1.7 Amazon Ads (AMS) Tools
**Features:**
- Sponsored Products keyword generator
- Auto-targeting suggestions
- Product targeting (ASIN lists)
- Competitor ASIN targeting
- Author/brand targeting
- Negative keyword suggestions
- Bid amount recommendations
- Campaign structure templates
- Keyword grouping for ad groups

---

## 2. Technical Architecture

### 2.1 Technology Stack Options

#### Option A: Web-Based Application (Free Hosted)

**Frontend:**
- **Framework:** React 18+ with TypeScript
- **UI Library:** Tailwind CSS + shadcn/ui components
- **State Management:** Zustand or Redux Toolkit
- **Data Visualization:** Recharts, Chart.js, or D3.js
- **Table Component:** TanStack Table (React Table v8)
- **Forms:** React Hook Form + Zod validation
- **API Client:** Axios or TanStack Query

**Backend:**
- **Runtime:** Node.js 20+ or Bun
- **Framework:** Express.js or Fastify (lightweight) OR Hono (ultra-fast)
- **API Design:** REST API with OpenAPI/Swagger documentation
- **Authentication:** JWT tokens + refresh tokens
- **Rate Limiting:** express-rate-limit or rate-limiter-flexible

**Database:**
- **Primary:** PostgreSQL 15+ (with pgvector for semantic search)
- **Cache:** Redis for session management and API response caching
- **Search:** PostgreSQL full-text search or Meilisearch

**Web Scraping:**
- **Library:** Playwright or Puppeteer (headless browser)
- **HTML Parsing:** Cheerio or jsdom
- **Proxy Management:** rotating-proxy-agent
- **Rate Limiting:** bottleneck library

**Free Hosting Options:**
- **Frontend:** Vercel, Netlify, or Cloudflare Pages
- **Backend:** Railway.app (free tier), Render.com, or fly.io
- **Database:** Neon (PostgreSQL), Supabase, or PlanetScale
- **Redis:** Upstash Redis (free tier)
- **Cron Jobs:** GitHub Actions or EasyCron

**DevOps:**
- **CI/CD:** GitHub Actions
- **Monitoring:** Sentry (error tracking), Uptime Robot
- **Analytics:** Plausible or Umami (privacy-friendly)

#### Option B: Desktop Application (Rust + Tauri)

**Frontend:**
- **Framework:** React, Svelte, or Vue.js
- **Bundler:** Vite
- **UI:** Tailwind CSS + DaisyUI or Material-UI
- **Same libraries as web version for consistency**

**Backend (Rust):**
- **Framework:** Axum or Actix-web for local API server
- **Database:** SQLite with rusqlite or diesel ORM
- **HTTP Client:** reqwest (async HTTP client)
- **HTML Parsing:** scraper (uses select.rs)
- **JSON:** serde_json
- **Async Runtime:** tokio

**Desktop Features:**
- **Cross-platform:** Windows, macOS, Linux (single codebase)
- **Small bundle size:** ~10-15MB
- **Native system tray integration**
- **Auto-updates:** Tauri updater
- **Local data storage:** All data kept on user's machine
- **No internet required** (after initial data download)

**Key Dependencies:**
```toml
[dependencies]
tauri = "2.0"
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
sqlx = { version = "0.7", features = ["sqlite", "runtime-tokio"] }
scraper = "0.18"
```

### 2.2 Database Schema (PostgreSQL/SQLite)

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  subscription_tier VARCHAR(50) DEFAULT 'free',
  api_quota_remaining INTEGER DEFAULT 100,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Keywords table
CREATE TABLE keywords (
  id SERIAL PRIMARY KEY,
  keyword VARCHAR(500) NOT NULL,
  search_volume INTEGER,
  competition_score DECIMAL(5,2),
  trend VARCHAR(20), -- 'rising', 'stable', 'declining'
  marketplace VARCHAR(5) DEFAULT 'US',
  last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(keyword, marketplace)
);

-- Categories table
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  browse_node_id BIGINT UNIQUE NOT NULL,
  name VARCHAR(500) NOT NULL,
  parent_id INTEGER REFERENCES categories(id),
  marketplace VARCHAR(5) DEFAULT 'US',
  book1_bsr INTEGER,
  book100_bsr INTEGER,
  books_last_30_days INTEGER,
  difficulty_score DECIMAL(5,2),
  is_ghost BOOLEAN DEFAULT FALSE,
  last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Books/Products table
CREATE TABLE books (
  id SERIAL PRIMARY KEY,
  asin VARCHAR(10) UNIQUE NOT NULL,
  title VARCHAR(1000),
  author VARCHAR(500),
  price DECIMAL(10,2),
  bsr INTEGER,
  rating DECIMAL(3,2),
  review_count INTEGER,
  publication_date DATE,
  page_count INTEGER,
  format VARCHAR(50), -- 'kindle', 'paperback', 'hardcover', 'audiobook'
  marketplace VARCHAR(5) DEFAULT 'US',
  estimated_sales INTEGER,
  estimated_revenue DECIMAL(10,2),
  last_scraped TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Book keywords junction table
CREATE TABLE book_keywords (
  book_id INTEGER REFERENCES books(id),
  keyword_id INTEGER REFERENCES keywords(id),
  position INTEGER, -- position in search results
  PRIMARY KEY (book_id, keyword_id)
);

-- Book categories junction table
CREATE TABLE book_categories (
  book_id INTEGER REFERENCES books(id),
  category_id INTEGER REFERENCES categories(id),
  rank INTEGER, -- rank within category
  PRIMARY KEY (book_id, category_id)
);

-- Reviews table (for analysis)
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  book_id INTEGER REFERENCES books(id),
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  review_text TEXT,
  verified_purchase BOOLEAN,
  helpful_count INTEGER DEFAULT 0,
  review_date DATE,
  sentiment_score DECIMAL(4,2), -- -1 to 1
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Saved searches/projects
CREATE TABLE user_projects (
  id SERIAL PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  keywords JSONB,
  target_categories JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- API cache table
CREATE TABLE api_cache (
  id SERIAL PRIMARY KEY,
  cache_key VARCHAR(500) UNIQUE NOT NULL,
  cache_value JSONB NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Scraping queue
CREATE TABLE scrape_queue (
  id SERIAL PRIMARY KEY,
  task_type VARCHAR(50) NOT NULL, -- 'keyword', 'asin', 'category', 'reviews'
  task_payload JSONB NOT NULL,
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
  priority INTEGER DEFAULT 0,
  retry_count INTEGER DEFAULT 0,
  scheduled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  error_message TEXT
);

-- Indexes for performance
CREATE INDEX idx_keywords_keyword ON keywords(keyword);
CREATE INDEX idx_keywords_marketplace ON keywords(marketplace);
CREATE INDEX idx_categories_browse_node ON categories(browse_node_id);
CREATE INDEX idx_books_asin ON books(asin);
CREATE INDEX idx_books_bsr ON books(bsr);
CREATE INDEX idx_reviews_book_id ON reviews(book_id);
CREATE INDEX idx_scrape_queue_status ON scrape_queue(status, scheduled_at);
```

---

## 3. Data Collection Strategy

### 3.1 Web Scraping Implementation

**Rate Limiting:**
- Implement exponential backoff
- Respect robots.txt
- Maximum 10 requests per minute per domain
- Randomized delays (2-5 seconds between requests)
- User-agent rotation
- Cookie management

**Scraping Architecture:**
```typescript
// Example scraper structure
class AmazonScraper {
  private queue: ScraperQueue;
  private cache: CacheManager;
  private rateLimiter: RateLimiter;
  
  async scrapeSearchResults(keyword: string, marketplace: string) {
    // Check cache first
    const cached = await this.cache.get(`search:${keyword}:${marketplace}`);
    if (cached) return cached;
    
    // Rate limit check
    await this.rateLimiter.waitForSlot();
    
    // Scrape with retry logic
    const html = await this.fetchWithRetry(url);
    const results = this.parseSearchResults(html);
    
    // Cache results (24 hour TTL)
    await this.cache.set(`search:${keyword}:${marketplace}`, results, 86400);
    
    return results;
  }
  
  private parseSearchResults(html: string): SearchResult[] {
    const $ = cheerio.load(html);
    const results: SearchResult[] = [];
    
    $('[data-component-type="s-search-result"]').each((i, elem) => {
      const asin = $(elem).attr('data-asin');
      const title = $(elem).find('h2 a span').text();
      const price = this.parsePrice($(elem).find('.a-price-whole').text());
      const rating = parseFloat($(elem).find('.a-icon-star-small span').text());
      const reviewCount = this.parseNumber($(elem).find('.a-size-base').first().text());
      
      results.push({ asin, title, price, rating, reviewCount });
    });
    
    return results;
  }
}
```

### 3.2 API Integration

**Keepa API (Primary Data Source):**
- Product history (price, BSR, ratings over time)
- Category information
- Offer data
- Statistics calculation

**Implementation:**
```typescript
class KeepaClient {
  private apiKey: string;
  private baseUrl = 'https://api.keepa.com';
  
  async getProductDetails(asins: string[]): Promise<KeepaProduct[]> {
    const response = await axios.get(`${this.baseUrl}/product`, {
      params: {
        key: this.apiKey,
        domain: 1, // 1 = US, 2 = UK, etc.
        asin: asins.join(','),
        stats: 90 // 90 days of stats
      }
    });
    
    return response.data.products.map(p => this.transformKeepaData(p));
  }
  
  private transformKeepaData(raw: any): KeepaProduct {
    // Transform Keepa's CSV format to usable JSON
    return {
      asin: raw.asin,
      currentPrice: this.decodeKeepaPrice(raw.csv[0]?.last),
      currentBSR: raw.csv[3]?.last,
      averageBSR90d: raw.stats.avg[3],
      // ... more transformations
    };
  }
}
```

### 3.3 Caching Strategy

**Cache Layers:**
1. **Memory Cache:** Frequently accessed data (5-minute TTL)
2. **Redis Cache:** API responses, search results (1-24 hour TTL)
3. **Database Cache:** Historical data, category listings (7-day TTL)

**Cache Invalidation:**
- Time-based expiration
- Manual refresh triggers
- Webhook-based updates (if available)

---

## 4. Algorithm Implementations

### 4.1 Keyword Difficulty Score

```typescript
function calculateKeywordDifficulty(keyword: string, competitors: Book[]): number {
  // SERP Intensity (30%): How competitive the top results are
  const serpIntensity = calculateSERPIntensity(competitors);
  
  // Keyword Usage (20%): How prominently keyword appears in titles
  const keywordUsage = calculateKeywordUsage(keyword, competitors);
  
  // BSR Toughness (25%): How well competitors are selling
  const bsrToughness = calculateBSRToughness(competitors);
  
  // Market Saturation (15%): Number of results
  const saturation = calculateMarketSaturation(competitors.length);
  
  // Search Volume Factor (10%): Estimated search demand
  const searchVolume = estimateSearchVolume(keyword);
  
  const difficulty = 
    (serpIntensity * 0.30) +
    (keywordUsage * 0.20) +
    (bsrToughness * 0.25) +
    (saturation * 0.15) +
    (searchVolume * 0.10);
  
  return Math.round(difficulty * 100); // 0-100 score
}

function calculateSERPIntensity(competitors: Book[]): number {
  const top10 = competitors.slice(0, 10);
  const avgReviews = top10.reduce((sum, b) => sum + b.reviewCount, 0) / 10;
  const avgRating = top10.reduce((sum, b) => sum + b.rating, 0) / 10;
  
  // Normalize to 0-1 scale
  const reviewScore = Math.min(avgReviews / 5000, 1);
  const ratingScore = avgRating / 5;
  
  return (reviewScore + ratingScore) / 2;
}
```

### 4.2 BSR to Sales Conversion

```typescript
interface SalesEstimate {
  dailySales: number;
  weeklySales: number;
  monthlySales: number;
  annualSales: number;
}

function estimateSalesFromBSR(
  bsr: number, 
  format: 'kindle' | 'paperback' | 'hardcover' | 'audiobook',
  marketplace: string = 'US'
): SalesEstimate {
  // Power-law coefficients by format
  const coefficients = {
    kindle: { a: 5500, b: -0.83 },
    paperback: { a: 2600, b: -0.75 },
    hardcover: { a: 1800, b: -0.70 },
    audiobook: { a: 1200, b: -0.65 }
  };
  
  // Marketplace adjustment factors
  const marketplaceMultipliers = {
    US: 1.0,
    UK: 0.25,
    DE: 0.20,
    FR: 0.15,
    ES: 0.10,
    IT: 0.08,
    CA: 0.12,
    AU: 0.10
  };
  
  const { a, b } = coefficients[format];
  const multiplier = marketplaceMultipliers[marketplace] || 1.0;
  
  // Daily sales = A × BSR^B
  const dailySales = Math.round(a * Math.pow(bsr, b) * multiplier);
  
  return {
    dailySales,
    weeklySales: dailySales * 7,
    monthlySales: dailySales * 30,
    annualSales: dailySales * 365
  };
}
```

### 4.3 Category Difficulty Algorithm

```typescript
function calculateCategoryDifficulty(category: Category, books: Book[]): number {
  const top100 = books.slice(0, 100);
  
  // Factor 1: Sales velocity required (40%)
  const book1Sales = estimateSalesFromBSR(category.book1_bsr, 'kindle').dailySales;
  const salesVelocity = Math.min(book1Sales / 100, 1); // Normalize
  
  // Factor 2: Competition density (30%)
  const competitionDensity = Math.min(category.books_last_30_days / 1000, 1);
  
  // Factor 3: Review barriers (20%)
  const avgReviews = top100.reduce((sum, b) => sum + b.reviewCount, 0) / 100;
  const reviewBarrier = Math.min(avgReviews / 500, 1);
  
  // Factor 4: Price pressure (10%)
  const avgPrice = top100.reduce((sum, b) => sum + b.price, 0) / 100;
  const pricePressure = avgPrice > 2.99 ? 0.5 : 0.8; // Lower price = higher pressure
  
  const difficulty = 
    (salesVelocity * 0.40) +
    (competitionDensity * 0.30) +
    (reviewBarrier * 0.20) +
    (pricePressure * 0.10);
  
  return Math.round(difficulty * 100);
}
```

### 4.4 Niche Opportunity Score

```typescript
interface NicheMetrics {
  searchVolume: number;
  competition: number;
  averageBSR: number;
  averagePrice: number;
  reviewVelocity: number;
  trendDirection: 'rising' | 'stable' | 'declining';
}

function calculateOpportunityScore(metrics: NicheMetrics): number {
  // Demand score (35%): Higher search volume = better
  const demandScore = Math.min(metrics.searchVolume / 10000, 1);
  
  // Competition score (30%): Lower competition = better
  const competitionScore = 1 - Math.min(metrics.competition / 1000, 1);
  
  // Profitability score (20%): BSR and price
  const profitScore = calculateProfitability(metrics.averageBSR, metrics.averagePrice);
  
  // Barrier score (10%): Review requirements
  const barrierScore = 1 - Math.min(metrics.reviewVelocity / 100, 1);
  
  // Trend score (5%): Growing markets are better
  const trendScore = metrics.trendDirection === 'rising' ? 1.0 :
                     metrics.trendDirection === 'stable' ? 0.5 : 0.2;
  
  const opportunity = 
    (demandScore * 0.35) +
    (competitionScore * 0.30) +
    (profitScore * 0.20) +
    (barrierScore * 0.10) +
    (trendScore * 0.05);
  
  return Math.round(opportunity * 100);
}
```

### 4.5 Review Sentiment Analysis

```typescript
class ReviewAnalyzer {
  private positiveWords = ['great', 'excellent', 'amazing', 'helpful', 'love', 'perfect'];
  private negativeWords = ['bad', 'terrible', 'waste', 'disappointed', 'awful', 'poor'];
  
  analyzeSentiment(reviewText: string): number {
    const words = reviewText.toLowerCase().split(/\s+/);
    let score = 0;
    
    words.forEach(word => {
      if (this.positiveWords.includes(word)) score += 1;
      if (this.negativeWords.includes(word)) score -= 1;
    });
    
    // Normalize to -1 to 1 scale
    return Math.max(-1, Math.min(1, score / words.length * 100));
  }
  
  extractTopPhrases(reviews: Review[], minCount: number = 5): PhraseFrequency[] {
    const phrases = new Map<string, number>();
    
    reviews.forEach(review => {
      const text = review.review_text.toLowerCase();
      // Extract 2-4 word phrases
      const matches = text.match(/\b(\w+\s+\w+(\s+\w+)?(\s+\w+)?)\b/g) || [];
      
      matches.forEach(phrase => {
        phrases.set(phrase, (phrases.get(phrase) || 0) + 1);
      });
    });
    
    return Array.from(phrases.entries())
      .filter(([_, count]) => count >= minCount)
      .map(([phrase, count]) => ({ phrase, count }))
      .sort((a, b) => b.count - a.count);
  }
}
```

---

## 5. API Endpoints Design

### 5.1 RESTful API Structure

```
Base URL: https://api.kdpresearch.com/v1

Authentication: Bearer JWT tokens
Rate Limiting: 100 requests/hour (free), 1000/hour (premium)
```

**Endpoint List:**

```
## Authentication
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout

## Keywords
GET    /keywords/search?q={query}&marketplace={US}
GET    /keywords/suggestions?seed={keyword}
GET    /keywords/volume?keywords[]={kw1}&keywords[]={kw2}
POST   /keywords/analyze
GET    /keywords/{id}
POST   /keywords/reverse-asin?asin={asin}

## Categories
GET    /categories
GET    /categories/{id}
GET    /categories/search?q={query}
GET    /categories/recommend?keywords[]={kw1}&price={9.99}
GET    /categories/tree?parent={id}
GET    /categories/bestsellers/{id}?limit=100

## Books/Products
GET    /books/{asin}
GET    /books/search?keyword={query}&page=1&limit=50
POST   /books/batch
GET    /books/{asin}/keywords
GET    /books/{asin}/categories
GET    /books/{asin}/reviews?limit=100&sentiment={positive}
GET    /books/{asin}/history?days=90

## Analysis
POST   /analyze/niche
POST   /analyze/competition
POST   /analyze/listing
POST   /analyze/opportunity

## Tools
POST   /tools/bsr-calculator
POST   /tools/keyword-generator
POST   /tools/ams-campaign
POST   /tools/listing-optimizer

## User Data
GET    /user/projects
POST   /user/projects
GET    /user/projects/{id}
PUT    /user/projects/{id}
DELETE /user/projects/{id}
GET    /user/saved-keywords
GET    /user/tracked-books

## Export
POST   /export/keywords?format={csv|xlsx}
POST   /export/categories?format={csv|xlsx}
POST   /export/analysis?format={pdf|xlsx}
```

### 5.2 Example API Request/Response

**Keyword Search:**
```json
GET /keywords/search?q=cookbook&marketplace=US

Response:
{
  "keyword": "cookbook",
  "marketplace": "US",
  "searchVolume": 165000,
  "competitionScore": 78,
  "difficulty": "high",
  "trend": "stable",
  "suggestions": [
    {
      "keyword": "cookbook for beginners",
      "searchVolume": 24500,
      "competitionScore": 52,
      "difficulty": "medium"
    },
    {
      "keyword": "air fryer cookbook",
      "searchVolume": 89000,
      "competitionScore": 85,
      "difficulty": "high"
    }
  ],
  "relatedKeywords": [
    "recipes", "cooking book", "recipe book"
  ],
  "topCompetitors": [
    {
      "asin": "B08XYZ123",
      "title": "The Ultimate Cookbook for Every Occasion",
      "bsr": 1247,
      "rating": 4.7,
      "reviewCount": 8934
    }
  ]
}
```

**Category Recommendation:**
```json
POST /categories/recommend

Request:
{
  "keywords": ["cookbook", "healthy eating", "meal prep"],
  "price": 4.99,
  "targetSales": 10,
  "format": "kindle"
}

Response:
{
  "recommendations": [
    {
      "categoryId": 12345,
      "name": "Kindle eBooks > Cookbooks, Food & Wine > Special Diet > Healthy",
      "difficultyScore": 45,
      "dailySalesRequired": 8,
      "estimatedRevenue": {
        "daily": 55.92,
        "monthly": 1677.60
      },
      "currentTop1BSR": 8234,
      "booksLast30Days": 234,
      "trend": "rising",
      "successProbability": 0.72,
      "bestMonthsToLaunch": ["January", "September"]
    }
  ]
}
```

---

## 6. Frontend Implementation

### 6.1 Key UI Components

**Dashboard:**
- Overview metrics cards (keywords tracked, categories analyzed, projects)
- Recent activity feed
- Quick action buttons
- Charts showing trends

**Keyword Research Page:**
- Search bar with autocomplete
- Filter panel (volume, competition, difficulty)
- Results table with sorting
- Bulk actions (save, export, add to project)
- Keyword expansion tool
- Visualization: keyword difficulty matrix

**Category Explorer:**
- Category tree navigation
- Search and filter
- Category comparison table
- Bestseller requirements calculator
- Difficulty score heatmap
- Seasonal trend charts

**Competitor Analysis:**
- ASIN input or search
- Book details card
- Keyword extraction results
- Review sentiment analysis
- Pricing history chart
- Sales estimation

**Listing Optimizer:**
- Multi-step wizard
- Title/subtitle/description fields with character counters
- Keyword usage tracker (color-coded)
- Real-time optimization score
- Suggestions panel
- Preview mode

**AMS Campaign Builder:**
- Campaign name and budget
- Keyword selector from research
- Bid amount calculator
- Negative keyword manager
- Export to Amazon format

### 6.2 State Management Structure (React)

```typescript
// Using Zustand for simple state management

interface AppState {
  // User & Auth
  user: User | null;
  isAuthenticated: boolean;
  apiQuotaRemaining: number;
  
  // Keywords
  keywords: Keyword[];
  selectedKeywords: string[];
  keywordFilters: KeywordFilters;
  
  // Categories
  categories: Category[];
  selectedCategory: Category | null;
  categoryFilters: CategoryFilters;
  
  // Books
  trackedBooks: Book[];
  currentBook: Book | null;
  
  // Projects
  projects: Project[];
  activeProject: Project | null;
  
  // UI State
  isLoading: boolean;
  error: string | null;
  sidebarOpen: boolean;
  
  // Actions
  setUser: (user: User) => void;
  addKeywords: (keywords: Keyword[]) => void;
  selectKeyword: (id: string) => void;
  // ... more actions
}

const useAppStore = create<AppState>((set) => ({
  user: null,
  isAuthenticated: false,
  keywords: [],
  // ... initial state
  
  setUser: (user) => set({ user, isAuthenticated: true }),
  addKeywords: (keywords) => set((state) => ({
    keywords: [...state.keywords, ...keywords]
  })),
  // ... action implementations
}));
```

### 6.3 Key Pages/Routes

```
/                        -> Dashboard
/login                   -> Login page
/register                -> Registration
/keywords                -> Keyword research tool
/keywords/suggestions    -> Keyword expander
/keywords/reverse-asin   -> Reverse ASIN lookup
/categories              -> Category browser
/categories/:id          -> Category detail
/books/:asin             -> Book detail page
/books/search            -> Book search
/competitor              -> Competitor analysis
/niche-finder            -> Niche discovery wizard
/listing-optimizer       -> Listing builder
/ams-tools               -> Amazon Ads tools
/projects                -> User projects
/projects/:id            -> Project detail
/export                  -> Export tools
/settings                -> User settings
/documentation           -> Help & docs
```

---

## 7. Rust + Tauri Desktop Application

### 7.1 Project Structure

```
kdp-research-desktop/
├── src-tauri/
│   ├── src/
│   │   ├── main.rs              # Tauri entry point
│   │   ├── api/                 # API clients
│   │   │   ├── amazon.rs        # Amazon scraper
│   │   │   ├── keepa.rs         # Keepa API client
│   │   │   └── cache.rs         # Cache manager
│   │   ├── database/            # SQLite operations
│   │   │   ├── mod.rs
│   │   │   ├── models.rs        # Data models
│   │   │   ├── queries.rs       # SQL queries
│   │   │   └── migrations/      # DB migrations
│   │   ├── analyzers/           # Analysis algorithms
│   │   │   ├── keywords.rs
│   │   │   ├── categories.rs
│   │   │   ├── bsr.rs
│   │   │   └── sentiment.rs
│   │   ├── scrapers/            # Web scraping
│   │   │   ├── mod.rs
│   │   │   ├── amazon_search.rs
│   │   │   └── rate_limiter.rs
│   │   ├── commands.rs          # Tauri commands (frontend API)
│   │   └── utils.rs             # Helper functions
│   ├── Cargo.toml
│   └── tauri.conf.json          # Tauri configuration
├── src/                         # Frontend (React/Svelte)
│   ├── components/
│   ├── pages/
│   ├── stores/
│   └── App.tsx
├── public/
├── package.json
└── vite.config.ts
```

### 7.2 Key Rust Implementations

**Tauri Commands (Frontend-Backend Bridge):**
```rust
use tauri::State;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct KeywordSearchResult {
    keyword: String,
    search_volume: i32,
    competition_score: f32,
    suggestions: Vec<String>,
}

#[tauri::command]
async fn search_keywords(
    keyword: String,
    marketplace: String,
    state: State<'_, AppState>,
) -> Result<KeywordSearchResult, String> {
    let scraper = &state.amazon_scraper;
    let db = &state.database;
    
    // Check cache first
    if let Some(cached) = db.get_cached_keyword(&keyword, &marketplace).await? {
        return Ok(cached);
    }
    
    // Scrape if not cached
    let results = scraper.search_suggestions(&keyword, &marketplace).await
        .map_err(|e| e.to_string())?;
    
    // Calculate metrics
    let competition = calculate_competition(&keyword, &marketplace, scraper).await?;
    let volume = estimate_search_volume(&keyword);
    
    let result = KeywordSearchResult {
        keyword: keyword.clone(),
        search_volume: volume,
        competition_score: competition,
        suggestions: results,
    };
    
    // Cache for future use
    db.cache_keyword(&result, &marketplace).await?;
    
    Ok(result)
}

#[tauri::command]
async fn get_book_data(
    asin: String,
    state: State<'_, AppState>,
) -> Result<BookData, String> {
    // Implementation similar to above
}

#[tauri::command]
async fn analyze_category(
    category_id: i64,
    state: State<'_, AppState>,
) -> Result<CategoryAnalysis, String> {
    // Implementation
}
```

**Database Manager:**
```rust
use sqlx::{SqlitePool, Row};
use crate::models::*;

pub struct Database {
    pool: SqlitePool,
}

impl Database {
    pub async fn new(db_path: &str) -> Result<Self, sqlx::Error> {
        let pool = SqlitePool::connect(db_path).await?;
        Self::run_migrations(&pool).await?;
        Ok(Self { pool })
    }
    
    pub async fn get_cached_keyword(
        &self,
        keyword: &str,
        marketplace: &str,
    ) -> Result<Option<KeywordSearchResult>, sqlx::Error> {
        let row = sqlx::query(
            "SELECT data FROM api_cache 
             WHERE cache_key = ? AND expires_at > datetime('now')"
        )
        .bind(format!("keyword:{}:{}", keyword, marketplace))
        .fetch_optional(&self.pool)
        .await?;
        
        match row {
            Some(r) => {
                let json: String = r.get("data");
                Ok(Some(serde_json::from_str(&json).unwrap()))
            }
            None => Ok(None),
        }
    }
    
    pub async fn save_book(&self, book: &Book) -> Result<i64, sqlx::Error> {
        let id = sqlx::query(
            "INSERT INTO books (asin, title, author, price, bsr, rating, review_count)
             VALUES (?, ?, ?, ?, ?, ?, ?)
             ON CONFLICT(asin) DO UPDATE SET
               title=excluded.title, price=excluded.price, bsr=excluded.bsr,
               rating=excluded.rating, review_count=excluded.review_count,
               last_scraped=datetime('now')
             RETURNING id"
        )
        .bind(&book.asin)
        .bind(&book.title)
        .bind(&book.author)
        .bind(book.price)
        .bind(book.bsr)
        .bind(book.rating)
        .bind(book.review_count)
        .fetch_one(&self.pool)
        .await?
        .get(0);
        
        Ok(id)
    }
}
```

**Amazon Scraper:**
```rust
use reqwest::Client;
use scraper::{Html, Selector};
use std::time::Duration;

pub struct AmazonScraper {
    client: Client,
    rate_limiter: RateLimiter,
    user_agents: Vec<String>,
}

impl AmazonScraper {
    pub fn new() -> Self {
        let client = Client::builder()
            .timeout(Duration::from_secs(30))
            .build()
            .unwrap();
        
        Self {
            client,
            rate_limiter: RateLimiter::new(10, Duration::from_secs(60)),
            user_agents: vec![
                "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...".to_string(),
                // More user agents
            ],
        }
    }
    
    pub async fn search_suggestions(
        &self,
        keyword: &str,
        marketplace: &str,
    ) -> Result<Vec<String>, Box<dyn std::error::Error>> {
        self.rate_limiter.wait().await;
        
        let url = format!(
            "https://completion.amazon.{}/api/2017/suggestions?mid=ATVPDKIKX0DER&alias=aps&prefix={}",
            self.get_domain(marketplace), 
            urlencoding::encode(keyword)
        );
        
        let response = self.client
            .get(&url)
            .header("User-Agent", self.get_random_user_agent())
            .send()
            .await?;
        
        let json: serde_json::Value = response.json().await?;
        
        let suggestions = json["suggestions"]
            .as_array()
            .unwrap_or(&vec![])
            .iter()
            .filter_map(|s| s["value"].as_str())
            .map(String::from)
            .collect();
        
        Ok(suggestions)
    }
    
    pub async fn scrape_search_results(
        &self,
        keyword: &str,
        marketplace: &str,
    ) -> Result<Vec<SearchResult>, Box<dyn std::error::Error>> {
        self.rate_limiter.wait().await;
        
        let url = format!(
            "https://www.amazon.{}/s?k={}",
            self.get_domain(marketplace),
            urlencoding::encode(keyword)
        );
        
        let html = self.client
            .get(&url)
            .header("User-Agent", self.get_random_user_agent())
            .send()
            .await?
            .text()
            .await?;
        
        self.parse_search_results(&html)
    }
    
    fn parse_search_results(&self, html: &str) -> Result<Vec<SearchResult>, Box<dyn std::error::Error>> {
        let document = Html::parse_document(html);
        let selector = Selector::parse("[data-component-type='s-search-result']").unwrap();
        
        let mut results = Vec::new();
        
        for element in document.select(&selector) {
            let asin = element.value().attr("data-asin").unwrap_or("");
            
            // Extract more data...
            let title_selector = Selector::parse("h2 a span").unwrap();
            let title = element.select(&title_selector)
                .next()
                .map(|e| e.text().collect::<String>())
                .unwrap_or_default();
            
            results.push(SearchResult {
                asin: asin.to_string(),
                title,
                // ... more fields
            });
        }
        
        Ok(results)
    }
}
```

### 7.3 Frontend Integration with Tauri

```typescript
// src/api/tauri-api.ts
import { invoke } from '@tauri-apps/api/tauri';

export interface KeywordSearchParams {
  keyword: string;
  marketplace: string;
}

export async function searchKeywords(params: KeywordSearchParams) {
  return await invoke<KeywordSearchResult>('search_keywords', params);
}

export async function getBookData(asin: string) {
  return await invoke<BookData>('get_book_data', { asin });
}

export async function analyzeCategory(categoryId: number) {
  return await invoke<CategoryAnalysis>('analyze_category', { 
    category_id: categoryId 
  });
}

// Usage in React component
function KeywordResearch() {
  const [results, setResults] = useState<KeywordSearchResult | null>(null);
  const [loading, setLoading] = useState(false);
  
  async function handleSearch(keyword: string) {
    setLoading(true);
    try {
      const data = await searchKeywords({ 
        keyword, 
        marketplace: 'US' 
      });
      setResults(data);
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div>
      <input 
        type="text" 
        onKeyDown={(e) => e.key === 'Enter' && handleSearch(e.target.value)} 
      />
      {loading && <Spinner />}
      {results && <KeywordResults data={results} />}
    </div>
  );
}
```

---

## 8. Deployment Strategy

### 8.1 Web Application Deployment (Free Hosting)

**Architecture:**
```
Frontend (Vercel) -> CDN -> Backend (Render/Railway) -> Database (Neon)
                                       ↓
                                   Redis (Upstash)
```

**Step-by-Step:**

1. **Frontend Deployment (Vercel):**
   - Connect GitHub repository
   - Auto-deploy on push to main
   - Environment variables in Vercel dashboard
   - Custom domain setup (optional)

2. **Backend Deployment (Railway.app):**
   ```bash
   # railway.json
   {
     "$schema": "https://railway.app/railway.schema.json",
     "build": {
       "builder": "NIXPACKS"
     },
     "deploy": {
       "startCommand": "node dist/server.js",
       "restartPolicyType": "ON_FAILURE",
       "restartPolicyMaxRetries": 10
     }
   }
   ```

3. **Database Setup (Neon PostgreSQL):**
   - Create project at neon.tech
   - Copy connection string
   - Run migrations:
   ```bash
   npm run migrate:up
   ```

4. **Redis Setup (Upstash):**
   - Create database at upstash.com
   - Configure in environment variables

5. **Environment Variables:**
   ```env
   # Backend .env
   DATABASE_URL=postgresql://...
   REDIS_URL=redis://...
   JWT_SECRET=...
   KEEPA_API_KEY=...
   NODE_ENV=production
   CORS_ORIGIN=https://yourapp.vercel.app
   ```

6. **Cron Jobs (GitHub Actions):**
   ```yaml
   # .github/workflows/daily-scrape.yml
   name: Daily Category Update
   on:
     schedule:
       - cron: '0 2 * * *'  # 2 AM daily
   jobs:
     scrape:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - run: npm install
         - run: npm run scrape:categories
           env:
             DATABASE_URL: ${{ secrets.DATABASE_URL }}
   ```

**Estimated Costs:**
- Vercel: Free (100GB bandwidth/month)
- Railway: $5/month (with free trial)
- Neon: Free tier (3GB storage)
- Upstash: Free tier (10k requests/day)
- **Total: $0-5/month**

### 8.2 Desktop Application Distribution

**Build for All Platforms:**
```bash
# Build for current platform
npm run tauri build

# Cross-compilation (requires setup)
cargo tauri build --target x86_64-pc-windows-msvc   # Windows
cargo tauri build --target x86_64-apple-darwin      # macOS Intel
cargo tauri build --target aarch64-apple-darwin     # macOS ARM
cargo tauri build --target x86_64-unknown-linux-gnu # Linux
```

**Distribution Channels:**
1. **GitHub Releases** (Free)
   - Automatic builds via GitHub Actions
   - Direct download links
   - Update checking built-in

2. **Website Download Page**
   - Hosted on GitHub Pages (free)
   - Auto-detect OS and offer correct installer

3. **Optional App Stores:**
   - Microsoft Store (Windows)
   - Homebrew (macOS)
   - Snap Store (Linux)

**Auto-Updates Configuration:**
```json
// tauri.conf.json
{
  "updater": {
    "active": true,
    "endpoints": [
      "https://github.com/youruser/kdp-tool/releases/latest/download"
    ],
    "dialog": true,
    "pubkey": "YOUR_PUBLIC_KEY"
  }
}
```

---

## 9. Feature Roadmap

### Phase 1: MVP (Months 1-3)
- [x] Basic keyword research (search, suggestions, competition)
- [x] Category browser with filtering
- [x] BSR to sales calculator
- [x] Simple book lookup (ASIN)
- [x] Basic web scraping (rate-limited)
- [x] SQLite database
- [x] Simple React UI
- [ ] User authentication
- [ ] Export to CSV

### Phase 2: Enhanced Features (Months 4-6)
- [ ] Advanced keyword analysis
- [ ] Category recommendations
- [ ] Competitor analysis
- [ ] Review sentiment analysis
- [ ] Listing optimizer
- [ ] API integrations (Keepa)
- [ ] Redis caching
- [ ] User projects/workspaces

### Phase 3: Premium Features (Months 7-9)
- [ ] AMS keyword generator
- [ ] Historical trend analysis
- [ ] Niche opportunity finder
- [ ] Automated reporting
- [ ] Bulk operations
- [ ] Chrome extension
- [ ] Mobile app (React Native)

### Phase 4: Advanced (Months 10-12)
- [ ] AI-powered recommendations (GPT-4 integration)
- [ ] Predictive analytics
- [ ] Market forecasting
- [ ] Collaborative workspaces
- [ ] API for developers
- [ ] White-label options

---

## 10. Testing Strategy

### 10.1 Unit Tests

**Backend (Jest/Vitest):**
```typescript
// tests/algorithms/bsr-calculator.test.ts
import { estimateSalesFromBSR } from '../src/algorithms/bsr';

describe('BSR to Sales Conversion', () => {
  it('should calculate Kindle sales correctly', () => {
    const result = estimateSalesFromBSR(1000, 'kindle', 'US');
    expect(result.dailySales).toBeGreaterThan(0);
    expect(result.dailySales).toBeLessThan(100);
  });
  
  it('should handle marketplace multipliers', () => {
    const us = estimateSalesFromBSR(1000, 'kindle', 'US');
    const uk = estimateSalesFromBSR(1000, 'kindle', 'UK');
    expect(us.dailySales).toBeGreaterThan(uk.dailySales);
  });
});
```

**Rust Tests:**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[tokio::test]
    async fn test_keyword_search() {
        let scraper = AmazonScraper::new();
        let results = scraper.search_suggestions("cookbook", "US").await;
        assert!(results.is_ok());
        assert!(!results.unwrap().is_empty());
    }
    
    #[test]
    fn test_difficulty_calculation() {
        let score = calculate_keyword_difficulty("test", &vec![]);
        assert!(score >= 0.0 && score <= 100.0);
    }
}
```

### 10.2 Integration Tests

```typescript
// tests/integration/api.test.ts
import request from 'supertest';
import app from '../src/app';

describe('API Integration Tests', () => {
  it('should search keywords', async () => {
    const response = await request(app)
      .get('/api/v1/keywords/search?q=cookbook')
      .expect(200);
    
    expect(response.body).toHaveProperty('keyword');
    expect(response.body).toHaveProperty('suggestions');
  });
});
```

### 10.3 E2E Tests (Playwright)

```typescript
// e2e/keyword-research.spec.ts
import { test, expect } from '@playwright/test';

test('keyword research workflow', async ({ page }) => {
  await page.goto('/keywords');
  await page.fill('[data-testid="keyword-input"]', 'cookbook');
  await page.click('[data-testid="search-button"]');
  
  await expect(page.locator('[data-testid="results-table"]')).toBeVisible();
  await expect(page.locator('[data-testid="result-row"]').first()).toContainText('cookbook');
});
```

---

## 11. Documentation Plan

### 11.1 Developer Documentation
- API reference (OpenAPI/Swagger)
- Database schema documentation
- Algorithm explanations
- Deployment guides
- Contributing guidelines

### 11.2 User Documentation
- Getting started guide
- Feature tutorials (with screenshots)
- Best practices for KDP research
- FAQ
- Video tutorials

### 11.3 In-App Help
- Tooltips on complex features
- Contextual help panels
- Interactive tours (for first-time users)
- Example searches

---

## 12. Legal & Compliance

### 12.1 Terms of Service
- Acceptable use policy
- Rate limiting explanation
- Data usage terms
- Account termination conditions

### 12.2 Privacy Policy
- Data collection (minimal)
- No selling of user data
- GDPR compliance (EU users)
- Data retention policy

### 12.3 Amazon ToS Compliance
- Rate-limited scraping
- No aggressive bot behavior
- Respect robots.txt
- Clear attribution
- No circumventing access controls

### 12.4 Disclaimer
- Estimates are not guaranteed
- Amazon data may change
- Tool is for research purposes only
- No affiliation with Amazon

---

## 13. Monetization Strategy (Optional)

### 13.1 Freemium Model

**Free Tier:**
- 100 keyword searches/day
- 50 category lookups/day
- Basic BSR calculator
- Export limit: 100 rows
- No historical data

**Premium Tier ($19/month or $15/month annual):**
- Unlimited searches
- Full historical data (12 months)
- Advanced analytics
- Bulk operations
- Priority support
- API access
- No rate limits

**Pro Tier ($49/month):**
- Everything in Premium
- White-label option
- Team collaboration (5 users)
- Custom reports
- Early access to features
- Dedicated support

### 13.2 Alternative Revenue
- Affiliate links to book formatting services
- Sponsored listings (relevant tools)
- One-time purchase for desktop app ($99)

---

## 14. Implementation Timeline

### Month 1-2: Foundation
- Set up development environment
- Design database schema
- Implement basic scrapers
- Create core UI components
- Deploy MVP to free hosting

### Month 3-4: Core Features
- Keyword research module
- Category browser
- BSR calculator
- Basic analytics
- User authentication
- Export functionality

### Month 5-6: Enhanced Features
- Competitor analysis
- Review sentiment
- Listing optimizer
- API integrations
- Advanced filtering
- Caching system

### Month 7-8: Polish & Optimization
- Performance tuning
- UI/UX improvements
- Comprehensive testing
- Documentation
- Beta user feedback

### Month 9-10: Desktop App
- Rust + Tauri setup
- Port features to desktop
- SQLite integration
- Offline mode
- Cross-platform builds

### Month 11-12: Launch & Scale
- Marketing website
- User onboarding
- Analytics setup
- Support system
- Continuous improvements

---

## 15. Success Metrics

### Technical Metrics
- Response time < 2 seconds
- Uptime > 99.5%
- API success rate > 95%
- Zero critical bugs

### User Metrics
- 1,000 active users (first 6 months)
- 100 premium subscribers (first year)
- 4.5+ star rating
- < 5% churn rate

### Business Metrics
- Server costs < $50/month
- Revenue > $1,000/month (after 12 months)
- Positive user testimonials
- Growing community

---

## 16. Risk Mitigation

### Technical Risks
- **Amazon blocks scraping:** Use proxies, rotate user agents, add Keepa API fallback
- **Database costs:** Start with free tier, optimize queries, add caching
- **Performance issues:** Implement pagination, lazy loading, CDN

### Legal Risks
- **ToS violations:** Respect rate limits, consult lawyer for ToS
- **Copyright issues:** Cite sources, don't copy content

### Business Risks
- **Competition:** Focus on unique features, excellent UX
- **User acquisition:** Content marketing, SEO, free tier
- **Retention:** Continuous improvements, listen to feedback

---

## 17. Next Steps

1. **Choose deployment option** (Web vs Desktop vs Both)
2. **Set up development environment**
3. **Create GitHub repository** with proper .gitignore
4. **Design database schema** (start with SQLite)
5. **Build MVP scraper** with rate limiting
6. **Create basic UI** with keyword search
7. **Deploy to free hosting** (Railway + Vercel)
8. **Iterate based on feedback**

---

## Appendix

### A. Technology Alternatives

**Backend:**
- Python (FastAPI) - Easier for ML integrations
- Go - Better performance, smaller memory footprint
- Deno - Modern, TypeScript-native

**Frontend:**
- Svelte - Smaller bundle, faster
- Vue.js - Easier learning curve
- Next.js - Built-in SSR, better SEO

**Database:**
- MongoDB - Flexible schema
- CockroachDB - Distributed SQL
- DuckDB - Analytical queries

### B. Useful Resources

**APIs:**
- [Keepa API](https://keepa.com/#!api)
- [Amazon Product Advertising API](https://webservices.amazon.com/paapi5/documentation/)
- [DataForSEO](https://dataforseo.com/)

**Libraries:**
- [Playwright](https://playwright.dev/) - Browser automation
- [Cheerio](https://cheerio.js.org/) - HTML parsing
- [Tauri](https://tauri.app/) - Desktop apps

**Learning:**
- [Web Scraping Best Practices](https://scrapfly.io/blog/web-scraping-best-practices/)
- [BSR to Sales Conversion](https://kdp.amazon.com/en_US/help/topic/G201834340)

### C. Sample Configuration Files

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: kdp_research
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  api:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/kdp_research
      REDIS_URL: redis://redis:6379

volumes:
  pgdata:
```

**package.json scripts:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:e2e": "playwright test",
    "lint": "eslint src --ext ts,tsx",
    "format": "prettier --write src/**/*.{ts,tsx}",
    "migrate:up": "node scripts/migrate.js up",
    "migrate:down": "node scripts/migrate.js down",
    "scrape:categories": "node scripts/scrape-categories.js",
    "seed": "node scripts/seed-database.js"
  }
}
```

---

## Conclusion

This comprehensive plan provides a complete roadmap for building an all-in-one KDP research tool that combines the best features from Publisher Rocket, KDP Spy, Titans Pro, BookBeam, and other market leaders. 

**Key Advantages:**
✅ Free hosting options available
✅ Offline desktop version for privacy
✅ Modular architecture for easy additions
✅ Scalable from MVP to enterprise
✅ Clear monetization path
✅ Comprehensive feature set

**Recommended Starting Point:**
Begin with the web-based MVP using the free hosting stack (Vercel + Railway + Neon). This allows for rapid iteration and user feedback before investing time in the desktop app.

Once the core features are validated and you have active users, consider adding the Rust + Tauri desktop version for users who prefer offline access and enhanced privacy.

Good luck with your development! 🚀
