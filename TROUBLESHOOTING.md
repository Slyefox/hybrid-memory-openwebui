# Hybrid Memory XL Tool - Troubleshooting Guide

## Installation Issues

### Tool doesn't appear after upload

**Symptom:** Uploaded the tool but it's not showing in Settings → Tools

**Solutions:**
1. **Hard refresh your browser**
   - Chrome/Firefox: `Ctrl + Shift + R` (Windows/Linux) or `Cmd + Shift + R` (Mac)
   - Clear browser cache if refresh doesn't work

2. **Restart Open WebUI completely**
   ```bash
   # If using Docker:
   docker restart open-webui
   
   # If running directly:
   systemctl restart open-webui
   # or
   sudo systemctl restart open-webui
   ```

3. **Check for JSON syntax errors**
   - Open the .json file in a text editor
   - Look for error messages at the top (if any)
   - Verify it starts with `"""` and ends with proper JSON structure

4. **Check file permissions**
   ```bash
   # Tool file should be readable
   chmod 644 Hybrid_Memory_XL.json
   ```

5. **Check Open WebUI logs**
   ```bash
   # Docker:
   docker logs open-webui | grep -i error
   
   # System service:
   journalctl -u open-webui -n 50
   ```

---

### Tool appears but won't run

**Symptom:** Tool shows in list but AI says "tool not found" or tool calls fail

**Solutions:**
1. **Restart the chat**
   - Tools are loaded when chat starts
   - Start a new conversation after tool changes

2. **Check tool is enabled**
   - Settings → Tools → Find "Hybrid Memory"
   - Toggle should be ON/green
   - If OFF, click to enable

3. **Verify Python dependencies**
   Most dependencies are built-in, but check:
   ```bash
   pip list | grep -E "aiohttp|pydantic"
   ```
   If missing:
   ```bash
   pip install aiohttp pydantic
   ```

4. **Check Open WebUI version**
   - Need v0.6.31 or higher
   - Update if you're on older version
   - Tool tested on v0.6.34

5. **Look for Python exceptions**
   Enable DEBUG valve and check logs for full stack traces

---

## Memory Storage Issues

### Memories not being stored automatically

**Symptom:** Chat with AI but no memories are created

**Cause:** This is the automatic filter's job, not the tool itself.

**Solutions:**
1. **Use manual storage as workaround**
   Tell AI: "Store in memory: [your fact]"
   This uses `add_specific_memory` directly

2. **Check if companion filter is installed**
   - Settings → Filters → Look for "Hybrid Memory Filter"
   - If not there, tool still works but only with manual storage

3. **Verify memory storage is working at all**
   Test with: "Add to memory: test message 12345"
   Then: "What do you remember about test message?"
   
4. **Some models need explicit instructions**
   - GPT-4, Claude work automatically
   - Smaller local models may need prompting
   - Try: "Please remember that I prefer dark mode"

---

### add_specific_memory fails

**Symptom:** Error when trying to store memory

**Error Messages & Solutions:**

**"User context required"**
- This shouldn't happen in Open WebUI
- Restart chat or check Open WebUI installation

**"Database insert returned None"**
- ChromaDB issue
- Check Open WebUI database is running:
  ```bash
  # Check if ChromaDB is accessible
  ls -la /path/to/open-webui/backend/data/
  ```
- Restart Open WebUI
- Check disk space: `df -h`

**"Memory content truncated at 500 chars"**
- This is intentional for regular memories
- Use Story Manager for longer content
- 500 char limit prevents context bloat

---

## Memory Recall Issues

### get_relevant_memories returns nothing

**Symptom:** "No memories found" when you know memories exist

**Solutions:**

1. **Check relevance threshold**
   - `memories_dist_max` valve might be too strict
   - Try increasing from 0.75 to 0.85
   - Settings → Tools → Hybrid Memory → Edit valves

2. **Query might be too vague**
   - Instead of: "my stuff"
   - Try: "my preferences about food"
   - More specific = better results

3. **Verify memories actually exist**
   ```
   Ask AI: "Run memory_health_check()"
   ```
   Should show total memory count

4. **Try search_memories_with_context instead**
   ```
   "Search for 'food' in memories with 100 words context"
   ```
   This uses simple string matching, more reliable

5. **Enable DEBUG logging**
   - Set DEBUG valve to `true`
   - Check logs for distance scores
   - Adjust `memories_dist_max` based on scores

