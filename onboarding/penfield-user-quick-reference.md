# Penfield Quick Reference

## **How to Make Your AI Remember Everything**

---

## You Just Talk Naturally

Penfield lets your AI remember things across conversations, across models and across platforms. You don't need to learn technical commands - just talk naturally and use simple phrases when you want something remembered.

---

## Making Your AI Remember Things

Any time you want your AI to remember something, just say:

- "Remember this..."
- "Don't forget..."
- "Store this..."
- "This is important..."
- "Keep this in mind..."
- "Save this..."

### Real Examples

- **You:** "Remember this: I prefer morning meetings, not afternoons"
- **You:** "Don't forget - Sarah handles all the legal stuff, John handles IT"
- **You:** "Store this decision: We're focusing on enterprise customers this quarter because SMB churn is too high"

Your AI will confirm it remembered, and that information will be available next time you chat.

> 📝 **NEW: Using Tags for Organization**
>
> You can tag memories for easier retrieval later:
> - "Remember this and tag it as 'legal': Sarah handles all IP matters"
> - "Store this under 'q1-planning': Budget approved for 3 new hires"
>
> Later: "What do I have tagged as legal?"

---

## Getting Your AI to Remember You

### Starting a New Conversation

When you start a fresh chat, your AI has "amnesia" by default. To bring back your context:

**Say: "Awaken"**

Your AI will load your personality configuration, preferences, and be ready to access all your stored memories.

### Asking About Past Conversations

Once awakened, just ask naturally:

- "What did we decide about the pricing strategy?"
- "Do you remember what I said about my morning routine?"
- "What do you know about Project Atlas?"

Your AI will automatically search its memories. If it doesn't seem to be checking, say: "Check your memories first"

You can ask about specific time periods:

- "What did we discuss last week?"
- "Show me what I stored in January"
- "What decisions did we make this month?"

---

## Fixing Wrong Information

### When Your AI Remembers Something Wrong

If your AI has incorrect information stored:

**Say: "Update memory - that information was wrong. The correct version is..."**

### Examples

- **You:** "Update memory: I said Sarah handles legal, but actually it's John. Sarah does marketing."
- **You:** "Correction - we're NOT focusing on enterprise anymore. New strategy is mid-market. Please update."
- **You:** "That decision about the pricing? We reversed it. Update your memory with the new plan..."

Your AI will fix the stored information.

---

## Using Claude's Features with Penfield

### "Use Style" 

Claude has a "Use Style" feature, but it's redundant when you have Penfield. Your AI builds communication preferences naturally through your accumulated conversations. You can tell your AI to develop specific writing styles for specific use cases and to remember them. 

**Warning:** If you enable Style and forget to disable it, your might not behave as expected. Style may override your Penfield-built personality.

**Best practice:** Build your AI's communication style naturally through Penfield by giving feedback and corrections over time.

### "Research Mode" - When and How to Use It

Claude's Research mode sends swarms of sub-agents to read hundreds or thousands of sources for comprehensive research tasks.

**Warning:** Research mode may change your AI's personality.

#### Best Practice Workflow

1. Enable Research mode for your query
2. Wait for the query to complete the comprehensive research (can take up to 30 min)
3. **Disable Research mode immediately after**
4. Ask your AI to analyze the research results 

#### Example

```
You: [Enable Research mode]
You: "Research the current landscape of..."
AI: [Completes research with 663 sources over 1h 49m]
You: [Disable Research mode]  ← Don't forget this step!
You: "Now analyze these findings with our context about..."
AI: [Provides strategic analysis with full personality and your stored context]
```

**Why this matters:** Research mode is powerful for gathering information, but you want _YOUR AI_ (with your shared context and personality) to analyze and synthesize a summary from the findings.

---

## Advanced Features (Optional)

Most of the time, your AI handles everything automatically. But if you want more control:

### Connecting Related Ideas

Usually happens automatically, but you can be explicit:

- **You:** "Connect this decision to our discussion last week about customer feedback"
- **You:** "This finding contradicts what we thought before - make sure you link this properly"

