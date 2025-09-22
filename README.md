

# ScheduleBud: AI-Powered Academic Management Platform

[![Production Ready](https://img.shields.io/badge/Status-Production%20Ready-green)](https://schedulebud.app/) [![React](https://img.shields.io/badge/React-18.2.0-blue)](https://reactjs.org/) [![TypeScript](https://img.shields.io/badge/TypeScript-5.8.3-blue)](https://www.typescriptlang.org/) [![Supabase](https://img.shields.io/badge/Supabase-Edge%20Functions-blue)](https://supabase.com/) [![AI Powered](https://img.shields.io/badge/AI-Gemini%20Flash%202.0-purple)](https://deepmind.google/technologies/gemini/)

**Live Application:** [**https://schedulebud.app**](https://schedulebud.app/)

## Project Overview

ScheduleBud is an AI-native, full-stack productivity platform designed to help students manage their academic lives. It solves the problem of fragmented academic tools by syncing Canvas LMS assignments from `.ics` calendar links, parsing syllabi with AI, providing a smart chatbot, and offering real-time task management in a single, intuitive interface. ScheduleBud is built for the modern student who needs to stay organized and efficient.

## Live Demo

A live video demo can be found here: [ScheduleBud Demo](https://youtu.be/TEmODMrIAvg)

## Tech Stack

| Frontend | Backend | AI/ML | Infrastructure | Payments | Testing |
|---|---|---|---|---|---|
| ![React](https://img.shields.io/badge/-React-61DAFB?logo=react&logoColor=white) | ![Supabase](https://img.shields.io/badge/-Supabase-3FCF8E?logo=supabase&logoColor=white) | ![Google Gemini](https://img.shields.io/badge/-Google%20Gemini-8A2BE2?logo=google&logoColor=white) | ![Render](https://img.shields.io/badge/-Render-46E3B7?logo=render&logoColor=white) | ![Stripe](https://img.shields.io/badge/-Stripe-6772E5?logo=stripe&logoColor=white) | ![Playwright](https://img.shields.io/badge/-Playwright-2EAD33?logo=playwright&logoColor=white) |
| ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white) | ![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-4169E1?logo=postgresql&logoColor=white) | ![Hugging Face](https://img.shields.io/badge/-Hugging%20Face-FFD000?logo=huggingface&logoColor=white) | | | |
| ![Tailwind CSS](https://img.shields.io/badge/-Tailwind%20CSS-06B6D4?logo=tailwind-css&logoColor=white) | ![Deno](https://img.shields.io/badge/-Deno-000000?logo=deno&logoColor=white) | ![pgvector](https://img.shields.io/badge/-pgvector-2F69AD?logo=postgresql&logoColor=white) | | | |

## High-Level System Design

My design for ScheduleBud was driven by five core principles, essential for a solo engineer building a production-ready application:

1.  **Serverless-First:** Eliminate infrastructure management by using serverless functions for all custom backend logic, ensuring automatic scaling and reducing operational overhead.
2.  **Maximize Velocity with Managed Services:** Leverage a Backend-as-a-Service (Supabase) for commodity components like user auth and a basic CRUD API, allowing me to focus engineering effort on unique, value-adding features.
3.  **Security at the Core:** Implement security at the lowest possible layer (the database) using Row-Level Security (RLS) and create secure boundaries for all external service interactions.
4.  **Resilience and Fallbacks:** Acknowledge that external services and complex processes can fail. The system must be designed to be self-healing, handle errors gracefully, and use fallback mechanisms wherever possible.
5.  **Cost Optimization:** Design the system to be intelligent about resource usage, especially expensive AI API calls, by implementing smart routing and caching.

Based on these principles, I architected ScheduleBud as a decoupled, multi-tier system. It consists of a React SPA frontend, a central BaaS platform, and a suite of specialized serverless microservices that handle complex, asynchronous tasks.

```mermaid
graph LR
    subgraph "User's Device"
        A[Frontend: React SPA]
    end

    %% Main Backend Platform Subgraph
    subgraph P1[ ]
        direction LR
        style P1 fill:#f9f7e8,stroke:#e0dcca

        %% Manual Title Node for Backend Platform
        P_Title["<b>Backend Platform (Supabase)</b>"]
        style P_Title fill:none,stroke:none,font-weight:bold,font-size:1.1em

        B[API Gateway & Auth]
        C[PostgreSQL Database w/ RLS]
        E[File Storage]
        
        %% Serverless Microservices Group
        subgraph F_Group [ ]
            direction LR
            style F_Group fill:#f9f7e8,stroke:#d3d0b8,stroke-dasharray: 5 5

            %% Manual Title Node for Serverless Microservices
            F_Title["<b>Serverless Microservices (Edge Functions)</b>"]
            style F_Title fill:none,stroke:none,font-weight:normal,font-style:italic,font-size:1em

            F1["ask-chatbot (RAG Pipeline)"]
            F2["embed-file (Data Ingestion)"]
            F3["stripe-webhook (Payments)"]
            F4["ai-analysis (Structured Extraction)"]
            F5["canvas-sync (ICS Calendar Parsing)"]
        end
    end

    subgraph "Third-Party Services"
        G[Google Gemini API]
        H[Stripe API]
        I["Resend (SMTP)"]
        J[Hugging Face API]
        K[Canvas LMS ICS Feeds]
    end

    %% Define connections
    A -- "API Calls & Real-time Stream" --> B
    
    B -- "Invokes" --> F1
    B -- "Invokes" --> F2
    B -- "Invokes" --> F3
    B -- "Invokes" --> F4
    B -- "Invokes" --> F5

    B -- "Manages Auth, RLS, Storage" --> C
    B -- "Manages Auth, RLS, Storage" --> E
    B -- "SMTP Integration" --> I

    F1 -- "Vector Search (RPC)" --> C
    F1 -- "Streaming & Non-Streaming Calls" --> G
    
    F2 -- "File Download" --> E
    F2 -- "Embedding Generation" --> J
    F2 -- "Writes Embeddings" --> C

    F3 -- "Webhook Events" --> H
    F3 -- "Updates Subscription" --> C

    F4 -- "Structured Data Calls" --> G

    F5 -- "Fetches ICS Calendar Data" --> K
    F5 -- "Writes Parsed Tasks" --> C
```

## Key Architectural Features & Implementations

### 1. The Stateful, Context-Aware AI Chatbot (RAG Pipeline)
**Feature:** A chatbot that provides personalized, context-aware, and accurate answers based on a user's private data, conversation history, and real-time task schedule.

**Technical Implementation:** I architected a complete **Retrieval-Augmented Generation (RAG)** pipeline as a serverless function. It uses a `classifyQueryIntent` algorithm to intelligently decide whether to perform an expensive vector search, a simple database query, or a direct LLM call, optimizing cost and latency. The system is self-healing: if a document search fails because files aren't processed, it auto-triggers the embedding function and retries. For a seamless UX, it uses a streaming connection to the Gemini API and serves the response via Server-Sent Events (SSE), with a robust buffer-based parser to prevent text truncation.

**Code Snippet (Query Intent Classification for Cost Optimization):**
```typescript
function classifyQueryIntent(
  query: string, 
  hasClassContext: boolean, 
  hasConversationHistory: boolean
): 'document_search' | 'task_related' | 'general_knowledge' | 'conversational' {
  const normalizedQuery = query.toLowerCase().trim();
  
  // Indicator keywords for different categories
  const documentIndicators = ['syllabus', 'lecture', 'reading', 'according to'];
  const taskIndicators = ['due', 'deadline', 'upcoming', 'overdue', 'my tasks'];
  
  // Priority scoring system to determine the most likely intent
  let documentScore = 0;
  let taskScore = 0;

  documentIndicators.forEach(ind => normalizedQuery.includes(ind) ? documentScore += 2 : null);
  taskIndicators.forEach(ind => normalizedQuery.includes(ind) ? taskScore += 2 : null);

  if (hasClassContext && documentScore === 0) documentScore += 1;
  if (hasConversationHistory && normalizedQuery.length < 30) return 'conversational';

  // Determine the highest score to select the query type
  const maxScore = Math.max(documentScore, taskScore);
  
  if (maxScore < 2) {
    if (hasClassContext) return 'document_search';
    return 'general_knowledge';
  }
  
  return documentScore > taskScore ? 'document_search' : 'task_related';
}
```

### 2. The Secure Data Ingestion & Embedding Pipeline
**Feature:** A secure pipeline to process user-uploaded syllabi (PDFs/DOCX), extract their content, and transform them into searchable vector embeddings.

**Technical Implementation:** This serverless function is designed for resilience and security. It uses a two-stage parsing system, trying a fast library first and then falling back to the more robust `pdfjs-dist` to maximize success. All extracted text is sanitized and validated against security patterns before processing. To ensure data integrity, the function is **idempotent**, deleting any stale embeddings for a file before generating new ones.

### 3. The Event-Driven Payments System
**Feature:** A reliable system to manage user subscriptions and synchronize payment status with Stripe.

**Technical Implementation:** I designed an asynchronous, event-driven system using Stripe webhooks. A dedicated serverless function acts as a secure endpoint. Its most critical task is to **cryptographically verify the webhook's signature** before processing any event. It then acts as a state machine, listening for events like `invoice.payment_failed` and updating the user's `subscription_status` in the PostgreSQL database, ensuring data integrity between my app and the payment processor.

### 4. The Canvas LMS Integration System (ICS Calendar Parsing)
**Feature:** Seamless synchronization of Canvas LMS assignments by parsing ICS calendar feeds, automatically importing due dates, assignment names, and course information into ScheduleBud.

**Technical Implementation:** I built a serverless edge function that fetches and parses Canvas ICS calendar feeds server-side, completely bypassing CORS restrictions that plague client-side implementations. The system implements intelligent duplicate detection using Canvas UIDs to ensure assignments aren't duplicated on subsequent syncs. It includes robust error handling with exponential backoff retries and supports bulk assignment processing for courses with heavy assignment loads. The parser extracts course codes, assignment details, and due dates, automatically creating tasks with proper class associations and Canvas metadata for seamless integration.

### 5. The AI-Powered Document Analysis Engine
**Feature:** Intelligent extraction of structured academic metadata from user-uploaded documents, automatically detecting courses, assignments, due dates, and academic requirements from syllabi and course materials.

**Technical Implementation:** I designed a specialized serverless edge function that leverages Google Gemini 2.0 Flash's structured output capabilities to parse academic documents and extract actionable task data. The system implements sophisticated JSON truncation recovery algorithms and domain-specific parsing for academic terminology (especially nursing/healthcare: ATI, HESI, clinical engagement). It handles complex date extraction from weekly ranges, checkbox assignments, and various academic formats, then stores the structured extraction results in PostgreSQL for integration with the task management system. This is separate from the embedding pipeline and focuses purely on extracting structured academic metadata rather than creating searchable vectors.

## Key Challenge & Solution: Multi-Tenant Data Security

**Challenge:** One of the biggest challenges was designing a system where multiple users could store their personal academic data with the absolute guarantee that their information would remain private. A simple mistake in a query could potentially expose one user's data to another.

**Solution:** I solved this by making data ownership a core principle of the database schema and enforcing security at the database level with **Row-Level Security (RLS)**. Every table containing user-generated content has a `user_id` column that links to the `auth.users` table.

By implementing RLS policies on all relevant tables, I created a fundamental security model that cannot be bypassed by application-level code. This approach ensures that even if there were a bug in an API call, the database itself would still prevent unauthorized data access, effectively creating a powerful security backstop.

**Code Snippet (PostgreSQL RLS Policy):**
```sql
-- Enable Row-Level Security on the 'tasks' table
ALTER TABLE public.tasks ENABLE ROW LEVEL SECURITY;

-- Create a policy that allows users to see only their own tasks
CREATE POLICY "Users can view their own tasks"
ON public.tasks FOR SELECT
USING (auth.uid() = user_id);

-- Create a policy that allows users to insert tasks for themselves
CREATE POLICY "Users can create their own tasks"
ON public.tasks FOR INSERT
WITH CHECK (auth.uid() = user_id);
```