---

### Memories returned are irrelevant

**Symptom:** Recall brings up wrong memories, not related to query

**Solutions:**

1. **Lower relevance threshold**
   - Decrease `memories_dist_max` from 0.75 to 0.65 or 0.6
   - Stricter threshold = fewer but more relevant results

2. **Use more specific queries**
   - Bad: "stuff"
   - Better: "medical appointments"
   - Best: "cardiology appointment schedule"

3. **Clean up duplicate/bad memories**
   ```
   "Find duplicate memories and show me what you found"
   deduplicate_memories(dry_run=True)
   
   # If looks good:
   deduplicate_memories(dry_run=False)
   ```

4. **Check for memory pollution**
   ```
   "Analyze my memory patterns"
   analyze_memory_patterns(time_range="all")
   ```
   Look for lots of "uncategorized" or "conversations" - might indicate noise

5. **Consolidate old related memories**
   ```
   "Consolidate my memories about [topic] from the last month"
   consolidate_memories(topic="topic", max_age_days=30)
   ```

---

### search_memories_with_context shows truncated results

**Symptom:** Context is cut off mid-sentence

**This is by design** for safety, but adjustable:

**Solutions:**
1. **Increase context_words parameter**
   ```
   search_memories_with_context(query="...", context_words=200)
   ```
   Max is 2000 characters

2. **Results still limited?**
   - Check `max_total_chars` (hardcoded to 10,000)
   - If hit this limit, search returns fewer results
   - Try more specific query to get fewer matches

3. **Warning about "Response size limit reached"**
   - This is the max_total_chars safety limit
   - Totally normal for broad queries
   - Results returned are the most relevant ones

---

## Story Manager Issues

### Chapter numbers are wrong or skipped

**Symptom:** Saved chapter 1, 2, 3 but get_full_story shows 1, 2, 4

**Solutions:**

1. **Check actual chapter numbers**
   ```
   story_manager(action="get_summary", story_name="YourStory")
   ```
   This shows all chapters with numbers

2. **Manual chapter numbers can cause gaps**
   - Auto-increment is usually better
   - Only specify `chapter_number` when needed

3. **Fix by updating**
   ```
   # If chapter 4 should be chapter 3:
   story_manager(action="update_chapter", story_name="YourStory", 
                 chapter_number=4, content="[the content]")
   # Then manually change the chapter tag if needed
   ```

4. **Start fresh if totally broken**
   ```
   # Export first:
   story_manager(action="export_for_physician", story_name="YourStory")
   # Then delete:
   story_manager(action="delete_story", story_name="YourStory")
   # Re-create with correct structure
   ```

---

### update_chapter loses metadata

**Symptom:** After update, lab_date or other metadata is gone

**This should NOT happen in the current version**

**Solutions:**

1. **Verify you have the latest version**
   - The current release includes metadata preservation fix

2. **If still happening, workaround:**
   ```
   # Manually include metadata in update:
   story_manager(action="update_chapter", 
                 story_name="...", 
                 chapter_number=X,
                 content="...",
                 metadata={"lab_date": "2025-01-15", "lab_type": "CBC"})
   ```

3. **Check if metadata was there originally**
   ```
   story_manager(action="get_full_story", story_name="...")
   ```
   Look at the chapter - does it have "metadata" field?

---

### Can't find story that definitely exists

**Symptom:** get_full_story returns "No chapters found"

**Solutions:**

1. **Verify exact story name**
   - Story names are case-sensitive
   - "MyStory" ≠ "mystory" ≠ "my_story"
   - Use `get_summary` to see what's actually stored

2. **Check for spaces in name**
   - Spaces can cause issues
   - Prefer: `My_Story` not `My Story`

3. **List all stories**
   ```
   # Use memory_health_check to see all stored data
   memory_health_check()
   ```
   Look for `STORY_CHAPTER:` entries

4. **Story might be in memories but corrupted**
   ```
   # Search for it:
   search_memories_with_context(query="STORY_CHAPTER:YourStory", 
                                 context_words=100)
   ```
   This will find story chapters even if formatting is broken

---

### export_for_physician format issues

**Symptom:** Export format looks wrong or has encoding issues

**Solutions:**

