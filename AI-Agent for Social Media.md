# A comprehensive implementation plan for the configurable AI Content Creator system:

## AfriXport Content Creator - Configurable AI Model Implementation

### Phase 1: Database Schema & Infrastructure

#### 1.1 Create Database Tables

**`blog_posts` table:**
- Stores all blog content with full metadata
- Fields: id, title, slug, excerpt, content, author, category, read_time, featured_image_url, status (draft/published/archived), published_at, created_at, updated_at, created_by, seo_keywords[], meta_description, ai_model_used, generation_metadata (jsonb)

**`social_media_posts` table:**
- Stores generated social media content
- Fields: id, platform (linkedin/facebook/instagram/twitter), content, hashtags[], suggested_post_time, campaign_id, status (draft/scheduled/published), image_suggestions (jsonb), created_at, created_by, published_at, ai_model_used

**`content_campaigns` table:**
- Manages content campaigns
- Fields: id, name, description, theme, start_date, end_date, status (active/completed/draft), target_platforms[], created_by, created_at, updated_at

**`ai_model_settings` table:**
- Stores admin preferences for AI model selection
- Fields: id, setting_name (e.g., 'blog_generation_model', 'social_media_model'), model_value ('google/gemini-2.5-flash', 'openai/gpt-5-mini', etc.), is_active, created_at, updated_at, updated_by

**`content_generation_logs` table:**
- Tracks AI generation usage and costs
- Fields: id, content_type (blog/social), model_used, tokens_used, generation_time_ms, prompt_tokens, completion_tokens, estimated_cost, created_at, created_by

**RLS Policies:**
- Admins can manage all content tables
- Public can read published blog posts
- Audit logging for all AI generations

#### 1.2 Edge Functions Setup

**A. `generate-blog-content` Function**
```typescript
Input: {
  topic: string,
  wordCount: number,
  category: string,
  targetKeywords: string[],
  tone?: 'professional' | 'casual' | 'inspiring',
  aiModel?: string // Optional override
}

Output: {
  title: string,
  content: string,
  excerpt: string,
  metaDescription: string,
  suggestedHashtags: string[],
  seoKeywords: string[],
  readTime: string,
  model: string,
  tokensUsed: number
}
```

**Features:**
- Check `ai_model_settings` table for default model
- Allow per-request model override
- Streaming support for real-time generation
- Comprehensive error handling with fallback
- Usage tracking to `content_generation_logs`
- AfriXport brand voice system prompt
- SEO optimization built-in

**B. `generate-social-content` Function**
```typescript
Input: {
  topic: string,
  platforms: ('linkedin' | 'facebook' | 'instagram' | 'twitter')[],
  campaignId?: string,
  contentType: 'announcement' | 'story' | 'product' | 'educational',
  aiModel?: string
}

Output: {
  posts: Array<{
    platform: string,
    content: string,
    hashtags: string[],
    suggestedPostTime: string,
    visualSuggestion: string
  }>,
  model: string,
  tokensUsed: number
}
```

**Features:**
- Platform-specific optimization (length, tone, formatting)
- Batch generation (3-5 variations per platform)
- Hashtag strategy (5-10 relevant tags)
- Optimal posting time suggestions
- Visual content recommendations
- Campaign association

**C. `generate-content-calendar` Function**
```typescript
Input: {
  startDate: string,
  endDate: string,
  campaignTheme?: string,
  postsPerWeek: number,
  platforms: string[],
  aiModel?: string
}

Output: {
  calendar: Array<{
    date: string,
    platform: string,
    contentIdea: string,
    suggestedTime: string,
    priority: 'high' | 'medium' | 'low'
  }>,
  summary: string,
  model: string
}
```

**D. `optimize-content-seo` Function**
```typescript
Input: {
  content: string,
  targetKeywords: string[],
  aiModel?: string
}

Output: {
  optimizedTitle: string,
  metaDescription: string,
  suggestedKeywords: string[],
  readabilityScore: number,
  recommendations: string[],
  model: string
}
```

**E. `test-ai-models` Function** (NEW)
```typescript
// Allows admins to test different models side-by-side
Input: {
  prompt: string,
  models: string[] // e.g., ['google/gemini-2.5-flash', 'openai/gpt-5-mini']
}

Output: {
  results: Array<{
    model: string,
    response: string,
    tokensUsed: number,
    responseTime: number,
    estimatedCost: number
  }>
}
```

#### 1.3 AI Gateway Configuration

**Lovable AI Gateway Integration:**
- Base URL: `https://ai.gateway.lovable.dev/v1/chat/completions`
- Use existing `LOVABLE_API_KEY` from Supabase secrets
- No additional API key needed!

