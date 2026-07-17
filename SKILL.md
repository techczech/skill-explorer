---
name: skill-explorer
description: "Visualize SKILL.md structure."
---

# Skill Explorer — Interactive Skill Visualization

Generate an interactive HTML visualization from any SKILL.md-based skill. Produces a single `index.html` with 4 tabs: foldable overview, structure treemap, code browser, and connection graph.

## Scope

Only handles standalone **SKILL.md-based skills** — the `05_skills/` pattern with YAML frontmatter + markdown + companion scripts. Does NOT handle NanoClaw internal skills.

## Workflow

### 1. Ask the User

Ask the user:
- **Which skill** to visualize — get the path to the skill directory (e.g. `~/gitrepos/05_skills/text-to-image`)
- **Where to output** — the directory for the generated `index.html` and companion files
- **Deploy?** — whether to deploy to Cloudflare Pages after generation

### 2. Read the Target Skill

Read the skill's `SKILL.md` file. Parse:
- **YAML frontmatter** — extract `name`, `description`, triggers
- **Markdown body** — identify all H2 (`##`) and H3 (`###`) sections

### 3. Inventory Companion Files

List all files in the skill directory. Look for:
- `scripts/` — shell scripts, Python scripts
- `references/` — documentation, guides
- `examples/` — example files
- Any other companion files (not `SKILL.md` itself, not `.git`, not `venv/`)

### 4. Analyze Each Section

For each H2/H3 section in the SKILL.md:
- **Summarize** its purpose (1 sentence)
- **Identify referenced files** — scripts, configs, paths mentioned in the section
- **Count** lines, code blocks (``` fenced blocks), tables (| delimited)
- **Classify** section type:
  - `setup` — Prerequisites, installation, environment setup
  - `workflow` — How-to steps, procedures, generation workflows
  - `reference` — Tables, comparisons, specifications, guides
  - `troubleshooting` — Debugging, issues, solutions
  - `other` — Related skills, notes, appendices

### 5. Read Companion Files

For each companion file found in step 3:
- Read the file content
- Note: line count, language (from extension), brief description (1 sentence)
- Note: which SKILL.md sections reference this file

### 6. Build the Data Model

Construct the JSON data model matching this structure:

```javascript
const DATA = {
  skill: {
    name: "skill-name",
    description: "From YAML frontmatter",
    triggers: ["trigger1", "trigger2"],
    totalLines: 381,
    path: "SKILL.md"
  },
  sections: [
    {
      id: "section-id",        // kebab-case of heading
      heading: "Section Title",
      level: 2,                // 2 for H2, 3 for H3
      type: "workflow",        // setup | workflow | reference | troubleshooting | other
      startLine: 19,
      endLine: 43,
      lines: 25,
      summary: "Brief description of the section",
      codeBlocks: 2,
      tables: 0,
      referencedFiles: ["scripts/setup.sh"],
      markdown: "## Section Title\n\nRaw markdown content..."
    }
  ],
  files: [
    {
      path: "scripts/extract-pdf.sh",
      name: "extract-pdf.sh",
      language: "bash",
      lines: 120,
      description: "Brief description",
      content: "#!/bin/bash\n...",
      referencedBy: ["quick-start", "workflow"]
    }
  ],
  docs: [
    { path: "SKILL.md", name: "SKILL.md", description: "Main skill documentation", content: "---\nname: ...\n---\n# ..." }
  ]
};
```

### 7. Generate the HTML

1. Copy the template from `~/gitrepos/05_skills/skill-explorer/templates/skill-viz.html`
2. Find the `const DATA = {};` placeholder near the top of the `<script>` block
3. Replace it with `const DATA = <your JSON>;` using the data model from step 6
4. Write the result to the user's chosen output directory as `index.html`

### 8. Optional Deploy

If the user wants deployment:

```bash
cd <output-directory>
npx wrangler pages deploy . --project-name=<skill-name>-explorer
```

## Important Notes

- The template is **self-contained** — all CSS, JS, and rendering logic is in the single HTML file
- File contents are embedded in the DATA object — no external file loading needed
- The template handles markdown rendering, syntax highlighting, treemap layout, and graph drawing
- Section IDs are generated as kebab-case from headings (e.g. "Quick Start" becomes "quick-start")
- File language is detected from extension: `.sh`/`.bash` = bash, `.py` = python, `.ts` = typescript, `.js` = javascript, `.md` = markdown, `.json` = json, `.yaml`/`.yml` = yaml

## Verification

After generating, open `index.html` in a browser. Check:
1. **Skill Overview** — all sections fold/unfold, file references are clickable
2. **Structure Map** — treemap shows sections sized by line count
3. **Browse Code** — sidebar lists all files, clicking shows syntax-highlighted content
4. **Connections** — graph shows section-to-file relationships