1. **Try different format**
   ```
   # Markdown (default):
   story_manager(action="export_for_physician", 
                 story_name="...",
                 metadata={"format": "markdown"})
   
   # Plain text:
   metadata={"format": "text"}
   
   # JSON:
   metadata={"format": "json"}
   ```

2. **Encoding issues (special characters broken)**
   - This shouldn't happen (uses ensure_ascii=False)
   - If it does, use JSON format which is most reliable
   - Then convert outside of Open WebUI if needed

3. **Export is too large**
   - Story Manager has no size limit per se
   - But very large exports might timeout
   - Try `get_from_chapter` to export in chunks:
   ```
   story_manager(action="get_from_chapter", 
                 story_name="...",
                 retrieve_from_chapter=1)
   # Then chapters 11-20, etc.
   ```

---

## Performance Issues

### Memory operations are slow

**Symptom:** Memory queries take 5+ seconds

**Solutions:**

1. **Clear embedding cache**
   ```
   "Clear the embedding cache"
   clear_embedding_cache()
   ```
   This is safe - just clears model cache files
   Model will re-download (~90MB) on next use

2. **Check memory count**
   ```
   memory_health_check()
   ```
   If you have 10,000+ memories, operations will be slower
   
3. **Consolidate old memories**
   ```
   consolidate_memories(max_age_days=60)
   ```
   Merges old related memories, reduces total count

4. **Check system resources**
   ```bash
   # CPU usage:
   top
   
   # Disk I/O:
   iostat
   
   # Memory:
   free -h
   ```
   ChromaDB can be I/O intensive with many memories

5. **Consider limiting memory size**
   - Regularly run `deduplicate_memories()`
   - Export and archive old memories
   - Delete truly irrelevant memories

---

### Story Manager saves are slow

**Symptom:** save_chapter takes several seconds

**This is usually normal** - ChromaDB indexing takes time

**But if it's extreme (10+ seconds):**

1. **Check chapter size**
   - Large chapters (>5000 chars) take longer
   - Break into smaller chapters if possible
   - Tool warns if chapter >3000 chars

2. **Check total story size**
   - Stories with 100+ chapters will be slower
   - Consider splitting into multiple stories
   - Example: `Patient_Labs_2024` and `Patient_Labs_2025`

3. **System under load**
   - Check if Open WebUI is resource-constrained
   - Monitor during save operation
   - May need better hardware for heavy use

---

## Database Issues

### "ChromaDB error" messages

**Symptom:** Various errors mentioning ChromaDB or vector database

**Solutions:**

1. **Restart Open WebUI**
   ```bash
   docker restart open-webui
   # or
   systemctl restart open-webui
   ```

2. **Check database file permissions**
   ```bash
   ls -la /path/to/open-webui/backend/data/
   # Should be owned by Open WebUI user
   ```

3. **Check disk space**
   ```bash
   df -h
   ```
   ChromaDB needs space to grow

4. **Backup and recreate database** (last resort)
   ```bash
   # Export all memories first!
   export_memories(format="json")
   
   # Stop Open WebUI
   # Move/rename data directory
   # Restart Open WebUI (creates fresh DB)
   # Re-import memories
   ```

---

### "Memory not found" when you just created it

**Symptom:** Create memory, immediately query for it, not found

**This is a race condition** - rare but possible

**Solutions:**

1. **Wait a second and try again**
   ChromaDB indexing is usually instant but can lag

2. **If persistent, check health**
   ```
   memory_health_check()
   ```
   Should show the memory in total count

3. **Try exact ID query**
   When you create a memory, note the ID returned
   Then query by that specific ID

---

## AI Model Issues

### AI isn't using tools automatically

**Symptom:** You ask for memory operations but AI just responds, doesn't call tools

**This is normal for some models**

**Solutions:**

1. **Be more explicit**
   - Instead of: "Do you know my favorite color?"
   - Try: "Check your memory for my favorite color"
   - Or: "Use the memory tool to recall my preferences"

2. **Some models need function-calling training**
   - GPT-4, Claude: Excellent automatic tool use
   - Qwen3-30B: Good with prompting
   - Smaller local models: May need explicit instructions
   - Check your model's documentation

3. **System prompt helps**
   Add to your system prompt:
   ```
   "You have access to a memory system. Use the memory tools proactively:
   - get_relevant_memories when user asks about past discussions
   - add_specific_memory when user says to remember something
   - search_memories_with_context for detailed recall"
   ```