**Supported Models (All via Lovable AI):**
- `google/gemini-2.5-pro` - Most powerful Gemini (complex content)
- `google/gemini-2.5-flash` - **Default** (balanced, currently FREE)
- `google/gemini-2.5-flash-lite` - Fastest Gemini (simple tasks)
- `openai/gpt-5` - Most powerful OpenAI (premium content)
- `openai/gpt-5-mini` - Balanced OpenAI (good quality, moderate cost)
- `openai/gpt-5-nano` - Fastest OpenAI (quick tasks)

**Cost Management:**
- Track usage in `content_generation_logs`
- Display cost estimates in UI
- Show "FREE" badge for Gemini during promo period
- Alert when approaching rate limits (429 errors)
- Surface 402 (payment required) errors with clear messaging

### Phase 2: Admin Dashboard UI

#### 2.1 Model Configuration Panel

**`AIModelSettings` Component** (`src/components/admin/AIModelSettings.tsx`)
- Card-based interface showing all available models
- Real-time pricing display
- "Currently FREE" badges for Gemini models
- Set default models for:
  - Blog content generation
  - Social media posts
  - Content calendar
  - SEO optimization
- Test models side-by-side with sample prompts
- Usage analytics (tokens used, costs by model)
- Rate limit monitoring

#### 2.2 Content Management Tab

**Update `CommunicationsManagement.tsx`:**
Add new "Content Creator" tab with 4 sub-tabs:
1. **Blog Posts** - Create, edit, publish
2. **Social Media** - Generate multi-platform content
3. **Campaigns** - Manage content campaigns
4. **Settings** - AI model preferences & analytics

#### 2.3 Blog Post Editor

**`BlogPostEditor` Component** (`src/components/admin/BlogPostEditor.tsx`)

**Features:**
- Toggle: "AI Generate" vs "Manual Write"
- AI Generation Panel:
  - Topic input
  - Word count slider (300-2000)
  - Category selector
  - Keywords input (multi-select)
  - Tone selector
  - **Model selector dropdown** (with cost preview)
  - "Generate" button with loading state
  - Stream content token-by-token
- Rich text editor (TipTap or similar)
- Live preview (split view)
- SEO optimization panel:
  - Meta description
  - Keywords
  - Readability score
  - "Optimize with AI" button
- Image upload/management
- Status management (Draft/Published)
- Schedule publishing
- View generation history (model used, cost)

**Workflow:**
1. Admin clicks "Create New Blog Post"
2. Chooses AI generation or manual
3. If AI: Fills form → Selects model → Click "Generate"
4. Content streams in real-time
5. Admin refines content in editor
6. Click "Optimize SEO" for AI suggestions
7. Preview → Publish or Schedule

#### 2.4 Social Media Generator

**`SocialMediaGenerator` Component** (`src/components/admin/SocialMediaGenerator.tsx`)

**Features:**
- Topic/theme input
- Platform multi-select (LinkedIn, Facebook, Instagram, X)
- Content type selector (announcement, story, product, educational)
- Campaign association dropdown
- **Model selector** (with cost preview)
- "Generate Posts" button
- Results display:
  - Platform-specific previews (visual mockups)
  - Character count per platform
  - Hashtag suggestions
  - Optimal posting time
  - Image/video suggestions
- Batch regenerate options
- Edit individual posts
- "Schedule All" or "Save as Draft"
- Export to CSV/JSON

**Special Features:**
- **Quick Post Mode:** Generate single urgent post in <30 seconds
- **Bulk Generation:** Create 7-14 day content batch
- **Variation Generator:** 3 versions of same post for A/B testing

#### 2.5 Content Calendar

**`ContentCalendar` Component** (`src/components/admin/ContentCalendar.tsx`)

**Features:**
- Month/Week view toggle
- Drag-and-drop post scheduling
- Color-coded by:
  - Platform (blue=LinkedIn, red=Facebook, etc.)
  - Campaign (campaign-specific colors)
  - Status (draft=gray, scheduled=yellow, published=green)
- Click post to edit
- Right-click for quick actions:
  - Edit, Duplicate, Reschedule, Delete, Regenerate with AI
- "AI Generate Calendar" button:
  - Opens wizard for campaign details
  - Select date range, platforms, frequency
  - Choose AI model
  - Preview suggested calendar
  - Approve → Auto-creates draft posts
- Filter by platform, campaign, status
- Export calendar view

#### 2.6 Campaign Manager

**`CampaignManager` Component** (`src/components/admin/CampaignManager.tsx`)

