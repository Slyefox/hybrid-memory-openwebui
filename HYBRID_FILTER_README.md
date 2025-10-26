# Hybrid Memory Filter
**Version 1.0.0** | Author: Slye Fox | License: AGPL-3.0

Automatic memory storage and retrieval filter that works in the background to improve the Hybrid Memory Tool's effectiveness.

---

## What It Does

The Hybrid Memory Filter runs **automatically** in the background to:

1. **Inject relevant memories** before your AI processes each message (so it remembers context)
2. **Store important messages** after conversations (so it builds memory over time)
3. **Skip duplicate injection** when the Hybrid Memory Tool is actively being used (prevents redundancy)

**Think of it as:** An automatic assistant that feeds your AI's memory system and keeps it organized.

---

## Do You Need This?

### Yes, Install It If:
- ✅ You're using the Hybrid Memory Tool
- ✅ You want automatic memory storage (don't want to manually say "remember this")
- ✅ You want better context awareness in conversations

### No, Skip It If:
- ❌ You only want manual memory control (using the tool's functions directly)
- ❌ You prefer minimalist setups

**Note:** The Hybrid Memory Tool works fine without this filter - this just makes it **better** by automating memory operations.

---

## Installation

### Step 1: Upload the Filter
1. Open WebUI → **Settings** → **Functions** (not Tools!)
2. Click **"Upload Function"** or **"+"**
3. Select `Hybrid_Memory_Filter.json`
4. Click **Save**

### Step 2: Verify It's Working
1. Start a new chat
2. Say something like: "Remember that I prefer dark mode"
3. In a new chat, ask: "What do you know about my preferences?"
4. Your AI should recall what you said

**That's it!** The filter is now running automatically in the background.

---

## How It Works

### Automatic Memory Injection (Inlet)
**Before each message you send:**
1. Filter searches your memories for relevant context
2. Injects up to 3 most relevant memories into the conversation
3. Your AI now has context from past conversations
4. Responds with better awareness

**Example:**
- You previously said: "I'm allergic to penicillin"
- Today you ask: "What antibiotics are safe for me?"
- Filter injects the allergy memory automatically
- AI responds with that knowledge

### Automatic Memory Storage (Outlet)
**After each conversation:**
1. Filter analyzes what was said
2. Stores important user messages (preferences, facts, plans)
3. Stores useful assistant responses
4. Skips generic replies ("ok", "thanks", etc.)

**What gets stored:**
- ✅ Preferences: "I prefer dark mode"
- ✅ Facts: "My name is Alex"
- ✅ Plans: "I'm traveling to Japan next month"
- ✅ Important context: "I work as a nurse"
- ❌ Generic: "ok", "thanks", "yes"

### Smart Tool Detection
**When you explicitly use memory tools:**
- Filter detects you're accessing memory directly
- Skips automatic injection (prevents duplication)
- Stays out of the way when not needed

---

## Configuration (Optional)

Default settings work great, but you can adjust:

### In Open WebUI:
1. Settings → Functions → Hybrid Memory Filter
2. Click the gear/settings icon
3. Adjust valves:

**Available Settings:**

| Setting | Default | What It Does |
|---------|---------|--------------|
| `enabled` | true | Turn filter on/off |
| `max_memories` | 3 | How many memories to inject per message |
| `memory_length_cutoff` | 200 | Max characters per memory |
| `memories_dist_max` | 0.7 | Relevance threshold (lower = stricter) |
| `min_memory_length` | 10 | Minimum message length to store |
| `skip_on_tool_processing` | true | Skip injection when using memory tools |

### When to Adjust:

**Too many irrelevant memories showing up?**
- Lower `memories_dist_max` to 0.6 (stricter matching)

**Want more comprehensive context?**
- Increase `max_memories` to 5
- Increase `memory_length_cutoff` to 300

**Storing too much noise?**
- Increase `min_memory_length` to 20

**Want to disable temporarily?**
- Set `enabled` to false

---

## How It Works With Hybrid Memory Tool

### Together They Provide:
1. **Automatic memory** (filter does this in background)
2. **Manual control** (tool lets you search/edit/organize)
3. **Best of both worlds** (passive + active memory management)

### Example Workflow:

**Automatic (Filter handles):**
- You chat naturally about your preferences
- Filter stores: "User prefers dark mode"
- Filter stores: "User is allergic to penicillin"
- Next conversation, filter injects these automatically

**Manual (Tool handles):**
- You want to see all health memories: "Show me memories about my health"
- Tool retrieves and displays them
- You want to track lab reports: Use Story Manager
- You want to delete wrong memory: "Delete memory about old address"

### The filter makes the tool better by:
- Keeping memory database populated automatically
- Injecting context so manual searches are more effective
- Reducing need for "remember this" commands

---

## Troubleshooting

### Filter isn't storing memories
1. Check it's enabled in Settings → Functions
2. Try saying something substantial (not just "ok" or "yes")
3. Check minimum message length setting

### Too many irrelevant memories appearing
1. Lower `memories_dist_max` to 0.6 or 0.5
2. Reduce `max_memories` to 2

### Filter interfering with memory tool
1. Should NOT happen - filter detects tool use
2. If it does, set `skip_on_tool_processing` to true
3. Restart chat

### Filter seems slow
1. Normal - memory search takes 1-2 seconds
2. Reduce `max_memories` to 2 if you want faster
3. This is inherent to semantic search

---

## Compatibility

**Tested with:**
- Open WebUI v0.6.32+
- Hybrid Memory Tool v8.3.0+
- All LLM backends (Ollama, OpenAI API, etc.)

**Requirements:**
- Open WebUI v0.6.32 or higher
- Hybrid Memory Tool (recommended but not required)

---

## Privacy & Data

- All memory stored locally in Open WebUI's database
- No external transmission
- Filter runs server-side (not in your browser)
- User-scoped (your memories are private to you)

---

## Disable/Remove

### Temporarily Disable:
1. Settings → Functions → Hybrid Memory Filter
2. Set `enabled` valve to false
3. Filter stops running but memories remain

### Permanently Remove:
1. Settings → Functions
2. Find Hybrid Memory Filter
3. Click delete/trash icon
4. Memories remain in database (filter just stops running)

---

## Technical Details

**How memory injection works:**
- Filter intercepts messages before AI processes them
- Performs semantic search on your memory database
- Injects top N relevant memories as system context
- AI sees memories alongside your current message

**How memory storage works:**
- Filter intercepts messages after AI responds
- Analyzes both user and assistant messages
- Stores important content with "User:" or "Assistant:" prefix
- Strips HTML/XML tags before storage

**Smart detection:**
- Checks if last message is from hybrid memory tool
- If yes, skips injection to prevent duplicating tool results
- If no, proceeds normally

---

## FAQ

**Q: Do I need both the tool and filter?**  
A: No. Tool works alone (manual only). Filter works alone (automatic only). Together is best.

**Q: Will this slow down my AI?**  
A: Slightly (1-2 seconds) but worth it for context awareness.

**Q: Can I see what memories the filter is injecting?**  
A: Not directly, but use the tool's `get_relevant_memories` function to see what's stored.

**Q: Does this replace the built-in Open WebUI memory?**  
A: No, works alongside it. Uses the same database, just smarter filtering.

**Q: What if I don't want certain things remembered?**  
A: Use the tool's `delete_specific_memory` function to remove them.

---

## Version History

**1.0.0** (First Public Release)
- Smart tool detection (no duplicate injection)
- Updated for Open WebUI 0.6.32+ compatibility
- Internal task detection (skips title/tag generation)
- Improved precision (only checks last message)

---

## License

AGPL-3.0 License

Copyright (c) 2025 Slye Fox

---

## Credits

**Created by:** Slye Fox  
**Works with:** Hybrid Memory Tool  
**For:** Open WebUI Community

---

**Questions?** See the Hybrid Memory Tool README for comprehensive memory management documentation.
