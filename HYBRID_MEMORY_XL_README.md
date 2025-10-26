# Hybrid Memory XL tool for Open WebUI
**Author: Slye Fox | License: AGPL-3.0**

A powerful, production-ready autonomous memory management system for Open WebUI that significantly improves upon the built-in memory functionality through better relevance filtering, context tracking, and specialized tools for managing different types of data. Works with the built-in memory filter of Openweb UI v0.6.31+ but works best with the Hybrid Memory Filter.

##CRITICAL | PLEASE NOTE:##

The Hybrid Memory XL tool is designed to be used by the AI/ LLM with complete autonomy. Therefore, regular system memory backups are highly recommended (The AI/ LLM does not need permission to call the tool if the tool is available/ active and system prompt tool-call syntax is in place. Depending on your AI model, autonomy may need to be encouraged or discouraged in your system prompt. You may also choose to omit the tool call syntax from the system prompt entirely and provide calls to the AI on a case-by-case basis). Hybrid Memory XL contains multiple customizeable tools, filters and actions (described below) to facilitate AI memory. Customization is not required for functionality but is recommended for your use case (see below). Also, as Openweb UI uses the CPU for embeddings natively, the Hybrid Memory tool can become HIGHLY demanding on system resources depending on your settings. Hybrid Memory XL has been tested on Windows 10 with 64GB RAM/ 16GB VRAM, Windows 11 with 64GB RAM/ 32GB VRAM and OpenWebUI v0.6.31+. Recommend i7 13th Gen processor or newer (liquid cooling preferred) in combination with at least 32GB RAM and 24GB VRAM. DO NOT operate the Hybrid Memory tool without a system resource monitor (i.e. HWMonitor)!

---

## Key Features

### 🎯 Core Memory Functions
- **Semantic Search with Filtering** - Better relevance scoring and distance thresholds
- **Context-Aware Search** - Get memories with surrounding conversation for better understanding
- **Manual Memory Control** - Add, delete, and manage specific memories
- **Smart Relevance** - Configurable distance thresholds to filter noise

### 📚 Story Manager (NEW!)
Specialized system for managing chronological data:
- Medical lab reports with metadata (dates, physicians, lab types)
- Serialized creative writing (bedtime stories, novels)
- Any data that needs chapter/version tracking
- Professional export formats (Markdown, JSON, plain text)

### 🔧 Memory Maintenance
- **Pattern Analysis** - Understand what's being remembered and when
- **Deduplication** - Find and remove duplicate memories
- **Consolidation** - Merge related old memories to reduce clutter
- **Health Check** - Comprehensive diagnostics on memory system state
- **Export** - Backup your memories in JSON format

### 🛡️ Data Safety
- **Update protection** - Saves new data before deleting old (prevents data loss)
- **Metadata preservation** - Maintains important tags and timestamps
- **Error recovery** - Graceful handling of database issues
- **Validation** - Input checking to prevent invalid operations

---

## Quick Start

### Installation

1. **Copy the tool file:**
   - Place `Hybrid_Memory_XL.json` in your Open WebUI tools directory
   - Or upload via the Open WebUI admin interface (Settings → Tools → Upload)

2. **Optional but Recommended - Install the companion filter:**
   - The Hybrid Memory Filter works in the background to improve automatic memory storage
   - Without it, the tool still works but relies entirely on built-in filtering
   - Upload `Hybrid_Memory_Filter.json` if you have it

3. **Configure the valves** (optional):
   ```
   USE_MEMORY: true (default)
   DEBUG: false (set to true for troubleshooting)
   max_memories: 5 (results returned by get_relevant_memories)
   memory_length_cutoff: 200 (max characters per memory in results)
   memories_dist_max: 0.75 (relevance threshold, lower = stricter)
   ```

4. **Test it:**
   Ask your AI: "Store in memory: I prefer dark mode"
   Then ask: "What do you remember about my preferences?"

---

## Basic Usage Examples

### Storing Information
```
"Remember that I'm allergic to penicillin"
"Store this in memory: My favorite color is blue"
"Add to memory: I work at Tesla as a software engineer"
```

Your AI will use `add_specific_memory(memory_content="...")` automatically.

### Recalling Information
```
"What do you remember about my allergies?"
"Do you have any memories about my job?"
"What have I told you about my preferences?"
```

Your AI will use `get_relevant_memories(query="...")` automatically.

### Finding Memories with Context
```
"Search for 'coffee' in your memories with 100 words of context"
"Find all memories about my medical conditions and show surrounding conversation"
```

Your AI will use `search_memories_with_context(query="...", context_words=100)`.

### Deleting Incorrect Memories
```
"Delete the memory about my favorite color"
"Remove all memories containing 'old address'"
```

Your AI will use `delete_specific_memory(search_content="...")`.

---

## Advanced: Story Manager

The Story Manager is perfect for any chronological data that needs to be tracked as chapters or segments.