**Features:**
- Create new campaigns:
  - Name, description, theme
  - Date range
  - Target platforms
  - Content frequency
- Campaign list with stats:
  - Total posts created
  - Published vs drafts
  - Engagement metrics (future)
- Bulk actions:
  - Generate campaign content with AI
  - Schedule all posts
  - Archive campaign
- Campaign templates:
  - "Made in Africa Week"
  - "Vendor Spotlight"
  - "Trade Tips Tuesday"
  - "Success Story Saturday"

#### 2.7 Analytics Dashboard

**`ContentAnalytics` Component** (`src/components/admin/ContentAnalytics.tsx`)

**Features:**
- **AI Usage Metrics:**
  - Total content pieces generated
  - Tokens used by model
  - Cost breakdown (Gemini vs OpenAI)
  - Response time averages
  - Success rate
- **Content Performance:**
  - Most generated content types
  - Blog posts published
  - Social posts created
  - Campaigns active
- **Model Comparison:**
  - Side-by-side quality ratings (admin feedback)
  - Cost efficiency
  - Speed comparison
- **Cost Alerts:**
  - Monthly spend tracking
  - Budget threshold warnings
  - Rate limit notifications

### Phase 3: Frontend Integration

#### 3.1 Dynamic Blog Page

**Update `src/pages/Blog.tsx`:**
- Remove hardcoded posts
- Fetch from `blog_posts` table
- Implement pagination (10 posts per page)
- Add search functionality (title, content, keywords)
- Filter by category, date
- Sort by: newest, popular, trending
- SEO optimization (meta tags from DB)
- Social sharing buttons
- Related posts suggestions (AI-powered, future)

#### 3.2 Individual Blog Post Page

**Create `src/pages/BlogPost.tsx`:**
- Dynamic route: `/blog/:slug`
- Fetch single post by slug
- Render rich text content
- Display metadata (author, date, read time)
- Social sharing buttons
- Related posts section
- Comments section (future)
- SEO meta tags (title, description, keywords)
- Open Graph tags for social sharing

#### 3.3 Public API Endpoints

**Create Edge Functions:**
- `blog-posts-public` - Returns published posts (paginated)
- `blog-post-by-slug` - Returns single post
- `blog-rss-feed` - RSS feed generation
- All functions are public (verify_jwt = false)

### Phase 4: AI System Prompts

#### 4.1 Blog Generation System Prompt

```
You are AfriXport's Content Creator AI, specialized in African trade, exports, and entrepreneurship.

Brand Identity:
- Name: AfriXport
- Tagline: Connecting Africa to Global Markets
- Mission: Empower African businesses to export globally through technology and partnerships

Voice Guidelines:
- Professional yet warm and approachable
- Inspiring and optimistic (Africa-positive narrative)
- Action-oriented and practical
- Culturally grounded yet globally relatable
- Data-driven when possible, storytelling when impactful

Target Audience:
- African entrepreneurs and exporters (SMEs)
- International buyers and importers
- Development partners and investors
- E-commerce professionals
- Trade facilitators and logistics providers

Content Structure Requirements:
- Clear H2 and H3 headings (SEO-optimized)
- Short paragraphs (2-3 sentences max)
- Bullet points for lists
- Include real-world examples
- Add actionable takeaways
- Use transition words for flow
- Write in active voice
- Avoid jargon, explain technical terms

SEO Guidelines:
- Naturally integrate target keywords
- Write compelling meta descriptions (150-160 chars)
- Include internal linking suggestions
- Suggest relevant hashtags (#AfricanExports, #TradeAfrica)

Content Themes to Highlight:
- Export readiness and market access
- Digital trade enablement
- African product excellence
- Success stories and case studies
- Trade policy and compliance
- Logistics and shipping solutions
- Payment and financial inclusion
- Market intelligence and trends

Always maintain authenticity and respect for African diversity, culture, and business practices.
```

#### 4.2 Social Media System Prompts

**LinkedIn Prompt:**
```
Platform: LinkedIn (Professional Network)
Character Limit: 3000 (optimal: 150-300)
Tone: Thought leadership, professional insights, B2B focus
Format: Paragraph style, professional narrative
Include: Industry insights, data points, professional calls-to-action
Hashtags: 5-10 professional tags (#AfricanTrade, #ExportBusiness, #GlobalTrade)
Best Times: Tuesday-Thursday, 8-10 AM, 12-2 PM, 5-6 PM (GMT)
```