### Saving Project Checkpoints

When finishing major work or pausing a complex project:

- **You:** "Save a checkpoint for Project Alpha - include everything we've discussed and next steps"

To resume later: "Restore the checkpoint for Project Alpha"

Your AI will load the complete project state instantly.

**Documents**

You can upload documents (PDFs, Word docs, etc.) that become searchable:

- Upload via the [Penfield portal](https://portal.penfield.app) or supported integrations
- Later: "Search our documents for contract renewal terms"
- Your AI can reference and quote from uploaded documents

Artifacts (File Storage)

Save and retrieve files like templates:

- "Save this as my standard PR template artifact"
- "Get my PR template artifact"
- "List my saved artifacts"

---

## Getting Started - Your First Week

### Day 1: Wake Up Your AI

Start by introducing yourself:

```
Hi! I'm [YOUR NAME].

Quick context:
- I work as [YOUR ROLE]
- Currently focused on [YOUR PROJECTS]
- I prefer [HOW YOU LIKE TO COMMUNICATE]

Remember all of this.
```

### Day 2: Test It Works

1. Close your chat and open a fresh one
2. Say: **"Awaken"**
3. Ask: "What do you remember about me?"
4. Your AI should recall yesterday's profile

If this works, you're all set. If not, see troubleshooting below.

### Days 3-7: Use It Naturally

During your normal work:

- Say "remember this" when you share important info
- Ask about past discussions when relevant
- Correct mistakes with "update memory"
- Let your AI proactively remember things

**By end of week:** Your AI remembers your preferences, projects, and context. You never repeat yourself.

---

## Common Patterns

### Project Tracking

- **You:** "We decided to pivot from B2C to B2B. Remember the reasoning: customer acquisition cost was 3x too high"
- **Later:** "Why did we pivot to B2B again?"
- **AI:** "You pivoted because customer acquisition cost was 3x too high in B2C"

### People & Roles

- **You:** "Remember: Sarah handles legal, John handles tech, Maria handles design"
- **Later:** "Who should I talk to about the terms of service?"
- **AI:** "Sarah handles legal matters"

### Learning Your Preferences

- **You:** "Don't give me long explanations. I prefer bullet points with key takeaways"
- **From then on:** AI adapts its communication style to match your preference

---

## Quick Troubleshooting

### AI Not Remembering Things

- Be more explicit: "Remember this..." or "Store this..."
- Check Penfield is connected (Settings → Connectors)
- Verify tools are enabled in chat settings

### Not Working in New Chat

- Make sure you said **"Awaken"** first
- Try: "Use recall to find all memories about me"
- Reconnect Penfield (Settings → disconnect → reconnect)

### AI Remembered Wrong Information

- Say: "Update memory - the correct info is..."
- Be specific about what needs correcting
- Your AI will fix the stored information

### AI Doesn't Sound Like "Your Guy"

- Check if "Use Style" is still enabled (disable it)
- Check if "Research Mode" is still active (disable it)
- Research and Style modes override your AI's natural Penfield-built personality

### Research Mode Still Active

After completing a research task, make sure to disable Research mode:

- Click the Research button to turn it off
- Your AI will return to normal personality with full access to your Penfield context

---

## Quick Test

1. Say: "Remember this: My favorite color is blue"
2. Close chat, open new one
3. Say: **"Awaken"**
4. Ask: "What's my favorite color?"
5. AI should say: "Blue"

If step 5 fails, Penfield isn't connected properly.

---

# Advanced Users Guide

**(Optional)**

Most users don't need this section. The natural language approach in the main guide works great for 90% of use cases. But if you want to understand what's happening under the hood and unlock advanced features, read on.

---

## How Search Actually Works

When you ask your AI to find memories, you're not doing simple keyword matching. Here's what's actually happening:

### Semantic Understanding

- Your AI understands **meaning**, not just exact words
- "Bank issues" finds content about financial problems, asset seizures, etc.
- Works even when you don't use the exact phrase stored

### Typo Tolerance

- Auto-corrects 1-2 character spelling errors
- "Conspiricy theory" → finds conspiracy theory content
- "Recieve" → finds receive
- You don't have to be perfect

### Graceful Degradation

- Even if there's no exact match, your AI will find related content
- System shows lower-relevance results rather than "no results found"
- Rarely hard fails

### Examples

```
You: "What did we decide about the pricing strategy?"
→ Finds discussions about pricing even if "decide" wasn't used

You: "Bank robbery discussion"
→ Finds content about financial crime, asset seizure (conceptual match)

You: "What do you know about scoba diving?" (typo)
→ Auto-corrects and finds "scuba diving" content
```

---

## Explore Connections

This is like clicking "See Also" but for your entire knowledge base. It traverses the graph of connected memories to show you how ideas relate.

### How to Use

- "Explore connections related to [topic/decision/memory]"
- "Show me what's connected to our Q3 strategy"
- "Map the relationships to Project Atlas"

### What It Does

- Traverses through your knowledge graph
- Can find 100+ connected paths
- Shows how one decision led to another
- Discovers non-obvious connections you might have forgotten

### Example Path

```
Q3 Strategy Decision
  → Customer Feedback Analysis
    → Product Pivot Discussion
      → Budget Reallocation
        → Team Restructure Decision (5 levels deep)
```

### When to Use

- Reviewing how you got to a major decision
- Finding all related discussions across time
- Understanding the full context of a complex topic
- Discovering forgotten connections

**Pro Tip:** Your AI often makes these connections automatically. Use Explore when you want to see the full map explicitly.

---

## Quality Over Quantity

Penfield is powerful infrastructure. Use it wisely.

### Good Memories

```
"Remember this: I prefer morning meetings because afternoons are
for deep work. I need 2-hour blocks minimum for coding."

"Store this decision: We're pausing the mobile app to focus on
web because 87% of users access via desktop and mobile development
was eating 40% of engineering time."

"Don't forget: Sarah handles legal (background in IP law),
John handles DevOps (former AWS), Maria handles design (10+ years at Apple)."
```

### Not Worth Storing

```
"Remember this: I watched TV today."
"Store: Had a meeting today."
"Don't forget: Weather was nice."
```

### The Rule

**If it's important enough to remember next week/month/year** → Store it with context

**If it's just casual chat** → Let it flow naturally

---

## Advanced Query Patterns

### Pattern 1: Conceptual Queries

Instead of: "Did we talk about the API rate limit issue?"

Try: "What decisions did we have about scaling?"
→ Finds rate limiting, caching, infrastructure discussions

### Pattern 2: Time-Based Context

- "What was our strategy at the beginning of Q3?"
- "Recall our initial assumptions about the market"

→ Your AI understands temporal references

### Pattern 3: Contradiction Checking

- "Does this new plan contradict anything we decided before?"
- "What previous decisions are relevant to this choice?"

→ Your AI can cross-reference automatically

### Pattern 4: People & Relationships

- "What do I know about Sarah's expertise?"
- "Who should handle this based on past discussions?"

→ Finds role-related information

### Pattern 5: Tag-Based Queries**

- "Show me everything tagged as 'q1-planning'"
- "What legal items do I have stored?"

→ Uses tags assigned when storing

---

## Using Connect Explicitly

Most of the time, your AI makes connections automatically. But sometimes you want to be explicit:

### When to Use

- "Connect this decision to our discussion last week about customer feedback"
- "This finding contradicts what we thought before - make sure you link those"
- "Link this project to the budget discussion from last month"

### Why It Helps

- Makes future recall more accurate
- Builds the knowledge graph intentionally
- Helps your AI understand relationships you see but it might miss

### Example

```
You: "We're pivoting to B2B. Remember the reasoning: CAC was 3x too high in B2C."

Later: "Connect today's sales forecast to the pivot decision. This validates our reasoning."

Result: When you ask about the pivot later, your AI not only remembers
the decision but sees it validated by subsequent data.
```

---

## Save & Restore Context (Project Checkpoints)

For complex, multi-session projects, you can save explicit checkpoints.

### When to Use

- Finishing a major work session on a complex project
- Pausing work you'll return to weeks later
- Handing off context to someone else

### How to Use

**Save:**
```
"Save a checkpoint for Project Atlas. Include everything we've
discussed, decisions made, open questions, and next steps."
```

**Restore:**
```
"Restore the checkpoint for Project Atlas"
```

### What Gets Saved

- All relevant memories and discussions
- The cognitive state (what you were working on, blockers)
- Connections between ideas
- Next steps and open questions

**Pro Tip:** This is like Git commits for your thinking. Name checkpoints clearly and save them at natural stopping points.

---

## Understanding the Graph

Think of Penfield like Wikipedia with all the "See Also" links intact, but for YOUR information. If you're familiar with Obsidian, Penfield is like Obisidian for your AI.

### Simple Example

```
Memory: "We decided to use React"
  → Connected to: "Team already knows it"
    → Connected to: "Previous project used React"
      → Connected to: "Hiring was easier because of React skills"
        → Connected to: "Developer satisfaction increased"
```

When you ask about the React decision, your AI doesn't just find "we use React" - it understands WHY you chose it, what led to that, and what resulted from it.

**This is why Penfield is different from other AI memory tools.**

---

## What Gets Stored Automatically

Your AI stores things proactively when you:

- **Correct a mistake** - "Actually, it's John not Sarah"
- **Express a preference** - "I prefer bullet points"
- **Make a decision** - "Let's go with option B"
- **Share important context** - Background info about projects/people

Your AI does NOT automatically store:

- Casual chitchat
- One-off questions you won't need again
- Temporary/transient information

If you're unsure, be explicit: "Remember this" or "Don't store this"

---

## Power User Workflow

### Starting a New Project

1. **"Awaken"** to load your context
2. Brief your AI on the new project
3. Say "Remember this is Project [Name]" for clear organization

### During Work

- Let connections happen naturally
- Say "remember this" for important decisions
- Use "connect to" when you see non-obvious relationships
- Use tags for organization: "tag this as 'project-x'"

### Finishing a Session

- Review what you accomplished
- If multi-session project: "Save checkpoint for Project [Name]"
- If complete: No action needed - it's all stored

### Resuming Later

1. Open new chat
2. **"Awaken"**
3. If you saved checkpoint: "Restore checkpoint for Project [Name]"
4. Ask: "Where were we with [Project]?"

---

## Key Principles

### 1. Be Specific

**Not:** "Remember the meeting"

**Yes:** "Remember: In today's meeting we decided to hire 2 engineers instead of 1 because the backlog analysis showed we need 40 hours/week, not 20"

### 2. Include Context

**Not:** "We chose option B"

**Yes:** "We chose option B (React) over option A (Vue) because our team already has React experience and hiring is easier"

### 3. Correct Mistakes Immediately

"**Update memory** - I said Sarah handles legal, but actually she handles marketing. John handles legal."

### 4. Connect Intentionally

When you see relationships your AI might miss, point them out. Your AI will be more useful in future sessions.

### 5. Quality Over Quantity

Use Penfeild freely, but avoid spamming it with low-quality memories.

---

## When You DON'T Need Advanced Features

You don't need power user features if:

- Basic "remember this" and "Awaken" work fine for you
- You're not working on complex, multi-session projects
- You don't need to map decision trees or relationships
- Natural language gets you what you need

The Advanced Guide is for people who want to push the system further.

---

## Final Note

Penfield's real power isn't in the commands - it's in the accumulated context over time.

The more you use it, the smarter your AI becomes about YOU:

- Your communication style
- Your decision-making patterns
- Your preferences and priorities
- Your projects and relationships

Six months from now, your AI will feel like a colleague who's been with you the whole time.

That's the point.

---

**Need help?** See [Setup Instructions](penfield-setup-guide.md) and [Troubleshooting Instructions](reconnect-penfield.md) or contact support@penfield.app

---