### Medical Records Example

**Storing a lab report:**
```
"Save this CBC lab report as chapter 1 of PatientName_Labs with metadata"

Tool call:
story_manager(
  action="save_chapter",
  story_name="PatientName_Labs",
  content="CBC Results: WBC 7.2, RBC 4.5, Hemoglobin 14.2...",
  metadata={"lab_date": "2025-10-24", "lab_type": "CBC", "physician": "Dr. Smith"}
)
```

**Updating a report (with data safety):**
```
"Update chapter 3 with corrected creatinine value"

Tool call:
story_manager(
  action="update_chapter",
  story_name="PatientName_Labs",
  chapter_number=3,
  content="Corrected: Creatinine 1.2 (was 1.5)..."
)
```
Note: Metadata is automatically preserved during updates.

**Retrieving all reports:**
```
"Get the full history of PatientName_Labs"

Tool call:
story_manager(action="get_full_story", story_name="PatientName_Labs")
```

**Searching for specific values:**
```
"Find all lab reports with elevated creatinine"

Tool call:
story_manager(
  action="search_story",
  story_name="PatientName_Labs",
  content="creatinine"
)
```

**Exporting for physician:**
```
"Export PatientName_Labs as a formatted report"

Tool call:
story_manager(
  action="export_for_physician",
  story_name="PatientName_Labs",
  metadata={"format": "markdown"}  # or "text" or "json"
)
```

### Creative Writing Example

**Starting a story:**
```
"Save chapter 1 of my novel 'Quantum Dreams'"

Tool call:
story_manager(
  action="save_chapter",
  story_name="Quantum_Dreams",
  content="The laboratory hummed with anticipation as Dr. Chen prepared..."
)
```

**Continuing the story:**
```
"Write chapter 2 of Quantum Dreams"

AI will:
1. Call story_manager(action="get_full_story", story_name="Quantum_Dreams")
2. Read what exists
3. Generate new chapter
4. Call story_manager(action="save_chapter", ...) with new content
```

**Getting a summary:**
```
"Show me the outline of Quantum Dreams"

Tool call:
story_manager(action="get_summary", story_name="Quantum_Dreams")
```

---

## Story Manager Actions Reference

| Action | Purpose | Required Parameters | Optional Parameters |
|--------|---------|---------------------|---------------------|
| `save_chapter` | Add new chapter/segment | `story_name`, `content` | `chapter_number`, `metadata` |
| `update_chapter` | Modify existing chapter | `story_name`, `chapter_number`, `content` | `metadata` |
| `get_full_story` | Retrieve entire story | `story_name` | - |
| `get_from_chapter` | Get story from chapter X onwards | `story_name`, `retrieve_from_chapter` | - |
| `get_summary` | List all chapters with previews | `story_name` | - |
| `search_story` | Find content within story | `story_name`, `content` (query) | - |
| `export_for_physician` | Generate formatted report | `story_name` | `metadata: {"format": "markdown"}` |
| `delete_story` | Remove entire story | `story_name` | - |

---

## Memory Maintenance Tools

### Analyze Memory Patterns
See what's being remembered and identify issues:
```
"Analyze my memory patterns from the last week"

Tool call:
analyze_memory_patterns(time_range="week")  # Options: "today", "week", "month", "all"
```

### Find Duplicates
```
"Find duplicate memories"

Tool call:
deduplicate_memories(similarity_threshold=0.95, dry_run=True)
```
Set `dry_run=False` to actually delete duplicates.

### Consolidate Old Memories
Merge related memories from the past month:
```
"Consolidate my old memories about cooking"

Tool call:
consolidate_memories(topic="cooking", max_age_days=30)
```

### Export All Memories
Backup your memories:
```
"Export all my memories to JSON"

Tool call:
export_memories(format="json", include_dates=True)
```

### System Health Check
```
"Run a memory health check"

Tool call:
memory_health_check()
```

---

## Configuration & Customization

### Valve Settings Explained

**max_memories** (default: 5)
- Number of memories returned by `get_relevant_memories`
- Increase for broader recall (up to 30)
- Decrease for more focused results

**memory_length_cutoff** (default: 200)
- Max characters per memory in tool results
- Prevents overwhelming context window
- Increase if you need full memory content

**memories_dist_max** (default: 0.75)
- Semantic similarity threshold (0.0 to 1.0)
- Lower = stricter matching (e.g., 0.6)
- Higher = broader matching (e.g., 0.85)
- Adjust based on false positive rate

### For Medical Use
```
max_memories: 10 (need comprehensive recall)
memory_length_cutoff: 500 (full lab values)
memories_dist_max: 0.7 (strict relevance)
```

### For Casual Chat
```
max_memories: 3 (keep responses concise)
memory_length_cutoff: 150 (shorter snippets)
memories_dist_max: 0.8 (broader matching)
```

---

## Troubleshooting

