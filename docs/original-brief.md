CLAUDE.md — AI Character Portfolio Platform
Project Overview
Build a SaaS platform for creators and businesses that operate multiple AI-generated social media characters.
The core idea is not to compete directly with AI video/avatar generation platforms. Instead, this product is the business operating system and analytics layer for AI characters.
Users may manage 5, 10, 15, or more recurring AI characters. Each character should feel like an independent digital asset/profile with its own identity, content library, social accounts, analytics, revenue, expenses, and AI-generated insights.
Working product concept name:
AI Character Portfolio Platform
Do not lock the architecture to this name. Make branding configurable.
Product Vision
The application should feel like a combination of:
	●	A modern SaaS analytics dashboard
	●	A social media management platform
	●	A portfolio/investment dashboard
	●	A character management system
	●	A business intelligence tool
	●	A lightweight “digital studio” for AI character businesses
The UI should make a creator feel like they are managing a portfolio of digital personalities.
Example characters:
	●	Luna
	●	Sofia
	●	Mia
	●	Ava
	●	Isabella
A user should be able to switch between characters and immediately understand:
	●	How the character is performing
	●	Which platforms generate the most engagement
	●	Which content performs best
	●	Audience growth
	●	Retention
	●	Revenue
	●	Expenses
	●	Profit
	●	Content output
	●	Growth trends
	●	Opportunities for improvement
Primary Product Principle
Character first. Analytics second. Content third.
The character is the central object in the application.
Do NOT build the initial product as a generic social media dashboard with characters added later.
The data model and UX should revolve around:
User → Character → Content → Platform → Performance → Revenue/Expenses → Insights
MVP
Build the first version around the following functionality.
1. Character Portfolio
Create a visually compelling portfolio page showing all characters.
Each character card should include:
	●	Profile image/avatar
	●	Character name
	●	Short description
	●	Status
	●	Platforms
	●	Total followers
	●	Total views
	●	Engagement rate
	●	Revenue
	●	Profit
	●	Recent growth
	●	Number of published videos
Allow the user to:
	●	Create character
	●	Edit character
	●	Archive character
	●	Delete character
	●	Open character dashboard
Character cards should feel premium and visual rather than like spreadsheet rows.
2. Character Profile
Every character gets its own dedicated profile/dashboard.
Example:
/characters/luna
Sections:
Overview
	●	Followers
	●	Views
	●	Engagement
	●	Average watch time
	●	Retention
	●	Revenue
	●	Expenses
	●	Profit
	●	Content published
Growth
Charts for:
	●	Followers over time
	●	Views over time
	●	Engagement over time
	●	Revenue over time
Content
Show videos/posts belonging to that character.
Each content item should display:
	●	Thumbnail
	●	Title/caption
	●	Platform
	●	Date published
	●	Views
	●	Likes
	●	Comments
	●	Shares
	●	Watch time
	●	Retention
	●	Revenue
Platforms
Show performance by:
	●	TikTok
	●	Instagram
	●	YouTube
	●	Facebook
	●	Other supported platforms
3. Content Library
Create a centralized content library.
Each piece of content should be associated with exactly one primary character.
Content fields should include:
	●	Character
	●	Platform
	●	URL
	●	Thumbnail
	●	Caption
	●	Publish date
	●	Views
	●	Likes
	●	Comments
	●	Shares
	●	Saves
	●	Watch time
	●	Average percentage watched
	●	Retention
	●	Revenue
	●	Production cost
	●	Status
Statuses:
	●	Draft
	●	Scheduled
	●	Published
	●	Archived
The architecture should allow additional fields later without requiring a major rewrite.
4. Analytics
Analytics should be one of the strongest parts of the product.
Users should be able to analyze:
Character-level analytics
	●	Total views
	●	Average views
	●	Followers
	●	Follower growth
	●	Engagement rate
	●	Retention
	●	Posting frequency
	●	Revenue
	●	Expenses
	●	Profit
Content-level analytics
Identify:
	●	Best-performing videos
	●	Worst-performing videos
	●	Highest retention
	●	Highest engagement
	●	Highest revenue
	●	Fastest-growing content
	●	Best content topics
	●	Best hooks
	●	Best posting times
Platform comparison
Allow the user to compare the same character across platforms.
Example:
Luna:
TikTok
	●	1.2M views
	●	8.4% engagement
Instagram
	●	640K views
	●	5.9% engagement
YouTube
	●	310K views
	●	7.1% engagement