4. **Tool-calling may be disabled**
   - Check Open WebUI settings
   - Ensure "Enable Tools" is ON
   - Check model-specific tool settings

---

### AI calls wrong tool or wrong parameters

**Symptom:** AI tries to call tool with invalid parameters or wrong tool name

**Solutions:**

1. **Check tool documentation in chat**
   Ask AI: "What memory tools do you have access to?"
   It should list all available functions

2. **Correct tool name issues**
   - Tool functions all start with lowercase
   - Example: `get_relevant_memories` not `getRelevantMemories`
   - Story Manager action should be string: `"save_chapter"` not `save_chapter`

3. **Parameter type errors**
   Most common issues:
   ```python
   # WRONG:
   chapter_number="1"  # Should be int, not string
   
   # RIGHT:
   chapter_number=1
   
   # WRONG:
   metadata="lab_date: 2025-01-15"  # Should be dict
   
   # RIGHT:
   metadata={"lab_date": "2025-01-15"}
   ```

4. **Enable DEBUG logging**
   See exact parameters being passed
   Helps identify what's malformed

---

## Error Messages Explained

### "User context required"
**Meaning:** Tool didn't receive user authentication  
**Fix:** This shouldn't happen in normal use - restart chat

### "Database insert returned None"
**Meaning:** ChromaDB failed to store memory  
**Fix:** Restart Open WebUI, check disk space, check logs

### "Memory X not found or not owned by user"
**Meaning:** Trying to access memory that doesn't exist or belongs to different user  
**Fix:** Check memory ID is correct, verify memory exists with health_check

### "chapter_number must be a positive integer"
**Meaning:** Invalid chapter number (negative, zero, or not a number)  
**Fix:** Use positive integers: 1, 2, 3, etc.

### "Response size limit reached at X matches"
**Meaning:** Hit max_total_chars limit (10,000)  
**Fix:** This is normal - you got the most relevant results, use more specific query

### "Story not found" / "No chapters found"
**Meaning:** Story name doesn't match anything in database  
**Fix:** Check exact name (case-sensitive), verify story exists with get_summary

### "Could not delete old chapter"
**Meaning:** update_chapter saved new version but couldn't remove old  
**Fix:** You have both versions (safe), manually delete old chapter by ID

### "Metadata parsing failed"
**Meaning:** Metadata JSON is malformed  
**Fix:** Check metadata is valid JSON dict, recreate chapter with correct format

---

## Still Having Issues?

### Enable DEBUG Logging

1. Set DEBUG valve to `true`
2. Reproduce the issue
3. Check Open WebUI logs:
   ```bash
   # Docker:
   docker logs open-webui -f
   
   # System:
   journalctl -u open-webui -f
   ```
4. Look for lines with `[FUNC:HYBRID_MEMORY]`
5. This shows exact tool calls, parameters, and any errors

### Run Diagnostics

```
"Run a complete memory health check"
memory_health_check()
```

This provides:
- Total memory count
- Memory distribution
- Largest memories
- Recent activity
- Potential issues

### Get Help

1. **Check GitHub Issues**
   - Someone may have had same problem
   - Search closed issues too

2. **Ask in Open WebUI Community**
   - Discord or forums
   - Include: OWUI version, tool version, error messages

3. **Open GitHub Issue**
   For confirmed bugs, provide:
   - Open WebUI version
   - Tool version
   - Exact steps to reproduce
   - Error messages from logs
   - Output of memory_health_check()

4. **Last Resort: Fresh Start**
   ```
   # Export everything first:
   export_memories(format="json", limit=None)
   
   # For Story Manager data:
   story_manager(action="export_for_physician", story_name="...")
   # (do this for each story)
   
   # Then reinstall tool
   # Import memories back (as Knowledge Files) if desired
   ```

---

## Preventive Maintenance

Run these periodically to keep memory system healthy:

**Weekly:**
```
deduplicate_memories(dry_run=True)  # Check for duplicates
```

**Monthly:**
```
memory_health_check()  # Overall system check
consolidate_memories(max_age_days=60)  # Merge old memories
export_memories(format="json")  # Backup
```

**For Medical Data:**
```
# After each session:
story_manager(action="export_for_physician", story_name="...")

# Keep airgapped backups!
```

---

**Remember:** Most issues are configuration or usage problems, not bugs. Check the README and this guide thoroughly before assuming something is broken!
