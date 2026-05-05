# Product Requirements Document (PRD)
## Project: Feast & Fest (Mood-Based Daily Meal & Experience Planner)

### 1. Introduction: The "Non-Formal" Pitch
Imagine waking up, feeling a bit gloomy, and wanting a day that just cheers you up without you having to plan it. **Feast & Fest** does exactly that. You tell the app how you feel, and it curates a complete, timeline-based itinerary for your day. It schedules your breakfast, lunch, snacks, and dinner using the **Swiggy Food API**, and throws in a fun evening activity (like a movie or concert) using the **Swiggy Dineout API**. 

The best part? You review the plan, make tweaks, pay *once* for the food upfront, and the app automatically places the orders at the exact right time throughout the day. It's like having a personal assistant for your cravings and entertainment!

---

### 2. Core User Flow
1. **Authentication:** User must log in or sign up (via phone OTP, email, or social login) to access personalized planning and ensure secure payment processing.
2. **Mood Input:** Logged-in user inputs their current mood, feelings, or vibe for the day (e.g., "Feeling stressed and need comfort food," or "Energetic and want to explore").
3. **AI Generation:** The LLM processes the mood and generates a suggested full-day timeline based on the user's saved dietary preferences and location.
4. **Review & Customize:** The user sees the timeline (Breakfast, Lunch, Event, Dinner) and can swap out specific dishes, restaurants, or events.
5. **Checkout & Payment:**
    * **Food Wallet (One-Time Pay):** User pays a lump sum for all scheduled food orders.
    * **Instant Pay:** User pays immediately for the locked-in event/Dineout reservation.
6. **Autopilot Execution:** As the day progresses, the app automatically places food orders via the delivery API at the scheduled times. Funds are deducted from the pre-paid temporary wallet.
7. **Delivery & Enjoyment:** User gets real-time notifications when their food is ordered and on the way.

---

### 3. Formal Functional Requirements

#### 3.1. Authentication & Identity
* **Secure Login:** Phone-based OTP or OAuth (Google/Apple) integration to securely identify users before they interact with personalized API data.
* **Profile Management:** Ability to save dietary restrictions, favorite cuisines, and default delivery addresses.

#### 3.2. User Interface (Frontend)
* **Mood Capture Engine:** A text box or interactive UI (emojis, sliders) to capture user sentiment.
* **Timeline Dashboard:** A visually appealing vertical or horizontal timeline displaying the day's itinerary.
* **Customization Module:** Ability to click on a suggested meal/event and replace it using search or alternative suggestions.
* **Unified Checkout Page:** A clear breakdown of costs: Total Food Cost + Total Event Cost. 
* **Live Tracker:** A dedicated page to track today's scheduled events and active deliveries.

#### 3.3. Automated Ordering Mechanism
* **Pre-payment/Escrow System:** The system must securely hold the "Total Food Cost" paid during checkout.
* **Order Schedulers:** Background jobs must wake up 30-45 minutes before the scheduled meal time to initiate the actual API call to Swiggy/Zomato.
* **Ledger/Deduction:** Upon successful order placement, the exact amount is deducted from the user's daily prepaid pool. Refunds for failed orders must be credited back instantly.

#### 3.4. AI & Context Engine
* **LLM Orchestration:** Convert natural language mood into searchable food/event tags (e.g., "Sad" -> "Ice Cream, High-carb, Comedy show").
* **Constraint Checking:** Ensure meal suggestions align with restaurant operating hours and delivery serviceability.

#### 3.5. API Integrations (Swiggy MCP)
* **Food Delivery API:** To fetch menus, validate serviceability, and trigger orders. *(Note: While Zomato was mentioned for final fulfillment, standardizing on Swiggy MCP ensures streamlined architecture).*
* **Dineout/Events API:** To fetch ongoing concerts, movies, or table reservations and process instant bookings.

---

### 4. Technical Architecture

#### 4.1. Tech Stack Overview
* **Frontend:** React.js (or Next.js for better SEO/performance), styled with Tailwind CSS.
* **Backend:** Node.js with Express (or Java Spring Boot if enterprise-grade typing/threading is preferred). RESTful APIs for client-server communication.
* **Database:** PostgreSQL (for transactional data like payments, user profiles, and orders) + Redis (for caching sessions and API rate-limiting).
* **Authentication:** JWT (JSON Web Tokens) or session-based auth via tools like Firebase Auth, Auth0, or custom implementation.
* **AI/LLM:** OpenAI GPT-4o or Google Gemini API (via Swiggy MCP if acting as a tool/agent).

#### 4.2. Infrastructure & Scheduling (The "Heavy Lifting")
* **Message Broker / Queues:** RabbitMQ or AWS SQS / BullMQ (if using Node). 
    * *Why?* When a user confirms a daily plan, the backend pushes individual "Order Jobs" into a delayed queue.
* **Job Scheduler:** Cron jobs or Quartz Scheduler (Java). The worker picks up the delayed job (e.g., "Order Lunch at 12:30 PM") and executes the API call to Swiggy.
* **Cloud Hosting:** AWS (ECS/EKS) or Google Cloud Platform for scalable deployment.

#### 4.3. Payment Flow Architecture
1. **Payment Gateway:** Stripe / Razorpay.
2. **Transaction Split:** * *Immediate Capture:* Event ticket cost is captured and settled immediately.
    * *Auth & Capture (or Wallet Credit):* Food cost is either authorized for the day (and captured per order) OR added to a temporary daily wallet on the app.
3. **Reconciliation:** At the end of the day, any unspent money (due to unavailable items or failed deliveries) is refunded to the user's original payment method.

---

### 5. Edge Cases & Constraints to Handle
* **Item Unavailability:** What if the scheduled dish goes out of stock by the time the automated order triggers? (Solution: Define a fallback item or auto-refund the specific meal amount).
* **Delivery Delays:** Real-time sync with delivery partners to adjust subsequent schedules if one meal is severely delayed.
* **Dynamic Pricing:** Prices on delivery apps fluctuate. The app must lock in prices at checkout or include a small buffer margin in the one-time payment.

---

### 6. Phases of Development
* **Phase 1 (MVP):** Basic authentication, mood input, LLM static suggestions, manual checkout.
* **Phase 2:** Integration with Swiggy MCP, dynamic timeline generation, and basic queue scheduler.
* **Phase 3:** Automated background ordering, daily wallet/payment system.
* **Phase 4:** Advanced analytics, user history preferences, and edge-case handling (refunds, fallbacks).
