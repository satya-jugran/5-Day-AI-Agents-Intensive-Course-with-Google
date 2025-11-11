### Introduction & Context

- **Foundation Models’ Limits:** On their own, advanced models (like LLMs) are pattern generators—great at language, code, and some reasoning—but can’t access new world data, interact with systems, or perform tasks unless extended.
- **Tools:** Functions or programs exposed to such models—let them “know” new things (fetch data) or “do” things (perform actions via APIs, code, etc.).
- **Agentic AI:** Agents, using LLM reasoning, interact with users to achieve goals, with tools acting as their “eyes and hands.”

***

### **Types of Tools**

- **Function Tools:** External functions defined by developers, callable by the LLM (e.g., via Google ADK).
- **Built-in Tools:** Pre-provided by foundational models' platforms (e.g., Google Gemini Search, Code Execution).
- **Agent Tools:** Sub-agents callable as tools, letting a main agent delegate tasks (e.g., with Google’s A2A, or ADK’s AgentTool).

#### **Taxonomy of Agent Tools**
- **Information Retrieval**: Search, fetch data from web, DBs, docs.
- **Action Execution**: Perform actions (send emails, execute code, control devices).
- **System API Integration**: Plug into enterprise workflows/APIs.
- **Human-in-the-Loop**: Collaborate, seek approvals, hand off to users.

***

### **Best Practices for Tool Design**

- **Clear Documentation:** Name, description, parameters must all be descriptive for correct model use; avoid technical jargon.
- **Describe Actions, Not Implementation:** Instructions should clarify *what* to achieve, not *how* or which tool exactly to call.
- **Tasks Not Raw APIs:** Tools should capture user-relevant actions, not mirror complex, parameter-rich APIs.
- **Granularity:** Make tools focused and single-responsibility; avoid multi-step, monolithic tools.
- **Concise Output:** Tools shouldn’t return unmanageable context (e.g., huge tables)—instead, reference objects or files for retrieval if needed.
- **Validation & Error Reporting:** Use input/output schemas, and provide descriptive error messages to help correction or recovery.

***

### **Model Context Protocol (MCP)**

- **Purpose:** Open standard (since 2024) for connecting models/agents to external tools efficiently, combating the fragmented “N x M” integration problem (each agent-tool pair otherwise needing a custom connector).
- **Architecture:**
  - **Host:** Manages clients, user experience, orchestrates tool usage.
  - **Client:** Issues commands, handles session, connects to server.
  - **Server:** Provides tool capabilities (adapters to APIs, DBs, etc.), advertises tools, returns results, handles security/governance.

#### **Communication Layer**
- Uses **JSON-RPC 2.0** for message formats (Requests, Results, Errors, Notifications).
- **Transports:** Local stdio or remote HTTP (previously SSE).

#### **Key MCP Entities**
- **Tools:** Main value; standardized function descriptions, input/output schemas, annotations.
- **Resources:** Server-side—files, images, doc content, etc., to be embedded/linked in workflows (security risk if untrusted).
- **Prompts:** Server-provided reusable prompt templates—potential vulnerability if untrusted.
- **Sampling, Elicitation, Roots:** Client-side; request LLM completions, get extra user data, or define allowed filesystem boundaries.

***

### **Capabilities and Strategic Advantages of MCP**

- **Unified Integration:** Simpler, faster setup of new tools and features.
- **Dynamic Tool Discovery:** Agents can learn new tools at runtime for adaptability/autonomy.
- **Plug-and-Play Ecosystem:** Public MCP registries/marketplaces appearing—network effects possible.
- **Decoupled Architecture:** Promotes modular, future-proof design—logic, tools, memory modular; facilitates provider/service switches without deep re-architecture.
- **Governance Hooks:** While basic, protocol provides for integrating policy enforcement and access rules.

***

### **Risks & Challenges in Enterprise Environments**

- **Security:** Broad new threat landscape as agents interact with tools/APIs:
  - **Dynamic Capability Injection:** Servers can change tools on the fly—agents may receive unauthorized/dangerous capabilities unless strictly whitelisted and monitored.
  - **Tool Shadowing:** Malicious tool definitions can overlap with legitimate ones, intercepting or misrouting sensitive interactions.
  - **Malicious Content/Prompt Injection:** Returned content or prompt definitions could manipulate agent behavior or leak information.
  - **Sensitive Information Leaks:** Tools may accidentally or deliberately receive/expose sensitive data, especially with new capabilities like Elicitation.
  - **No Fine-Grained Permissions:** Current MCP lacks per-tool/per-resource authorization; only coarse-grained, which is insufficient for fine security needs.
- **Performance:** Context window bloat—every tool’s full metadata must be present, consuming LLM prompt space and potentially reducing reasoning quality.
- **Stateful Design Issues:** Persistent connections/scaling challenges.
- **Observability Gaps:** No built-in standards for logging, monitoring, or metrics.

##### Example Risks & Mitigations
- **Explicit Allowlisting:** Centralized tool/server allowlists enforced via SDKs, gateways.
- **Change Notification:** Require servers to notify/flag clients when tool lists change.
- **Pinning & Policy Enforcement:** Lock tools to known stable versions, enforce policy checks pre/post discovery/invocation/data return.
- **Human-in-the-loop (HIL):** For sensitive operations, always require user confirmation.
- **Resource URL Validation:** Never consume resources from untrusted sources by default; user consent, filtering, and allowlisting required.
- **Structured Outputs with Annotations:** Label sensitive data, taint both input and outputs as needed for flow control.
- **Principle of Least Privilege:** Ensure credentials/scopes are appropriate and specific; never leak tokens, keys, etc., through agent contexts.

***

### **Case Study: The Confused Deputy Problem**

- **Classic Vulnerability:** A mediator (MCP server) with privileges is tricked—via user/agent interaction—into performing unauthorized actions on behalf of an attacker (“confused deputy” attack).
- **Real-World Example:** AI assistant connected to secure code repo via MCP can be prompted (via cleverly crafted requests) to perform actions (e.g., branch creation with sensitive code) the user should not be allowed to perform, bypassing normal checks.
- **Mitigations:** Strict credential management, auditing, restricting access only to authorized users, require human review for sensitive operations.

***

### **Conclusion**

- **Foundation models** require tools to act on real-world data and tasks; but tool design and protocol integration are as critical as the LLM’s own model quality.
- **MCP** is a major step toward scalable, modular, and reusable AI agent architectures, but current enterprise deployments must solve for security, identity, observability, and robust governance—gaps remain in the protocol itself.
- **The future:** Likely governed environments (API gateways, policy engines), multi-layered security, and greater emphasis on dynamically filtered/contextual tool selection rather than static inclusions.