Do not hard-code these values. They are examples only.
5. Revenue & Expense Tracking
Treat every character as a business asset.
Track revenue sources such as:
	●	Platform monetization
	●	Sponsorships
	●	Affiliate revenue
	●	Product sales
	●	Subscription revenue
	●	Other income
Track expenses such as:
	●	AI video generation
	●	Image generation
	●	Voice generation
	●	Editing
	●	Software subscriptions
	●	Advertising
	●	Contractors
	●	Other production costs
Calculate:
Revenue - Expenses = Profit
Also calculate:
	●	Revenue per video
	●	Cost per video
	●	Profit per video
	●	Revenue per 1,000 views
	●	Cost per 1,000 views
	●	Monthly profit
	●	Character ROI
Make the financial system extensible.
6. AI Insights
This should eventually become a major differentiator.
The platform should analyze historical performance and generate actionable observations.
Examples:
	●	“Luna’s average retention is 18% higher on videos under 20 seconds.”
	●	“Gym-related content has generated 2.3× the average engagement for Sofia.”
	●	“Instagram Reels are generating more followers per 1,000 views than TikTok.”
	●	“Videos posted between 7 PM and 9 PM have historically produced higher retention.”
	●	“This character’s production cost has increased while revenue per video has remained flat.”
IMPORTANT:
These insights must be generated from actual stored data.
Do not fabricate analytics.
The system should clearly distinguish:
	●	Observed data
	●	Calculated metrics
	●	AI interpretation
	●	AI recommendations
7. Character Intelligence
Eventually each character should have a structured “character profile.”
Potential fields:
	●	Name
	●	Age/persona
	●	Description
	●	Personality
	●	Interests
	●	Visual identity
	●	Voice
	●	Content niche
	●	Target audience
	●	Brand guidelines
	●	Common topics
	●	Prompt templates
	●	Negative prompts
	●	Image references
	●	Video references
This information should eventually be usable by AI workflows to maintain character consistency.
Do not overbuild this during the initial MVP.
Create the data model so it can be expanded later.
8. Social Media Integrations
Design the architecture for API integrations even if the first MVP uses mock/imported data.
Potential integrations:
	●	TikTok
	●	Instagram
	●	YouTube
	●	Facebook
Do not pretend an API integration exists if it has not actually been implemented.
Use an integration abstraction such as:
SocialPlatformAdapter
so each platform can eventually implement:
	●	Authentication
	●	Account connection
	●	Content retrieval
	●	Analytics retrieval
	●	Follower retrieval
	●	Publishing where supported
Keep platform-specific code isolated.
9. Dashboard Design
The main dashboard should feel like a command center.
Possible layout:
Sidebar:
	●	Overview
	●	Characters
	●	Content
	●	Analytics
	●	Revenue
	●	Expenses
	●	Insights
	●	Integrations
	●	Settings
Main area:
Portfolio Summary
	●	Total characters
	●	Total followers
	●	Total views
	●	Total revenue
	●	Total expenses
	●	Total profit
Character Performance
Visual cards for each character.
Recent Content
Recently published content.
Performance Trends
Charts.
AI Insights
Recent automatically generated observations.
10. Character Selector
The character selector should be a prominent UI element.
Think of it almost like selecting a character in a game.
Example:
All Characters
Luna
Sofia
Mia
Ava
When selecting a character, the entire dashboard can filter to that character.
Eventually support:
	●	Multi-character comparison
	●	Character groups
	●	Favorites
	●	Search
	●	Sorting
UI / UX DIRECTION
The application should feel modern, premium, futuristic, and extremely polished.
Avoid generic admin-dashboard aesthetics.
Desired characteristics:
	●	Dark/light theme support
	●	Smooth transitions
	●	Beautiful cards
	●	Strong typography
	●	Large character imagery
	●	Clean charts
	●	Subtle animation
	●	Excellent spacing
	●	Responsive design
	●	Mobile-friendly layout
The product should visually communicate:
“I am managing a portfolio of digital characters.”
rather than:
“I am looking at a spreadsheet.”
TECHNICAL ARCHITECTURE
Prefer a modern TypeScript stack unless the existing repository dictates otherwise.
Recommended:
	●	React
	●	TypeScript
	●	Vite or Next.js
	●	Tailwind CSS
	●	Component library where appropriate
	●	Supabase
	●	PostgreSQL
	●	Recharts or another reliable charting library
