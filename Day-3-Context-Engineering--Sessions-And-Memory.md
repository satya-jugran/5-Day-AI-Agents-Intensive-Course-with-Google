
## **Context Engineering—Key Concepts**
- **Context Engineering:** The process of dynamically assembling and managing information in an LLM’s context window to create intelligent, stateful agents. Unlike prompt engineering (static, instruction-focused), context engineering crafts the entire payload per turn using user/conversation history and external data for relevance.
- **Goal:** Ensure models get only the most relevant information needed—no more, no less.
- **Components of Context:**
  - *System Instructions:* Agent persona, capabilities, constraints
  - *Tool Definitions:* APIs, functions agent can use
  - *Few-Shot Examples:* In-context learning via demonstrations
  - *Factual Data:* Evidence for reasoning—pre-existing or query-specific
  - *Long-Term Memory:* Persisted user/topic knowledge across sessions
  - *External Knowledge:* Retrieved dynamically, e.g., via RAG (Retrieval-Augmented Generation)
  - *Tool/Sub-Agent Outputs, Artifacts, Conversation History, Scratchpad, User Query*

## **Sessions**
- **Definition:** Sessions are containers for a single conversation, holding chronological history and working memory.
- **Events:** User input, agent response, tool usage—forms the event list (like Gemini’s API structure).
- **State/Scratchpad:** Temporary, structured data relevant to a session (e.g., shopping cart contents).
- **Persistence:** For production use, sessions should live in robust databases—not just memory—to ensure continuity between user interactions.
- **Framework Variance:**
  - ADK: Explicit Session object with events and state
  - LangGraph: State object serves as session; history is mutable and can be compacted
- **Multi-Agent Systems:**
  - *Shared Unified History:* All agents append to one log (good for collaborative tasks)
  - *Separate Individual Histories:* Each agent keeps its own log, communicates final outputs only
  - *Interoperability:* Requires framework-agnostic data layer (e.g., Memory) for true collaboration

## **Production Considerations for Sessions**
- **Security & Privacy:** Strict isolation; ensure only the session owner can access their data. Redact PII before writing, comply with regulations (GDPR/CCPA).
- **Data Integrity:** Use policies like TTL (Time-to-Live) to delete old sessions; maintain strict chronological order.
- **Performance:** Fast read/write needed; optimize by compacting history before LLM usage (removing old/unneeded data).

## **Managing Long Contexts**
- **Challenges:**
  - *Context Window Limits:* API fails if history exceeds the window
  - *API Costs:* Token counts control costs
  - *Latency:* Shorter histories = faster responses
  - *Quality:* Excess tokens add noise
- **Compaction Strategies:**
  - *Keep Last N Turns*: Sliding window
  - *Token-Based Truncation*: Limit token count
  - *Recursive Summarization*: Periodic AI-generated summaries of old dialogue
- **Triggers for Compaction:**
  - Count-based (turns, tokens)
  - Time-based (inactivity)
  - Event-based (task completion)

## **Memory—Core Ideas**
- **Definition:** Memories are condensed, persistent snapshots extracted from dialogue/data, not just verbatim conversation. They are organized for future retrieval and continuous personalization.
- **Types of Memory:**
  - *Structured/Unstructured*: Facts vs. natural language
  - *Declarative/Procedural*: Knowing what vs. how
- **Organization:**
  - *Collections*: Pool of independent memories per topic/event
  - *Structured Profile*: Updated user facts
  - *Rolling Summary*: Evolving summary of interactions
- **Storage:** Usually in vector databases (semantic similarity) or knowledge graphs (entity relationships), sometimes hybrid.
- **Creation Mechanisms:**
  - *Explicit*: User commands agent to remember
  - *Implicit*: Agent infers and extracts info
  - *Internal/External*: Within agent framework, or dedicated service
- **Scope:**
  - *User-level:* Detailed, persistent profile across sessions
  - *Session-level:* Insights from one session, for compaction
  - *Application-level:* System-wide shared knowledge, sanitized for safety

## **Multimodal Memory**
- Most memories are generated from multimodal sources (text, images, audio) but stored as textual insights.
- Directly storing non-text (image/audio) in memory is complex and rare; usually, these are transcribed for text-based storage and retrieval.

## **Memory Generation, Extraction & Consolidation**
- *Extraction*: LLM distills meaningful facts/preferences/goals, filtering out conversational noise.
- *Consolidation*: Deduplication and conflict resolution—merging/updating/deleting as needed, keeping the memory corpus coherent.
- *Storage*: Persist new/updated memories for retrieval.

## **Differences: RAG vs. Memory**
| Feature            | RAG Engines                                 | Memory Managers                        |
|--------------------|---------------------------------------------|----------------------------------------|
| Primary Goal       | Inject factual, external knowledge          | Personalized, persistent agent memory  |
| Source             | Static databases/documents/APIs             | Dialogue between user and agent        |
| Isolation          | Usually shared/global knowledge             | Highly isolated per user               |
| Info Type          | Authoritative, domain-specific, static      | User-specific, context-rich, dynamic   |
| Write & Read Mode  | Batch admin, event-based/tool-invoked       | Event-based, per-turn or tool-triggered|
| Data Format        | Text chunk                                  | Text snippet/structured profile        |

**Analogy:**  
- RAG: Research librarian—finds facts in public library  
- Memory: Personal assistant—tracks user’s history, preferences

## **Memory Retrieval**
- Strategies balance relevance, recency (time), and importance (significance).
- Blended scoring often yields better results.
- Advanced methods: query rewriting, reranking, custom retriever (tradeoff: higher latency, complexity).
- *Timing*: Proactive retrieval at every turn vs. dynamic, as needed.

## **Inference with Memories**
- Placement matters: append to system instructions for persistent memory, inject into conversation for transient/episodic context.
- *System Instructions*: Stable data for all turns; risk of overfitting every response to memories.
- *Dialog Injection*: Risk of confusion if irrelevant memories are added.

## **Memory Quality & Evaluation**
- *Precision*: % of accurate memories
- *Recall*: % of relevant facts captured
- *F1*: Harmonic mean
- *Usability metrics*: Impact on agent performance and user experience

## **Resilience, Scalability, Security**
- Non-blocking asynchronous memory generation is vital—background processing to keep user experience snappy.
- Mitigate race conditions (e.g., transactional DB writes, optimistic locking).
- Use message queues for buffering, retry logic for failed LLM operations.
- Multiple region DB replication for availability; memory service presents a unified datastore.

## **Procedural Memories**
- Declarative: facts, history, user data (*what*)
- Procedural: playbooks/workflows, reasoning strategies (*how*). Procedural memory gives agents the ability to adapt in real-time via prompt injection, which is quicker than offline fine-tuning.

## **Privacy and Security**
- Redact sensitive data before storage.
- Strict ACLs (access control).
- Follow compliance (GDPR/CCPA).

## **Conclusion**
- **Context Engineering** is the discipline of assembling the right mix of session history, memory, and external knowledge for LLM context windows.
- **Sessions** manage short-term interaction; **Memories** provide persistent, cross-session personalization.
- *Security, performance, and resilience* are foundational for production systems.
- High-quality context management and memory generation elevate agents from simple chatbots to intelligent, adaptive companions.

***
