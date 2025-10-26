# Hybrid Memory XL Tool - Quick Installation Guide

## 5-Minute Setup

### Step 1: Upload the Tool (2 minutes)

**Method A: Via Open WebUI Interface (Easiest)**
1. Log into Open WebUI as admin
2. Go to **Settings** → **Tools**
3. Click **Upload Tool** or **Import**
4. Select `Hybrid_Memory_XL.json`
5. Click **Save**
6. Tool should appear in your tools list

**Method B: Manual File Placement**
1. Locate your Open WebUI installation directory
2. Find the tools folder (usually `open-webui/backend/tools/`)
3. Copy `Hybrid_Memory_XL.json` into that folder
4. Restart Open WebUI
5. Tool should auto-load on startup

### Step 2: Verify Installation (1 minute)

Start a new chat and ask your AI:
```
"Store in memory: testing the hybrid memory tool"
```

Your AI should respond with confirmation that it stored the memory. Check the tool invocation in the response - you should see `hybrid_memory_xl/add_specific_memory` was called.

Then ask:
```
"What do you remember about testing?"
```

Your AI should recall the memory you just stored.

✅ **If both work, you're done!** Skip to "Next Steps" below.

❌ **If something failed, see Troubleshooting section.**

### Step 3: Optional - Configure Valves (2 minutes)

Only needed if you want to customize behavior:

1. In Open WebUI, go to **Settings** → **Tools**
2. Find "Hybrid Memory" in the list
3. Click the settings/gear icon
4. Adjust valves:
   - `max_memories`: Number of results (5 is good default)
   - `memory_length_cutoff`: Max chars per result (200 is good)
   - `memories_dist_max`: Relevance threshold (0.75 is good)
5. Click **Save**

**When to adjust:**
- Getting too many irrelevant results? → Lower `memories_dist_max` to 0.65
- Need more comprehensive recall? → Increase `max_memories` to 10
- Need fuller memory content? → Increase `memory_length_cutoff` to 500

---

## Testing Story Manager

If you plan to use Story Manager (for medical records, creative writing, etc.), test it:

```
"Save a test chapter as chapter 1 of MyTest with some content"
```

Tool call should show:
```
story_manager(action="save_chapter", story_name="MyTest", content="...")
```

Then retrieve it:
```
"Get the full story for MyTest"
```

Tool call should show:
```
story_manager(action="get_full_story", story_name="MyTest")
```

✅ **Working?** You're ready to use Story Manager for real data.

Clean up the test:
```
"Delete the story MyTest"
```

---

## Next Steps

### For Personal/General Use
1. Just start chatting naturally - memory will work automatically
2. Try asking about past conversations: "What did we discuss yesterday?"
3. Manually store important facts: "Remember that I'm a vegetarian"

### For Medical Record Management
1. Read the **Story Manager** section in the README
2. Create a naming convention (e.g., `PatientInitials_Labs`, `PatientInitials_Visits`)
3. Always include metadata with dates and lab types
4. **Keep airgapped backups** - don't rely solely on memory

### For Creative Writing
1. Start a story with `story_manager(action="save_chapter")`
2. Continue by retrieving full story before writing new chapters
3. Use `get_summary` to track your chapters
4. Export completed works with `export_for_physician` action

---

## Troubleshooting Quick Fixes

### Tool doesn't appear after upload
**Try:**
1. Refresh your browser (hard refresh: Ctrl+Shift+R)
2. Restart Open WebUI completely
3. Check admin panel for error messages
4. Verify file is valid JSON (open in text editor - should be proper JSON format)

### "Function not found" error
**Try:**
1. Verify tool is enabled in Settings → Tools
2. Check that you're using the right function names
3. Restart the chat (tools load per-chat)
4. Check Open WebUI logs for Python errors

### AI doesn't use the tool automatically
**This is normal!** The AI decides when to use tools. You can:
1. Be more explicit: "Store this in your memory: ..."
2. Ask directly: "Use the memory tool to remember that..."
3. Some models need prompting to use tools effectively

### Memory recall is poor
**Try:**
1. Lower `memories_dist_max` valve (from 0.75 to 0.65)
2. Use more specific queries
3. Try `search_memories_with_context` for better results
4. Run `deduplicate_memories()` to clean up duplicates

### Story Manager issues
**Try:**
1. Verify story name doesn't have spaces (use `_` instead)
2. Check chapter numbers with `get_summary`
3. Ensure story exists before trying to update/retrieve
4. Use `story_manager(action="get_summary", story_name="...")` to debug

---

## Common Questions

**Q: Do I need the companion filter?**
A: No, the tool works standalone. The filter just improves automatic memory storage quality.

**Q: Will this conflict with built-in memory?**
A: No, they coexist. This tool adds capabilities, doesn't replace built-in system.

**Q: Can I use this with multiple AI models?**
A: Yes! Works with any Open WebUI backend (Ollama, OpenAI, etc.)

**Q: How much memory does this use?**
A: Negligible - stores data in existing ChromaDB, no extra database needed.

**Q: Is my data private?**
A: Yes, stored locally in your Open WebUI database. Never sent elsewhere.

**Q: Can I migrate my old memories?**
A: Old memories remain accessible. This tool adds new capabilities without disrupting existing data.

---

## Getting Help

1. **Check the full README** - Most questions answered there
2. **Enable DEBUG logging** - Set valve to `true` and check logs
3. **Run health check** - Ask AI to run `memory_health_check()`
4. **Check Open WebUI forums** - Community is helpful
5. **Open GitHub issue** - For confirmed bugs

---

## Uninstalling

If you need to remove the tool:

1. Go to Settings → Tools
2. Find "Hybrid Memory" in the list
3. Click delete/trash icon
4. Confirm deletion

**Note:** Your stored memories remain in the database. This only removes the tool, not the data.

To also clear memories:
1. Before uninstalling, export them: `export_memories()`
2. Then manually delete from ChromaDB if desired
3. Or keep them - they won't hurt anything

---

## Success Checklist

✅ Tool uploaded and appears in tool list  
✅ Test memory storage works  
✅ Test memory recall works  
✅ Valves configured (if desired)  
✅ Story Manager tested (if needed)  
✅ Read the appropriate sections of README for your use case  

**You're ready to use Hybrid Memory!**

---

*Installation issues? See TROUBLESHOOTING.md for detailed solutions.*
*Advanced usage? See HYBRID_MEMORY_README.md for complete documentation.*
