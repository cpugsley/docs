# Contribute to the CloudConcierge documentation

This guide covers how to edit these docs, MDX gotchas that have bitten us, and how Mintlify's auto-generated PRs are reviewed.

## How to contribute

### Option 1: Edit directly on GitHub

1. Navigate to the page you want to edit
2. Click the "Edit this file" button (the pencil icon)
3. Make your changes
4. Submit a pull request to `main` — direct pushes to `main` are blocked

### Option 2: Local development

1. Clone this repository
2. Install the Mintlify CLI: `npm i -g mint`
3. Create a branch: `git checkout -b docs/your-change`
4. Make your changes
5. Run `mint dev` from the repo root to preview at `http://localhost:3000`
6. Commit, push, and open a pull request to `main`

## MDX gotchas — read this before adding content

Mintlify renders `.mdx` files with a stricter parser than typical Markdown. Plain Markdown that looks fine in your editor can break the build. Two patterns have bitten us:

### 1. Dollar signs near digits trigger LaTeX parsing

Mintlify's parser treats `$...$` as LaTeX math delimiters when content matches LaTeX shapes. Paired dollar signs on the same line are the most common failure:

```mdx
<!-- BROKEN — $159/mo = $1,908/year reads as math span -->
Professional Annual ($159/mo = $1,908/year)

<!-- FIXED — escape each $ that precedes a digit -->
Professional Annual (\$159/mo = \$1,908/year)
```

**Rule:** any literal dollar amount in body text uses `\$`. Tables, examples, and inline prose all follow the same convention.

Wrapping in backticks (`` `$159/mo` ``) also works and renders the price as code — fine for pricing tables, less natural in prose.

### 2. `<` followed by a digit is parsed as a JSX tag

The character `<` opens a JSX element in MDX. If the next character isn't a valid tag-name start (letter, `_`), the parser errors:

```mdx
<!-- BROKEN — <50% read as start of JSX tag with name "50" -->
Concerning: <50% conversion rate

<!-- FIXED — rephrase or escape -->
Concerning: below 50% conversion rate
Concerning: less than 50% conversion rate
Concerning: \<50% conversion rate
```

`>` followed by a digit is fine — only `<` triggers this.

### 3. Curly braces in body text are parsed as JSX expressions

`{` in MDX opens a JavaScript expression. Body text that contains literal braces breaks:

```mdx
<!-- BROKEN — {team member name or 'someone'} read as JS, parser fails -->
1. AI says "Let me get you to {team member name or 'someone'} right away."

<!-- FIXED — remove the braces or escape them -->
1. AI says "Let me get you to the team member's name (or 'someone') right away."
1. AI says "Let me get you to \{team member name or 'someone'\} right away."
```

Curly braces ARE valid inside JSX component props (e.g. `<Frame caption={...}>`). The issue is only braces in plain body text.

### Sanity check before pushing

If you're touching pricing or examples, grep before committing:

```bash
# Dollar signs that need escaping (false positives are OK — already-escaped \$ will not match)
grep -rn --include="*.mdx" '[^\\]\$[0-9]' .

# < followed by digit (the dangerous pattern)
grep -rn --include="*.mdx" '<[0-9]' .

# Curly braces in body text (manual review; component-prop braces are fine)
grep -rn --include="*.mdx" '{[^a-zA-Z_$]' .
```

If any of those produce hits in content you're editing, fix them before pushing.

## Writing guidelines

- **Use active voice**: "Run the command" not "The command should be run"
- **Address the reader directly**: Use "you" instead of "the user"
- **Keep sentences concise**: Aim for one idea per sentence
- **Lead with the goal**: Start instructions with what the user wants to accomplish
- **Use consistent terminology**: Don't alternate between synonyms for the same concept
- **Include examples**: Show, don't just tell
- **Match existing voice**: Builder talking to a builder, not consultant to client. Concrete file paths, line numbers, real numbers over generic optimism.

## Mintlify bot PRs — review policy

Mintlify periodically opens PRs to this repo for things like:
- Adding new features (changelog tab, OpenAPI auto-routing, etc.)
- Suggesting structural improvements
- Updating starter content

**These PRs are reviewed manually, not auto-merged.** Specifically:

1. Read what the bot changed. Pay attention to `docs.json` modifications — bot PRs have added navigation tabs that made existing pages appear "hidden" behind a default-selected new tab.
2. If the bot changed `docs.json` navigation order or structure, verify the result is what you want. Tab order matters — Mintlify's left nav shows pages for the currently-selected tab, so the first tab is the default landing.
3. Test in a Mintlify preview before merging.
4. After merging, eyeball the live site for the next ~5 minutes to confirm rebuild succeeded and nothing is missing.

## Branch protection on `main`

Direct pushes to `main` are disabled. All changes route through PRs:
- Required: at least one approving review
- Required: status checks pass (Mintlify build preview)
- Mintlify bot PRs follow the same gate — review before merging

If you have repo-admin access and need to bypass for an urgent fix, document why in the commit message and consider whether the rule itself needs to be loosened.

## Common content patterns

- **Pricing**: use `\$N`, not `$N`. Confirm with the grep above before pushing.
- **Comparisons**: use "below 50%" or "under 80%" instead of `<50%` or `<80%`.
- **Template syntax in examples**: when documenting placeholder syntax like `{{customerName}}`, wrap the whole example in a code block (triple-backtick) so MDX doesn't try to interpret it.
- **Component props**: JSX attributes accepting expressions (e.g. `<Frame caption={...}>`) are fine — the MDX parser handles those correctly.
