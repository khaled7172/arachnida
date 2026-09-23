# Project Agnostic Walkthrough Template & Instructional Rules (`template.md`)

This template defines the mandatory, standardized 8-part structure used across every `.md` file in our walkthrough pipeline for any codebase.

Whenever creating or modifying a walkthrough file for a codebase module, you **must** follow this exact format, explaining every single line of code and assuming zero prior programming knowledge on the part of the reader.

---

## 📋 Mandatory Frontmatter Box

Every walkthrough file must begin immediately with a blockquote frontmatter box containing exact metadata about the file's position and role in the pipeline:

```markdown
# `filename.ext` — High-Level Feature Title

> **Pipeline position**: [X] of [Y]
> **What this file does in plain English**: 3–5 sentences explaining what this file accomplishes in everyday language without assuming technical background. Use analogies if helpful.
> **Files it depends on**: `file1.ext`, `file2.ext`
> **Files that depend on it**: `file3.ext`, `main.ext`
```

---

## 📑 Section 1: Line-by-Line Code Breakdown

For every logical block or function in the file, create a section titled by exact line numbers and code signature:

```markdown
## Lines [Start]–[End]: `function_or_class_signature(...)`

### The Code
\`\`\`language
// Exact, verbatim code from the source file
\`\`\`

### Plain English
2–3 paragraphs explaining what this specific block of code is doing, why it is needed, and how data flows through it. Assume the reader knows nothing about programming.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 10 | \`code snippet\` | Plain English explanation of this specific line |
```

---

## 💡 Section 2: New Concepts Introduced

Immediately following any code section that introduces a new syntax feature, design pattern, library, or core concept, insert a blockquote concept card:

```markdown
### New Concepts Introduced

> **`Concept Name`**: An explanation of what this concept is, why the language/framework uses it, and how it helps prevent bugs or improve performance.
```

---

## 📊 Section 3: Complete ASCII Data Flow Diagram

Every file walkthrough must include at least one complete ASCII diagram visualizing how inputs transform into outputs across the functions inside that file:

```markdown
## 📊 Complete Data Flow Diagram

\`\`\`text
[Raw Input / Argument]
          │
          ▼
   `function_one(input)`
     Performs initial step
          │
          ▼
[Final Output / Return Value]
\`\`\`
```

---

## 🧠 Section 4: Key Takeaways

Summarize the most important engineering principles and design choices demonstrated in the file:

```markdown
## 🧠 Key Takeaways

1. **First Key Insight**: Concise explanation of why a design choice was made.
2. **Second Key Insight**: Explanation of performance, memory, or architectural benefits.
3. **Third Key Insight**: Explanation of how this module connects to the broader system.
```

---

## 📎 Section 5: Concepts Introduced in This File

Provide a master summary table listing every programming concept introduced throughout the file:

```markdown
## 📎 Concepts Introduced in This File

| Concept | First seen at line | Quick definition |
|---------|-------------------|-----------------|
| Concept 1 | 4 | Plain English definition |
| Concept 2 | 15 | Plain English definition |
```

---

## ➡️ Section 6: What's Next

End the file with a clear pointer to the next file in our sequence:

```markdown
## ➡️ What's Next

Next up: **`next_file.ext`** — Brief teaser explaining what we will build next and how it connects to the file we just completed!
```

---

## 📏 Mandatory Writing & Instructional Rules

1. **Zero Prior Knowledge Assumption**: Never assume the reader knows what an `import`, `function`, `loop`, or `variable` is without defining it the first time it appears in the pipeline sequence.
2. **No Skipped Lines**: Every single line of code in the target source file must appear inside a line-by-line breakdown table. Do not write `"Lines 20–50: same as above"` or omit helper functions.
3. **No Unexplained Jargon**: If you mention technical terms, you must define them immediately with clear analogies.
4. **Exact Line Numbers**: All table row line numbers must exactly match the real line numbers of the source code file on disk.
5. **Architectural Alignment**: Whenever an implementation choice is explained, explicitly highlight how this choice optimizes speed, memory, scalability, or security for the overarching system architecture.
