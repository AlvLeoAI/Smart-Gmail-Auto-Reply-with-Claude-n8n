## Goal

Automate personalized email responses by combining contact data lookup with AI-generated replies, eliminating manual email handling while maintaining authentic, context-aware communication.

## Solution

- **Gmail trigger** monitors incoming emails every minute
- **Contact lookup** retrieves customer history and notes from Data Tables
- **Claude AI** generates personalized HTML email responses based on contact context
- **Python parser** extracts structured subject and body from AI output
- **Automated Gmail reply** sends professional formatted response
- **Smart filtering** only responds to known contacts, preventing spam auto-replies

## Features

- **Context-aware responses**: Uses contact notes and history for personalization
- **HTML email formatting**: Professional, clean email structure
- **Selective automation**: Only responds to contacts in database
- **Claude Sonnet 4.5 integration**: Latest AI model for intelligent responses
- **Structured output parsing**: Reliable extraction of subject and body
- **Real-time monitoring**: 1-minute polling interval

---

## Steps

### 1. Email Monitoring

**Node:** `Gmail Trigger`

**Purpose:** Monitors Gmail inbox for new incoming emails.

**Configuration:**

- **Poll Time:** Every minute
- **Event:** New email received
- **Output:** Email data including sender, subject, body

### 2. Contact Lookup

**Node:** `Get row(s)` (Data Table)

**Purpose:** Searches for sender email in Contacts database.

**Configuration:**

- **Data Table:** Contacts
- **Operation:** Get
- **Condition:** Email = sender's email address
- **Must Match:** All Conditions

**Output:** Contact record with Name, Email, Phone, Notes (if exists)

**Design Decision:** If contact not found, workflow stops here. Only known contacts receive automated responses to prevent spam-like behavior.

### 3. AI Response Generation

**Node:** `Message a model` (Claude)

**Purpose:** Generates personalized email response using contact context.

**Configuration:**

- **Model:** Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)
- **Input:** Contact data + received email

**System Prompt:**

```jsx
You are an AI assistant helping to write professional email responses.

Create a well-formatted HTML email response based on the customer 
information and received email.

Customer Data:
1-Name: {{ $
```

### 4. Response Parsing

**Node:** `Code in Python (Beta)`

**Purpose:** Extracts structured subject and HTML body from Claude's response.

**Logic:**

- Uses regex to extract Subject line
- Finds HTML content between `HTML_BODY_START` and `HTML_BODY_END` markers
- Fallback: If no HTML found, converts plain text with `<p>` and 
`` tags
- Returns JSON with `subject`, `body`, and `body_type`

**Why needed:** Gmail node requires separate subject and body fields

### 5. Email Delivery

**Node:** `Send a message` (Gmail)

**Purpose:** Sends formatted HTML email response to original sender.

**Configuration:**

- **To:** Original sender's email
- **Subject:** Parsed subject from Claude
- **Body:** Parsed HTML body
- **Options:** Attribution disabled for clean signature

---

## Tools Used

- **n8n:** Workflow automation and orchestration
- **Claude Sonnet 4.5:** AI model for response generation (via Anthropic API)
- **Gmail API:** Email monitoring and sending
- **Data Tables:** Contact database storage
- **Python:** Response parsing and text processing

---

## Impact

- **Zero manual email responses** for known contacts
- **Context-aware personalization** using contact history and notes
- **Professional HTML formatting** in all responses
- **Smart filtering** prevents spam auto-replies to unknown senders
- **Sub-2-minute response time** from email receipt to reply
- **Cost-effective:** ~$0.01 per email with Claude API
- **Scalable architecture** can handle multiple contacts simultaneously

---

## Learnings

- **Prompt engineering for structured output:** Using markers like `HTML_BODY_START` ensures reliable parsing
- **Edge case handling:** Workflow intentionally stops for unknown contacts
- **Multi-step data flow:** Combining database lookup → AI generation → parsing → delivery
- **Claude integration best practices:** System prompts with clear format requirements
- **Python parsing in n8n:** Regex patterns for extracting structured data from text
- **Production-ready automation:** Thoughtful design prevents unwanted auto-replies

---

## Database Schema

**Contacts Table:**

- Name (string)
- Email (string) - used for lookup
- Phone_Number (string)
- Notes (string) - context for AI personalization

---

## Bonus: Sales Data Analysis Agent

This workflow also includes a **conversational AI agent** for analyzing sales data using natural language queries.

### Architecture

**Trigger:** Chat interface (When chat message received)

