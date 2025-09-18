# OpenProfile.ai — Architecture

**OpenProfile.ai** is a framework that lets AI agents and websites securely access **user-owned data** — but *only with the user’s permission*.  
The goal: deliver **truly personalized** experiences without sacrificing **security** or **privacy**.

It’s **modular** — so it works whether you’re building:
- A standalone AI agent,
- An embedded feature inside a website,
- Or an app that needs external user context.

At the heart of OpenProfile.ai are **two main building blocks**:

## 1. Gateway — *The Secure Middleman*

Think of the **Gateway** as a **stateless, secure messenger** between:
- An **AI agent / website** asking for data
- And the **places that hold the data** (Fact Pods)

**What it does:**
- Handles authentication (OAuth 2.0)
- Requests “facts” (pieces of data)
- Never stores your data — just passes it through
- Can run as:
    - **Standalone** — separate service for multiple agents
    - **Embedded** — built into a website or app

![architecture-standalone-gateway.png](../assets/img/architecture-standalone-gateway.png)  
  *Example: A standalone Gateway fetching approved facts from Fact Pods.*

![architecture-embedded-gateway.png](../assets/img/architecture-embedded-gateway.png)  
*Example: An embedded Gateway inside a website.*

It speaks standard protocols:
- **[MCP Protocol](https://modelcontextprotocol.io)** — sharing context securely with LLMs
- **A2A (Agent-to-Agent) Protocol** — (experimental) letting agents talk directly

It can connect to **many Fact Pods** at once, merging data from different sources — with the user’s explicit consent.

## 2. Fact Pod — *The Data Owner*

A **Fact Pod** is any service or website **that already has your data** (e.g., purchase history, profile preferences).  
It’s basically an **API endpoint** that:
- Knows how to answer fact requests
- Enforces **user consent** rules
- Speaks a standardized data format ([Schema.org](https://schema.org))

**Key features:**
- Shared over simple **HTTP**
- Describes itself via a public metadata file:

  ```
  /.well-known/openprofile.json
  ```
  
  This file tells Gateways:
    - Which version of OpenProfile it supports
    - Supported data categories (“facts”)
    - OAuth endpoints for secure authentication

- Uses **OAuth 2.0** plus extensions:
    - [RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591) — Dynamic Client Registration
    - [OIDC Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html) — Metadata discovery
- Lets users **pick what to share** and **revoke access anytime**

> If access is revoked, the Fact Pod instantly invalidates the connection — no more data flows.

## How They Work Together
In OpenProfile.ai, the User, AI Assistant (LLM), Gateway, and Fact Pod interact through two main flows:

### 1. **Connecting a Fact Pod** *(One-Time Setup)*
Before the AI can use your personal data, you first connect (“enable”) a Fact Pod.

In plain terms:
1. The user tells the AI to enable a Fact Pod (e.g., “Enable my Site.com account”).
2. The AI asks the Gateway to set up the connection.
3. The Gateway checks the Fact Pod’s OpenProfile support and registers itself using **OAuth 2.0** with [Dynamic Client Registration (RFC 7591)](https://datatracker.ietf.org/doc/html/rfc7591).
4. The user is sent to the Fact Pod’s authorization page, logs in, and chooses which types of “facts” (data categories) to share.
5. The Fact Pod gives the Gateway secure tokens for future requests — but **no data is sent yet**.
6. The AI is notified that the Fact Pod is now active.

This step only happens once per Fact Pod, unless the user revokes access.

More details: [Enable Fact Pod flow](./flows/enable-fact-pod.md)

### 2. **Using a Fact Pod** *(Data Retrieval)*
When the AI needs personalized data:
1. The user makes a request (e.g., “Recommend a body lotion for my skin”).
2. The AI asks the Gateway what Fact Pods and fact categories are available for this user.
3. The AI selects relevant categories and asks the Gateway to fetch them.
4. The Gateway uses the previously stored OAuth tokens to securely request the facts from the Fact Pod.
5. The Fact Pod returns only the facts the user authorized.
6. The AI uses these facts to generate a personalized response.

More details: [Get Facts flow](./flows/get-facts-flow.md)

## Ready-Made Integrations
To make adoption easy, there are Fact Pod plugins for common platforms:
- **[WordPress (WIP)](https://github.com/openprofile-ai/wordpress-fact-pod)**
- **Joomla** (TBD)
- **Shopify** (TBD)
- **Magento** (TBD)

These can be installed like normal plugins — no advanced coding needed.

---
