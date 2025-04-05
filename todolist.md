# ToDo List for LetMeExplain (letmeexplain.co)

This is a step-by-step, actionable roadmap for building the LetMeExplain SaaS platform. Each section lists specific tasks with clear instructions to ensure a smooth development process.

---

## 1. Project Setup
- [✅] **Initialize Next.js Project:**
  - Create a new Next.js project using the App Directory with:
    ```bash
    npx create-next-app@latest
    ```
  - Install and configure TailwindCSS:
    ```bash
    npm install -D tailwindcss
    npx tailwindcss init
    ```
  - Integrate Shadcn UI components:
    ```bash
    npx shadcn-ui@latest init
    ```
- [✅] **Version Control & CI/CD:**
  - Initialize a Git repository:
    ```bash
    git init
    ```
  - Connect the repository to GitHub (create a repo, then add remote and push):
    ```bash
    git remote add origin <your-repo-URL>
    git push -u origin main
    ```
  - Configure Vercel for continuous deployment by linking the GitHub repository in the Vercel dashboard.
- [✅] **Environment Configuration:**
  - Create a `.env` file in the project root.
  - Install `dotenv-safe`:
    ```bash
    npm install dotenv-safe
    ```
  - Define required environment variables:
    - `SUPABASE_URL`, `SUPABASE_API_KEY`
    - `PINECONE_API_KEY`, `PINECONE_ENVIRONMENT`
    - `OPENAI_API_KEY`, `GOOGLE_API_KEY`, `MISTRAL_API_KEY`
    - `SQUARE_ACCESS_TOKEN`

---

## 2. Backend Setup
- [ ] **Database Setup:**
  - **Local Development:**  
    - Set up PostgreSQL using Docker:
      ```bash
      docker run --name postgres -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres
      ```
  - **Production:**  
    - Create a new Supabase project and retrieve PostgreSQL credentials.
- [ ] **Prisma ORM Integration:**
  - Install Prisma:
    ```bash
    npm install prisma --save-dev
    npx prisma init
    ```
  - **Define the Prisma Schema:**  
    Create or update the `schema.prisma` file:
    ```prisma
    model Business {
      id            String    @id @default(uuid())
      name          String    @unique
      ownerEmail    String
      logoUrl       String?   // Uploaded business logos
      branding      Json?     // Custom branding info (colors, text, etc.)
      qrUrl         String    // Generated QR code URL
      plan          String    @default("basic")
      feedbacks     Feedback[]
      createdAt     DateTime  @default(now())
      updatedAt     DateTime  @updatedAt
    }

    model Feedback {
      id            String    @id @default(uuid())
      businessId    String
      originalText  String
      generatedText String
      selectedText  String
      vectorId      String
      contactEmail  String?   // Encrypted contact info
      contactPhone  String?   // Encrypted contact info
      createdAt     DateTime  @default(now())

      business      Business  @relation(fields: [businessId], references: [id])
    }
    ```
  - Run database migrations:
    ```bash
    npx prisma migrate dev --name init
    ```

---

## 3. Authentication & Multi-Tenancy
- [ ] **User Authentication:**
  - Integrate Supabase Auth (or Clerk) by installing `@supabase/supabase-js` and configuring with your Supabase credentials.
  - Implement secure session management including password hashing and optional MFA.
- [ ] **Tenant Isolation:**
  - Ensure each business is assigned a unique `business_id` during signup.
  - Modify database queries to always filter by `business_id`.
- [ ] **QR Code Integration:**
  - Build an API endpoint (`/api/qr/[business_id]`) using Next.js API routes.
  - Use a library like `qrcode` to generate QR codes.
  - Embed the business context in the QR code URL (e.g., `https://letmeexplain.co/submit?biz=MyCoffeeShop`).

---

## 4. Feedback Submission Flow & AI Integration
- [ ] **Feedback Form:**
  - Create a public feedback form page at `/submit` using Next.js.
  - Parse the `biz` query parameter to preselect the business in the form.
- [ ] **AI Transformation Endpoint:**
  - Develop a Next.js API route at `/api/ai/transform` to interface with LLM APIs.
  - Install and configure SDKs for OpenAI, Google, and Mistral.
  - Create stateless utility functions to call these APIs and standardize the output.
- [ ] **Regeneration Logic:**
  - Limit feedback regeneration to **3 attempts per 24 hours**.
    - Track attempts using IP addresses or client cookies.
  - Enforce a 24-hour delay between regeneration attempts.
- [ ] **Logging:**
  - Log API usage and associated costs in a Supabase table (e.g., `ApiLogs`) with fields like `api_name`, `input`, `output`, `cost`, `timestamp`.

---