**AI Agent:** Sales Data Agent with Claude Sonnet 4.5 (via OpenRouter)

**Tools Available:**

1. **All Rows** - Returns complete sales dataset
2. **Product Name Query** - Filters by product (Wireless Headphones, Bluetooth Speaker, Phone Case)
3. **Date Query** - Filters by date (YYYY-MM-DD format)
4. **Product ID Query** - Filters by ID (WH001, BS002, PC003)
5. **Calculator** - Performs mathematical operations (ALWAYS used for calculations)

### System Prompt

```jsx
# Overview
You are **Master Sales Data Analyst AI**, an expert in analyzing sales data and answering user queries with precision.  
You have access to the following tools:

1. **AllRows** – returns all rows in the sales database.  
2. **ProductNameQuery** – returns rows filtered by product name.  
   - Valid product names: *Wireless Headphones*, *Bluetooth Speaker*, *Phone Case*.  
3. **DateQuery** – returns rows filtered by a specific date.  
   - Date format: `YYYY-MM-DD` (e.g., `2025-09-15`).  
4. **ProductIDQuery** – returns rows filtered by product ID.  
   - Valid IDs: *WH001* (Wireless Headphones), *BS002* (Bluetooth Speaker), *PC003* (Phone Case).  
5. **Calculator** – use this whenever you need to perform math operations (e.g., sum, averages, totals, percentages). Never attempt math manually.

---

### Core Instructions:
- Always **use the most specific tool** available for the user’s query.  
  - If the query references a product name → use `ProductNameQuery`.  
  - If the query references a product ID → use `ProductIDQuery`.  
  - If the query references a date → use `DateQuery`.  
  - If multiple filters are implied (e.g., “How many Wireless Headphones were sold on 2025-09-15?”), combine multiple queries logically by calling the relevant tools in sequence.  
  - If no filter is given, or you need to explore broadly, use `AllRows`.  
- If calculations are required (sums, averages, revenues, comparisons, etc.), **always send the data to the Calculator tool**.  
- Never assume values—always confirm by retrieving data through the tools.  
- Be concise, clear, and professional in your explanations.

---

### Example Behaviors:
- **User**: “How many Wireless Headphones were sold on 2025-09-15?”  
  - Use `ProductNameQuery` for Wireless Headphones.  
  - Then filter with `DateQuery` for 2025-09-15.  
  - Send results to **Calculator** to sum quantities.  
  - Return the answer with explanation.

- **User**: “What’s the total revenue for product ID BS002?”  
  - Use `ProductIDQuery` for BS002.  
  - Multiply *price × quantity* via **Calculator**.  
  - Provide total revenue.

- **User**: “Show me all sales.”  
  - Use `AllRows` and return the dataset.

---

### Your Mission:
You are the ultimate sales analyst.  
Answer questions with precision, use the right tool every time, and provide clear explanations of how results were derived.  
Always confirm with data before performing analysis.  
Current date/time: {{ $now }}

```

### Sales Data Table Schema

**Sales Table:**

- ProductName (string)
- DateSold (dateTime)
- Price (number)
- QuantitySold (number)
- ProductID (string)
- TotalRevenue (number)

### Agent Capabilities

- Natural language queries about sales performance
- Multi-filter queries (product + date + ID)
- Automatic calculation of totals, averages, revenues
- Intelligent tool selection based on query context
- Conversational interface for non-technical users

### Example Queries

- "What's the total revenue for Bluetooth Speakers?"
- "How many Phone Cases sold on 2025-09-20?"
- "Show me all sales for product WH001"
- "Calculate average price across all products"
    
    

---

## Model Selection & Prompt Engineering Analysis

### Why Claude Sonnet 4.5?

**Selected Model:** `claude-sonnet-4-5-20250929`

**Technical Justification:**

1. **Structured Output Reliability**
    - Excels at following specific format instructions (HTML_BODY_START/END markers)
    - XML-friendly architecture aligns with structured output requirements
    - Lower hallucination rate for template adherence
2. **Context Window Utilization**
    - 200K token context handles full email threads + contact history
    - Critical for maintaining conversation context in email chains
3. **Cost-Performance Balance**
    - $3/MTok input, $15/MTok output
    - Average email response: ~500 input + 300 output tokens = ~$0.006/email
    - Sonnet provides GPT-4 level quality at ~40% lower cost
4. **Tool Use Capability** (Sales Agent)
    - Native function calling for Data Table tools
    - Reliable parameter extraction from natural language
    - Better reasoning for multi-step queries