Keep the architecture modular.
Suggested structure:
src/
  components/
  pages/
  layouts/
  features/
    characters/
    content/
    analytics/
    revenue/
    expenses/
    insights/
    integrations/
  lib/
  services/
  hooks/
  types/
Do not create one giant component.
Separate:
	●	UI
	●	business logic
	●	API/data access
	●	analytics calculations
	●	integrations
DATABASE CONCEPT
Design around these core entities:
users
characters
character_platform_accounts
content
content_metrics
platforms
revenue
expenses
insights
Potential relationships:
User
 └── Characters
      ├── Platform Accounts
      ├── Content
      │    └── Metrics
      ├── Revenue
      ├── Expenses
      └── Insights
Use foreign keys and appropriate indexes.
Avoid duplicating data unnecessarily.
ANALYTICS ENGINE
Create a dedicated analytics layer.
Do not calculate complex metrics directly inside UI components.
Examples:
calculateEngagementRate()
calculateRetentionRate()
calculateFollowerGrowth()
calculateRevenuePerVideo()
calculateCostPerVideo()
calculateProfit()
calculateROI()
calculateViewsPerFollower()
Keep calculations testable.
If a metric cannot be calculated because data is missing, return an explicit unavailable state rather than inventing a number.
MOCK DATA
During development, create realistic mock data.
Include approximately:
	●	5 characters
	●	3–4 platforms
	●	50+ content items
	●	Multiple months of historical metrics
	●	Revenue
	●	Expenses
	●	Different performance profiles
Make characters meaningfully different.
For example:
One character may have:
	●	High views
	●	Low conversion
Another:
	●	Lower views
	●	High engagement
Another:
	●	Strong follower growth
	●	Low monetization
This allows the analytics UI to demonstrate meaningful differences.
Clearly label mock/demo data where appropriate.
IMPORTANT DEVELOPMENT RULES
	1.	Build the smallest useful version first.
	2.	Do not implement fake integrations as if they were real.
	3.	Do not fabricate real-world social analytics.
	4.	Use mock data only for development/demo purposes.
	5.	Keep APIs and integrations modular.
	6.	Keep analytics calculations outside UI components.
	7.	Write reusable components.
	8.	Use TypeScript types consistently.
	9.	Validate user input.
	10.	Handle loading, empty, and error states.
	11.	Make the application responsive.
	12.	Do not over-engineer features that are not needed for the MVP.
	13.	Preserve existing functionality when modifying an existing repository.
	14.	Before making major architectural changes, inspect the existing codebase.
	15.	Prefer incremental implementation over rewriting the entire project.
DEVELOPMENT WORKFLOW
Before coding:
	1.	Inspect the repository.
	2.	Identify the existing framework and architecture.
	3.	Identify existing database configuration.
	4.	Identify existing styling/component systems.
	5.	Identify existing authentication.
	6.	Identify reusable components.
	7.	Determine what already exists before creating replacements.
Then:
	1.	Establish the data model.
	2.	Build character management.
	3.	Build the portfolio dashboard.
	4.	Build character dashboards.
	5.	Build content library.
	6.	Add analytics.
	7.	Add revenue/expense tracking.
	8.	Add AI insights using the analytics layer.
	9.	Add social integrations incrementally.
After every significant feature:
	●	Run the project.
	●	Check for TypeScript errors.
	●	Check for runtime errors.
	●	Test the relevant UI.
	●	Fix regressions before continuing.
PRODUCT ROADMAP
Phase 1 — MVP
	●	Authentication
	●	Character creation
	●	Character portfolio
	●	Character dashboards
	●	Mock content
	●	Basic analytics
	●	Revenue/expense tracking
	●	Responsive UI
Phase 2
	●	Real social integrations
	●	Automated metric synchronization
	●	Advanced analytics
	●	Content scheduling
	●	Better financial reporting
Phase 3
	●	AI insights
	●	Content recommendations
	●	Trend detection
	●	Character performance predictions
	●	Automated reporting
Phase 4
	●	Character generation workflows
	●	Prompt management
	●	Character consistency tools
	●	AI content ideation
	●	Automated production workflows
Phase 5
Potentially evolve into a full:
AI Character Business Operating System
where users can manage the entire lifecycle:
Character
   ↓
Content Idea
   ↓
AI Generation
   ↓
Publishing
   ↓
Analytics
   ↓
Audience
   ↓
Monetization
   ↓
Profit
   ↓
AI Optimization
NORTH STAR
The long-term product should answer one question extremely well:
“Which of my AI characters are actually becoming successful businesses, and why?”

