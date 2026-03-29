---
name: stock-report-writer
description: Takes raw stock analysis and produces a clean, simplified markdown report. Explains financial jargon in plain English. Used by the /stock-report command via the Report Writer agent.
---

# Stock Report Writer

Transform raw stock analysis into a clean, readable markdown report. This skill does **not** fetch data or form new opinions — it formats, simplifies, and writes.

Used by the **Report Writer** agent in the `/stock-report` command.

---

## Input

The Report Writer agent receives:
- **Path to `.raw.md`** — the full raw analysis from the Stock Analyst agent
- **Path to `.sources.md`** — the approved sources list from the Source Scout agent
- **Output path** — where to write the final `.md` report
- **Stock name and date** — for file naming and headings

The raw analysis may vary in structure depending on the stock (small-cap vs large-cap, banking vs pharma, etc.). The writer should adapt to whatever sections are present rather than expecting a rigid format.

---

## Workflow

1. **Read the raw analysis file** (`.raw.md`).

2. **Read the sources file** (`.sources.md`) for the Sources section at the end of the report.

3. **Map raw analysis sections to the report template.** Use `assets/simplified-report-template.md` as the structural backbone. For each section:
   - If data exists in the raw analysis → fill the section
   - If data is missing → skip the section entirely (do not include empty sections or placeholder text)
   - If data is partial → include what's available with a note

4. **Simplify financial jargon.** Use `references/glossary.md` to add plain-English explanations:
   - In ratio tables, add a "What It Means" column
   - For banking/sector-specific metrics, add brief inline explanations
   - Do not over-explain obvious terms (e.g., "Revenue" needs no explanation)
   - Only explain terms that a non-finance reader would not immediately understand

5. **Apply tone guidelines:**
   - Neutral and direct — not formal, not conversational
   - Get to the point without being dry or boring
   - Use active voice where possible
   - Bold key numbers and important observations
   - No filler phrases ("It is worth noting that...", "Interestingly...")

6. **Write the conclusion.** Every report must end with a clear conclusion:
   - Summarize the investment case in 2–3 sentences
   - State what to watch going forward (3–5 monitoring parameters)
   - State review triggers (what would change the thesis)
   - End with a one-line bottom-line statement

7. **Check for existing file.** Before writing:
   - If the output path already has a `.md` report → ask the user: "A report for [stock] on [date] already exists. Overwrite?"
   - If user says yes → overwrite
   - If user says no → stop

8. **Write the report** to the output path.

9. **Update `stock-reports/README.md`.** Add a new row to the Index table:
   - Read the current README
   - Add the new entry at the top of the table (newest first)
   - Include relative links to all three files (report, raw, sources)
   - Write the updated README

---

## Formatting Rules

- Use GitHub-flavored markdown with tables, headers, and bullet points
- Present numerical data in tables where possible
- Bold key metrics and important observations
- Round financial ratios to 2 decimal places
- Express large numbers in Indian convention: Cr (Crores) and L (Lakhs)
- Include directional indicators (+/- signs) for percentage changes
- Use `---` horizontal rules between major sections
- Include a data timestamp near the top so the reader knows how current the data is
- Always include a disclaimer section before Sources
- Keep tables aligned and readable

---

## Glossary Usage

When presenting metrics in tables, check `references/glossary.md` for plain-English equivalents. Apply them as follows:

**For key ratio tables** — add a "What It Means" column:

```markdown
| Metric | Value | What It Means |
|--------|-------|---------------|
| PE Ratio | 16.86x | Years of current earnings the price reflects |
| NIM | 3.35% | Spread between loan earnings and deposit costs |
```

**For banking/sector metrics** — add inline explanations in a dedicated column or parenthetical:

```markdown
| Metric | Value | Rating |
|--------|-------|--------|
| CASA Ratio | 33.6% | Below par — proportion of cheap deposits (current + savings accounts) |
```

**Do not explain** common terms that any reader would know: Revenue, Profit, Market Cap, Share Price, Volume, Dividend.

---

## Error Handling

- If the raw analysis file is missing or empty → stop and report the error
- If the raw analysis is too sparse (fewer than 3 major sections) → write what's available, note limitations prominently at the top
- If the sources file is missing → write the report without a Sources section, note that sources are unavailable
- If README.md doesn't exist → create it with the standard header and the first index entry