**Facebook Prompt:**
```
Platform: Facebook (Community Engagement)
Character Limit: 63,206 (optimal: 100-250)
Tone: Community-driven, storytelling, engaging
Format: Conversational, emotionally connecting
Include: Questions, community stories, calls for engagement
Hashtags: 3-5 accessible tags
Best Times: Wednesday-Friday, 1-3 PM (GMT)
```

**Instagram Prompt:**
```
Platform: Instagram (Visual Storytelling)
Character Limit: 2,200 (optimal: 50-150)
Tone: Inspirational, visual-first, authentic
Format: Short, punchy, emoji-friendly
Include: Visual descriptions, motivational quotes, cultural pride
Hashtags: 10-15 mix of popular and niche tags
Best Times: Daily, 11 AM - 1 PM, 7-9 PM (GMT)
```

**Twitter/X Prompt:**
```
Platform: X/Twitter (Quick Insights)
Character Limit: 280
Tone: Concise, timely, conversational
Format: Thread-ready, single tweet or 3-tweet thread
Include: Quick facts, breaking news, trending topics
Hashtags: 2-3 relevant tags
Best Times: Daily, 9 AM, 12 PM, 5 PM (GMT)
```

### Phase 5: Implementation Sequence

#### Week 1: Foundation
1. Create all database tables with RLS policies ✅
2. Build `generate-blog-content` edge function ✅
3. Create `AIModelSettings` component ✅
4. Build `BlogPostEditor` with AI integration ✅
5. Update Blog.tsx for dynamic content ✅

#### Week 2: Social Media
1. Build `generate-social-content` edge function ✅
2. Create `SocialMediaGenerator` component ✅
3. Build `ContentCalendar` component ✅
4. Create `CampaignManager` component ✅

#### Week 3: Advanced Features
1. Build `generate-content-calendar` edge function ✅
2. Build `optimize-content-seo` edge function ✅
3. Create `test-ai-models` edge function ✅
4. Build `ContentAnalytics` dashboard ✅
5. Add BlogPost.tsx with dynamic routes ✅

#### Week 4: Polish & Testing
1. Integration testing across all components ✅
2. UI/UX refinements ✅
3. Performance optimization ✅
4. Documentation ✅
5. Admin training materials ✅

### Phase 6: Key Features Summary

**For Admins:**
✅ Switch between AI models anytime (Gemini vs OpenAI)
✅ See cost estimates before generation
✅ Test models side-by-side
✅ Track usage and costs
✅ Generate blog posts in <5 minutes
✅ Create 7-14 day social calendars in <3 minutes
✅ Optimize existing content with AI
✅ Manage campaigns with AI assistance
✅ Schedule content across platforms
✅ Export content for external tools

**For AfriXport:**
✅ Consistent brand voice across all content
✅ SEO-optimized blog posts
✅ Platform-specific social media optimization
✅ Reduced content creation time by 90%
✅ Scalable content production
✅ Cost-effective (use free Gemini models)
✅ Analytics-driven content strategy
✅ Flexible AI model selection

### Phase 7: Security & Compliance

**Access Control:**
- Only admins can access Content Creator
- RLS policies enforce data security
- Audit logs for all AI generations

**Content Moderation:**
- Admin review before publishing
- Draft → Published workflow
- Version history tracking
- Rollback capabilities

**Rate Limiting:**
- Handle 429 errors gracefully
- Display rate limit warnings
- Queue requests during high traffic
- Show alternative model suggestions

**Cost Management:**
- Display real-time cost estimates
- Set monthly budget limits
- Alert when approaching limits
- Optimize for free models (Gemini)

### Phase 8: Success Metrics

**Efficiency Gains:**
- Blog post creation: 2+ hours → 5 minutes (95% reduction)
- Social media batch: 1+ hour → 3 minutes (97% reduction)
- Content calendar: 4+ hours → 5 minutes (98% reduction)

**Quality Metrics:**
- Admin approval rate target: >85%
- SEO score improvement: +30%
- Engagement rate improvement: +20% (future tracking)

**Cost Savings:**
- Using Gemini (free): $0/month during promo
- Alternative (manual): ~$2000/month (content writer)
- ROI: Immediate during free period

This plan gives you complete flexibility to:
1. **Start with free Gemini models** (save costs during promo period)
2. **Test OpenAI models** side-by-side to compare quality
3. **Switch models anytime** based on budget, quality, or speed needs
4. **Track costs transparently** so you know exactly what you're spending
5. **Optimize for your needs** - use Gemini for high-volume, OpenAI for premium content

The system is built to make model switching seamless - you can even override the default model on a per-request basis if you want to use OpenAI for a particularly important blog post while using Gemini for routine social media posts.

Would you like me to start implementation with Phase 1 (Database + Blog Generation)?
