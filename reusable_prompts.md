# FDE Tools — Reusable Prompts (Shared)

Cross-project prompt library for SAP Forward Deployed Engineering discovery work.
Each prompt ends with "don't do anything yet" so you can correct details before work begins.

> **All prompts assume Claude Code running in the terminal.**

> **SAP-internal access required:** Several resources referenced in these prompts (MCP setup guides, the shared FDE Tools folder, SAP GitHub) are only accessible on the SAP network or VPN.

---

## Setup — define your paths

Before using any prompt below, set these three placeholders to match your machine:

| Placeholder | What it points to | Example |
|---|---|---|
| `[FDE_TOOLS_ROOT]` | Your local copy of the shared FDE Tools folder | `C:\Users\yourname\OneDrive - SAP SE\FDE Tools - Shared` |
| `[DISCOVERY_ROOT]` | Your Discovery projects folder | `C:\Users\yourname\OneDrive - SAP SE\Discovery` |
| `[MUTOOL_PATH]` | Path to mutool.exe (needed for prompt #6 only) | Install via: `winget install ArtifexSoftware.mutool` — then find it under `C:\Users\yourname\AppData\Local\Microsoft\WinGet\Packages\ArtifexSoftware.mutool_*\` |

---

## 1. Build a Word spec from a Markdown file

Use this whenever you have a completed or partially completed spec in Markdown and want to generate a formatted Word document matching the SAP FDE spec style.

**Prerequisites:**
- A filled-in `.md` spec file (copy and fill `spec_template.md` from this folder)
- A styled Word `.docx` to use as the formatting template (e.g. a prior spec from the same client)
- The build script at `[FDE_TOOLS_ROOT]\build_spec_docx.py`

**Prompt:**

> I want to generate a formatted Word spec document from a Markdown file.
>
> Please run the build script at:
> `[FDE_TOOLS_ROOT]\build_spec_docx.py`
>
> With these arguments:
> - `--template` : [PATH TO STYLED WORD TEMPLATE .docx]
> - `--input`    : [PATH TO YOUR FILLED-IN .md FILE]
> - `--output`   : [PATH FOR THE GENERATED .docx OUTPUT]
>
> Run the script and report the result. If there are errors, fix them and re-run.
> Don't do anything yet — confirm you can see all three files exist before running.

**Notes:**
- The script clones the cover page, logo, and metadata table from the `--template` file. Use a Word doc that already has the correct client branding.
- Output goes to `claude_output\` by convention.
- If the `72 Brand` font is not installed on the machine opening the doc, Word will substitute a fallback — this is expected on non-SAP machines.

---

## 2. Start a new discovery project

Use this when spinning up a new client project folder from scratch.

> I'm starting a new FDE discovery project for [CLIENT NAME].
>
> Please set up the project folder at:
> `[DISCOVERY_ROOT]\[CLIENT NAME]\`
>
> Create this folder structure:
> - `Spec work\[CLIENT NAME] spec\` — for Word and MD spec files
> - `Spec work\[CLIENT NAME] spec\MD versions\` — for markdown versions
> - `Demo\` — for HTML demos
> - `Diagrams\` — for draw.io and SVG files
> - `Meetings\` — for session recordings and transcripts
> - `From Client\` — for client-provided materials
> - `claude_output\` — for generated files
> - `tools\` — for project-specific scripts
>
> Also copy these files from `[FDE_TOOLS_ROOT]\`:
> - `spec_template.md` → `Spec work\[CLIENT NAME] spec\MD versions\[CLIENT NAME]_Spec_v0.1.md`
> - `.mcp.json` → project root (enables Playwright, SAP Jira, Teams, Wiki, Outlook, and GitHub MCP tools)
>
> Don't do anything yet — confirm the structure looks right first.

---

## 3. Process a new meeting recording into a UI reference

Use this whenever you have a new Teams recording to process.

> I have a new Teams meeting recording I want to process into a UI reference document. The pipeline is:
>
> 1. **Extract stills** every 5 seconds from the video
> 2. **Crop** the rightmost 13% (black participant panel) from each frame
> 3. **Deduplicate** consecutive similar frames using a stacked approach:
>    - Layer 1: exact pixel hash (MD5) — drop identical frames
>    - Layer 2: perceptual hash (pHash, hamming distance < 8) — drop near-identical frames like single-character typing changes
>    - Layer 3: pixel diff on left 55% of frame only (the form panel), 70% similarity threshold — drop frames where only the static right-side document image differs
> 4. **Save frames** to `claude_output/[session_name]_frames/`
> 5. **Analyze** all frames + the meeting transcript using **10 parallel agents (~18 frames each)** — use 10 not 15; fewer agents = fewer failure points
> 6. **Write each agent's output to a numbered part file immediately** as it completes — `Meetings/[date]/[session_name]_part01.md` through `_part10.md`. Do not hold results in memory — this checkpoints the work so a later failure doesn't lose everything.
> 7. **Write UI reference** to `Meetings/[date]/[session_name]_UI_reference.md` — a dedicated synthesis agent reads the 10 part files (not raw images) and merges them into a structured reference document for future Claude sessions, not a human narrative.
> 8. **Write transcript delta** to `Meetings/[date]/[session_name]_transcript_delta.md` — only after the UI reference is confirmed written. Contains ONLY content from the transcript that is absent from or meaningfully clarifies the UI reference. Do NOT re-summarize what the UI ref already covers. Each section states briefly what the UI ref says (if anything), then gives the richer transcript context. End with a "Next Session Note" section if the transcript mentions a next meeting or upcoming focus. Use the .txt transcript, not the .docx.
>
> Part files are intermediate checkpoints — delete them after the UI reference is confirmed written.
>
> The video is: "[PATH TO VIDEO]"
> The transcript is: "[PATH TO TRANSCRIPT .txt]"
>
> Don't do anything yet — confirm you understand and flag any issues.

---

## 4. Sync a content change across spec and demo files

Use this when a logic or wording change needs to be applied consistently across multiple files.

> I need to make a change to the [CLIENT] agent content. The change is:
>
> **What's changing:** [DESCRIBE THE CHANGE]
>
> **Why:** [ONE SENTENCE REASON]
>
> Before making any edits, grep all relevant files for the affected terms and show me every location that needs updating:
> - [LIST FILES]
>
> Show me what you find before making any edits. Don't do anything yet.

---

## 6. Create a SAP Fiori UI render

Use this when building a static HTML mockup that follows SAP Fiori Horizon aesthetics and SAP brand guidelines.

**Prerequisites:**

The SAP brand assets (design tokens, icon list, UI examples, and brand colour CSS) live in the `Brand\` folder inside the shared FDE Tools folder. These are SAP-internal files — **request the folder from ACN team members who are on the SAP side**. They can share it via Teams or the FDE Engagements SharePoint.

Once you have the folder, place it at `[FDE_TOOLS_ROOT]\Brand\` and proceed with the prompt below.

**Key rules from the brand files:**
- Use `--sap-blue-7` (`#0070F2`) as the primary CTA colour; white shell bar (`--fiori-shell: #FFFFFF`); `#F5F6F7` page background
- Indigo = AI/agent concepts only; Red = error/functional only, never decorative
- Event/status chips: tinted background (colour-2) + matching text (colour-7), e.g. `background: #D1EFFF; color: #0070F2`
- Icons: use `<ui5-icon name="...">` with named icons verified against `SAP-icons.json` — never guess unicode codepoints
- Load UI5 web components via esm.sh (resolves bare module specifiers correctly):
  ```html
  <script type="module" src="https://esm.sh/@ui5/webcomponents@2.25.1/dist/bundle.esm.js"></script>
  <script type="module" src="https://esm.sh/@ui5/webcomponents-icons@2.25.1/dist/AllIcons.js"></script>
  ```

**Prompt:**

> I want to create a static HTML UI render following SAP Fiori Horizon design standards.
>
> Before designing, read these files from `[FDE_TOOLS_ROOT]\Brand\`:
> - `sap-brand.css`
> - `fiori-horizon-design-tokens.css`
> - `UI examples to follow.pdf` (convert with mutool if needed — mutool.exe is at `[MUTOOL_PATH]`)
> - `UI elements and other graphics.pdf`
>
> For any icons, look up exact names from:
> `[FDE_TOOLS_ROOT]\Brand\ui5-webcomponents-icons\dist\generated\assets\v5\SAP-icons.json`
>
> The screen I want to design is: **[DESCRIBE THE SCREEN — what it shows, who uses it, what the main interaction is]**
>
> Output a single self-contained HTML file to: `[OUTPUT PATH]`
>
> No interactivity yet — this is a static render only.
> Don't do anything yet — confirm you can read all the brand files first.

**Notes:**
- Name files with a version suffix (`-v2`, `-v3`) when iterating — keep prior versions
- Use a local HTTP server + Playwright to screenshot renders for verification (`npx serve . --listen [PORT]`)

---

## 7. Extract a transcript from a Teams recording (SharePoint video)

Use this when a Teams meeting **has a video recording hosted on SharePoint** and you need the transcript as a text file. For transcript-only (no recording), use section 8 instead. The transcript panel must be open and visible before running.

**How to use:**
1. Open the recording in SharePoint and click the **Transcript** tab to make it visible
2. Open browser DevTools (F12) → Console
3. Paste and run the script below
4. It auto-scrolls the transcript panel, collects all entries, then downloads `transcript.txt`

```javascript
const panel = document.querySelector('[class*="focusZoneWithSearchBox"]');
let collected = new Set();
const scroll = () => {
  document.querySelectorAll('[class*="listItemWithSpeaker"]').forEach(el => collected.add(el.innerText));
  panel.scrollTop += 400;
  if (panel.scrollTop + panel.clientHeight < panel.scrollHeight) setTimeout(scroll, 400);
  else {
    document.querySelectorAll('[class*="listItemWithSpeaker"]').forEach(el => collected.add(el.innerText));
    const a = document.createElement('a');
    a.href = URL.createObjectURL(new Blob([[...collected].join('\n\n')], {type:'text/plain'}));
    a.download = 'transcript.txt';
    a.click();
    console.log('done, items:', collected.size);
  }
};
panel.scrollTop = 0;
setTimeout(scroll, 500);
```

**Notes:**
- The script scrolls `focusZoneWithSearchBox` — this is the actual scrollable container; other containers like `contentContainer` or `listContainerWithSearch` don't work
- Uses a `Set` to deduplicate items that appear in multiple scroll windows
- The console will log `done, items: N` when complete — N should match the visible transcript entry count
- Once downloaded, rename the file to match the meeting (e.g. `ClientName_Session_20260921_transcript.txt`) and place it in the meeting's `Meetings/[date]/` folder

---

## 8. Extract a transcript from a Teams chat (transcript-only, no recording)

Use this when a Teams meeting has only a transcript (no video recording) — accessed via the **Transcript** tab in a Teams chat. The transcript must be open and visible before running.

**How to use:**
1. Open the Teams chat, click the **Transcript** tab to open it
2. Scroll the transcript panel manually to the top (click inside it, press Ctrl+Home)
3. Open browser DevTools (F12) → Console
4. Paste and run the script below
5. It scrolls the virtualised list slowly, collecting entries as it goes, then downloads `transcript.txt` automatically

```javascript
(async () => {
  const firstEntry = document.querySelector('[data-testid="entryRenderer"]');
  if (!firstEntry) { console.error('No entryRenderer found'); return; }

  let scrollable = firstEntry.parentElement;
  while (scrollable) {
    const style = getComputedStyle(scrollable);
    if ((style.overflowY === 'scroll' || style.overflowY === 'auto') && scrollable.scrollHeight > scrollable.clientHeight) break;
    scrollable = scrollable.parentElement;
  }
  if (!scrollable) { console.error('Scrollable parent not found'); return; }

  const seen = [];
  const seenSet = new Set();

  const collect = () => {
    document.querySelectorAll('[data-testid="entryRenderer"]').forEach(el => {
      const text = el.innerText.trim();
      if (text && !seenSet.has(text)) {
        seenSet.add(text);
        seen.push(text);
      }
    });
  };

  scrollable.scrollTop = 0;
  await new Promise(r => setTimeout(r, 1000));
  collect();

  while (scrollable.scrollTop + scrollable.clientHeight < scrollable.scrollHeight) {
    scrollable.scrollTop += 150;
    await new Promise(r => setTimeout(r, 100));
    collect();
  }

  console.log(`Collected ${seen.length} entries`);
  const blob = new Blob([seen.join('\n\n')], { type: 'text/plain' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'transcript.txt';
  a.click();
  console.log('Done');
})();
```

**Notes:**
- **Requires the browser version of Teams (`teams.microsoft.com`), not the desktop app** — the script needs access to the browser DevTools console, which the desktop app does not expose. Open the meeting chat in a browser if you are currently in the Teams desktop app.
- **The transcript panel is inside a cross-origin iframe.** Before pasting the script, click the context dropdown at the top of the Console tab (it says "top") and select **RecapxPlatIframe (xplatplugins.aspx)** from `sapnam-my.sharepoint.com`. Run the script from that iframe context — running it from "top" will return "No entryRenderer found" even if the transcript is visible.
- **Note:** The iframe switch may or may not be required depending on the meeting. If `entryRenderer` is not found, the first thing to try is switching to the RecapxPlatIframe context.
- Teams virtualises the transcript list — items are removed from the DOM as you scroll. This script collects entries continuously during the scroll so nothing is lost.
- Must scroll to the top manually before running — the script starts from `scrollTop = 0` but Teams may not rerender already-passed items if you skip this step.
- If the downloaded file is missing the middle section, the scroll was too fast — increase the delay from `100` to `200` ms.
- The console will log `Collected N entries` and `Done` when complete. For a ~56 min meeting expect ~400–450 entries.
- Once downloaded, rename to match the meeting (e.g. `ClientName_Session_20260929_transcript.txt`) and place in `Meetings/[date]/`.
- **This script is for the Teams chat Transcript tab only.** For SharePoint-hosted recordings, use entry 7 instead.

---

## 9. Clean a transcript for use as Claude context

Use this after extracting a raw transcript (via entry 7 or 8) to produce a leaner version optimized for feeding to Claude as context.

**Prompt:**

> I have a Teams meeting transcript I want to clean up.
>
> Input: `[PATH TO transcript.txt]`
> Output: `[PATH TO transcript_clean.txt]` (new file — do not modify the original)
>
> Write the following script to a temp file and run it with those two paths as arguments:
>
> ```python
> """
> clean_transcript.py — clean a Teams meeting transcript for use as Claude context
> 
> Handles both Teams transcript formats:
>   - Inline timestamp: "Speaker Name (suffix) [bracket] X minutes Y seconds"
>   - Separate line:    "Speaker Name" on one line, "X minutes Y seconds" on the next
> 
> Transformations applied:
>   1. Strip timestamps and system messages (started/stopped transcription)
>   2. Merge consecutive same-speaker utterances into one block
>   3. Simplify speaker names (remove parentheticals and brackets)
> """
> 
> import re
> import sys
> 
> def simplify_name(name):
>     name = re.sub(r'\s*\(.*?\)', '', name)
>     name = re.sub(r'\s*\[.*?\]', '', name)
>     return name.strip()
> 
> def clean_transcript(input_path, output_path):
>     with open(input_path, 'r', encoding='utf-8') as f:
>         raw = f.read()
> 
>     blocks = [b.strip() for b in re.split(r'\n\s*\n', raw) if b.strip()]
> 
>     entries = []
>     for block in blocks:
>         lines = block.splitlines()
>         if not lines:
>             continue
>         if any(kw in block for kw in ['started transcription', 'stopped transcription']):
>             continue
> 
>         first_line = lines[0]
>         first_line = re.sub(r'\s+(\d+\s+hour[s]?\s+)?(\d+\s+minute[s]?\s*)?(\d+\s+second[s]?)?\s*$', '', first_line).strip()
>         first_line = re.sub(r'\s+\d+\s+minute[s]?\s*$', '', first_line).strip()
>         speaker = simplify_name(first_line)
> 
>         rest_start = 1
>         original_first = lines[0].strip()
>         first_was_bare_abbrev = ' ' not in original_first
>         if first_was_bare_abbrev and len(lines) > 1:
>             candidate = lines[1].strip()
>             is_plain_name = (
>                 candidate
>                 and not re.search(r'\d', candidate)
>                 and not re.match(r'^\d+:\d+', candidate)
>                 and len(candidate) > len(speaker)
>                 and len(candidate.split()) <= 5
>             )
>             if is_plain_name:
>                 speaker = simplify_name(candidate)
>                 rest_start = 2
> 
>         content_lines = []
>         for line in lines[rest_start:]:
>             stripped = line.strip()
>             if re.match(r'^\d+:\d+(:\d+)?$', stripped):
>                 continue
>             if re.match(r'^(\d+\s+hour[s]?\s+)?(\d+\s+minute[s]?\s*)?(\d+\s+second[s]?)?\s*$', stripped) and re.search(r'\d', stripped):
>                 continue
>             if re.match(r'^.+\s+(\d+\s+hour[s]?\s+)?(\d+\s+minute[s]?\s*)?(\d+\s+second[s]?)?\s*$', stripped) and re.search(r'\d+\s+(hour|minute|second)', stripped):
>                 continue
>             content_lines.append(stripped)
> 
>         text = ' '.join(content_lines).strip()
>         text = re.sub(r'^\d+:\d+(:\d+)?\s+', '', text).strip()
>         text = re.sub(r'\d+:\d+\s+[A-Z][^\d]+?(\d+\s+hour[s]?\s+)?(\d+\s+minute[s]?\s+)?\d+\s+second[s]?\s*', '', text).strip()
>         if speaker and text:
>             entries.append((speaker, text))
> 
>     merged = []
>     for speaker, text in entries:
>         if merged and merged[-1][0] == speaker:
>             merged[-1] = (speaker, merged[-1][1] + ' ' + text)
>         else:
>             merged.append((speaker, text))
> 
>     with open(output_path, 'w', encoding='utf-8') as f:
>         for speaker, text in merged:
>             f.write(f'**{speaker}**\n{text}\n\n')
> 
>     print(f'Done. {len(merged)} entries written to {output_path}')
> 
> if __name__ == '__main__':
>     if len(sys.argv) != 3:
>         print('Usage: python clean_transcript.py <input.txt> <output.txt>')
>         sys.exit(1)
>     clean_transcript(sys.argv[1], sys.argv[2])
> ```
>
> Run the script and report how many entries were written. Don't do anything yet — confirm both file paths look correct first.

**Notes:**
- This produces a file that is typically 20–70% smaller than the raw transcript depending on how timestamps were exported. All substantive content is preserved.
- Keep short filler utterances ("Yeah.", "Mhm.") — they preserve conversational flow and help Claude understand agreement, interruptions, and tone
- Name the output file with a `_clean` suffix (e.g. `ClientName_Session_20260929_transcript_clean.txt`) and place it alongside the original in `Meetings/[date]/`
- Use the clean version when giving Claude transcript context — it uses ~7% of context vs ~15% for the raw version for a typical 56-minute meeting

---

## Notes on working across projects

- Spec Word docs and MD files live in `Spec work\[CLIENT] spec\` and its `MD versions\` subfolder
- The build script and spec template live in `[FDE_TOOLS_ROOT]\` — portable across all projects
- `claude_output\` is for generated files — never commit these to GitHub
- MD spec files in `MD versions\` can be committed to GitHub; Word files should not
- Always use the `.txt` transcript, not the `.docx`, when processing meeting recordings
- Say "don't do anything yet" to review a plan before execution starts
- The `72 Brand` font is SAP-internal — docs will render correctly on SAP machines only