## 5. Data Storage & Analytics
- [ ] **Feedback Storage:**
  - Save feedback data in Supabase in JSONL format.
  - **Example Format:**
    ```json
    {
      "original": "Your place is filthy...",
      "selected": "The location could benefit from more frequent cleaning...",
      "submittedAt": "2025-04-05T18:00:00Z"
    }
    ```
- [ ] **Pinecone Integration:**
  - Install `@pinecone-database/pinecone` and configure with your API keys.
  - Generate embeddings for feedback text.
  - Upsert embeddings to Pinecone with metadata (`business_id`, `timestamp`).
- [ ] **Analytics & Keywords:**
  - Extract keywords from feedback (e.g., “cheese” for a pizzeria) using an NLP library or LLM.
  - Store keywords in Supabase.
  - Develop a dropdown filter on the dashboard to display top-discussed keywords.
  - Implement sentiment analysis and trend summaries.
- [ ] **Automated Alerts:**
  - Set up email and in-app notifications using Supabase Edge Functions (or a similar service) to alert business owners when negative feedback exceeds a threshold.

---

## 6. Dashboard Development
- [ ] **Dashboard Layout:**
  - Create a dashboard page at `/dashboard` using Next.js and TailwindCSS.
  - Implement a sidebar with navigation links:
    - Dashboard, Comments, Billing, Settings, Sign Out.
- [ ] **Dashboard Components:**
  - **Dashboard Cards:**  
    - Display alerts, latest feedback, sentiment trends, and storage usage.
  - **Comments Section:**  
    - Show a table of feedback pairs (crass vs. refined).
    - Include filtering options by keywords.
  - **Billing Section:**  
    - Display subscription details and billing history from the Square API.
  - **Settings:**  
    - Allow business owners to update business details, upload logos/branding, and regenerate QR codes.
- [ ] **Export Feature:**
  - Add an export button to download feedback data in JSON, Excel, or .txt formats using libraries like `xlsx`.

---

## 7. Billing & Payment Integration
- [ ] **Subscription Model:**
  - Implement a 7-day free trial.
  - Transition to paid plans:
    - **Base Plan:** $10/month (25GB storage)
    - **Tier 1:** $50/month (customized LLM insights)
    - **Tier 2:** $100/month (human-AI analysis, detailed weekly reports)
- [ ] **Square API Integration:**
  - Install `@square/web-sdk` and integrate it for subscriptions and recurring payments.
  - Set up webhooks to monitor subscription status and update user quotas.
- [ ] **Future Marketing:**
  - Plan an advertising module to enable merchants to target businesses based on feedback trends.

---

## 8. Security & Compliance
- [ ] **Input Sanitization & Validation:**
  - Use a library like `sanitize-html` to clean all user inputs.
- [ ] **Data Encryption:**
  - Encrypt sensitive fields (e.g., `contactEmail`, `contactPhone`) using AES-256 via Node.js’s `crypto` module.
  - Decrypt data only on the frontend after proper authentication.
- [ ] **Session & Transport Security:**
  - Enforce HTTPS in production (via Vercel settings).
  - Implement rate limiting on sensitive endpoints (e.g., `/api/ai/transform`) using a package like `express-rate-limit`.
- [ ] **Audit Logs:**
  - Create a Supabase table (`AuditLogs`) to log user actions with fields like `user_id`, `action`, and `timestamp`.
- [ ] **Regular Security Audits:**
  - Schedule quarterly security audits and penetration tests (using tools like OWASP ZAP).

---

## 9. Testing & Deployment
- [ ] **Testing:**
  - Write unit tests for critical functions using Jest.
  - Develop integration tests for the feedback submission and regeneration flows.
  - Set up end-to-end tests using Cypress:
    ```bash
    npm install cypress --save-dev
    ```
- [ ] **Deployment:**
  - Deploy the Next.js frontend on Vercel.
  - Use Vercel’s CI/CD pipeline for automated deployments from GitHub.
  - Monitor application performance and error logs via Vercel’s analytics.

---

## 10. Future Enhancements
- [ ] **User Engagement:**
  - Add an in-app feedback form at `/feedback` for app experience surveys.
  - Integrate push notifications using a service like Firebase.
- [ ] **Additional Marketing Tools:**
  - Expand the advertising module with a prototype.
  - Explore multi-language support and additional LLM integrations.
- [ ] **Enhanced Analytics:**
  - Refine keyword extraction and sentiment analysis with advanced NLP tools.
  - Implement more advanced filtering and reporting options in the dashboard.

---

## Final Steps
- [ ] Review and finalize all environment configurations.
- [ ] Validate the complete feedback flow: QR scan → form submission → AI transformation → dashboard display.
- [ ] Ensure all security, encryption, and audit measures are active.
- [ ] Deploy to Vercel and schedule an initial security audit.
- [ ] Collect initial user feedback via a post-launch survey.

---

*Let’s get coding and launch LetMeExplain with confidence!* 🚀