### "No memories found" but I stored something
1. Check if memory filter is running (Ensure OWUI Memory (Experimental) is toggled "On")
2. Try using `add_specific_memory` manually
3. Lower `memories_dist_max` valve - might be filtering too strictly
4. Check DEBUG logs if valve is enabled

### Search returns irrelevant results
1. Lower `memories_dist_max` (e.g., from 0.75 to 0.65)
2. Use more specific search terms
3. Try `search_memories_with_context` for better accuracy
4. Run `deduplicate_memories` to clean up noise

### Story Manager chapter numbers are wrong
- Chapters auto-increment if `chapter_number` not specified
- Use `get_summary` to see actual chapter numbers
- Update with correct chapter number to fix

### Memory system feels slow
1. Run `clear_embedding_cache()` - safe operation that clears model cache
**Note: The embedding cache is shared with your entire system. Clearing the embedding cache with the Hybrid Tool will only clear OWUI embeddings and has no effect on embeddings by other UI's (i.e. LMStudio).
2. Check if you have thousands of memories - run `memory_health_check()`
3. Consider running `consolidate_memories()` to merge old data

### Tool doesn't appear in Open WebUI
1. Check Open WebUI version (requires v0.6.33+)
2. Restart Open WebUI after uploading
3. Check admin logs for Python errors
4. Verify file is valid JSON
5. Close and restart the Ollama server

---

## Comparison with Built-in Memory

| Feature | Built-in Memory | Hybrid Memory Tool |
|---------|----------------|-------------------|
| Semantic Search | ✅ Basic | ✅ Enhanced with filtering |
| Context Tracking | ❌ No | ✅ With surrounding conversation |
| Manual Control | ⚠️ Limited | ✅ Full add/delete/update |
| Chronological Data | ❌ No | ✅ Story Manager |
| Maintenance Tools | ❌ No | ✅ Dedupe, consolidate, analyze |
| Export/Backup | ❌ No | ✅ Multiple formats |
| Medical Use | ⚠️ Not recommended | ✅ Production-tested |
| Relevance Tuning | ❌ Fixed | ✅ Configurable valves |
| Data Safety | ⚠️ Basic | ✅ Update protection, validation |

---

## Best Practices

### For Personal Use
1. Let automatic memory storage work naturally
2. Use manual `add_specific_memory` for critical facts
3. Periodically run `deduplicate_memories` to clean up
4. Export memories monthly as backup

### For Medical Data
1. **Use Story Manager exclusively** - don't rely on automatic storage
2. Always include metadata (dates, physicians, lab types)
3. Keep backups: run `export_for_physician` after each update
4. Use descriptive story names: `PatientName_Labs`, `PatientName_Imaging`
5. Never rely solely on memory - keep airgapped backups

### For Creative Writing
1. Use Story Manager for serialized content
2. Call `get_full_story` before writing new chapters
3. Use `get_summary` to track progress
4. Export completed stories with `export_for_physician` (yes, works for any text)

### General Tips
- Be specific in your queries for better recall
- Use `search_memories_with_context` when you need to understand the full conversation
- Run `memory_health_check` if something feels off
- Check DEBUG logs (set valve to true) if troubleshooting

---

## Technical Notes

### Requirements
- Open WebUI v0.6.31 or higher (tested on v0.6.34)
- Python 3.9+
- Works with any LLM backend (Ollama, OpenAI API, etc.)

### Data Storage
- Memories stored in Open WebUI's native ChromaDB vector database
- Story chapters stored as specially formatted memories with prefix `STORY_CHAPTER:`
- Metadata embedded in chapter content as JSON
- No external dependencies or databases required

### Performance
- Handles thousands of memories efficiently
- Story Manager supports unlimited chapters per story
- Context search capped at 2000 characters for safety
- Search results limited by `max_total_chars` (10,000 default)

### Safety Features
- Update operations save before deleting (prevents data loss)
- Metadata automatically preserved during updates
- Input validation on all parameters
- Comprehensive error handling and logging
- Dry-run modes for destructive operations

---

## Support & Contributing

This tool is open source under MIT license. Community contributions welcome!

**Found a bug?** Open an issue with:
- Open WebUI version
- Tool version
- Description of the issue
- Steps to reproduce

**Want to contribute?** Submit a pull request with:
- Clear description of changes
- Test results
- Compatibility notes

**Questions?** Check the troubleshooting section first, then ask in the Open WebUI community forums.

---

## Credits

**Original Development:** Slye Fox & Claude (Anthropic)  
**Inspired by:** 
- CookSleep's memory tools
- wawawario2's LTM script

**Special Thanks:**
- Open WebUI development team
- Open WebUI community for feedback and testing

---

## License

AGPL-3.0 License - See LICENSE file for details

Copyright (C) 2025 Slye Fox

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published
by the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program. If not, see <https://www.gnu.org/licenses/>.
