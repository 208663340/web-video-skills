# Codex Adapter

This file maps the original web-video-presentation workflow onto Codex. Use it
when running the skill from Codex CLI or Codex desktop.

## Execution Model

- Treat the main `SKILL.md` as the contract. This adapter only translates tool
  usage, checkpoints, and shell details for Codex.
- Use the user's current workspace as the project root. Create a focused video
  folder only when the user has not already provided one.
- Keep hard checkpoints real. When the workflow says "must stop", stop and ask
  the user unless the user explicitly requested a fully autonomous pass and the
  decision has a safe default in this file.
- In Default mode, do the work rather than proposing. Still stop at the three
  product checkpoints: Plan, first chapter acceptance, and Audio.

## Tool Mapping

| Original wording | Codex behavior |
|---|---|
| Agent Teams / reviewer agent | Prefer Codex subagents if available; otherwise do the checklist inline and say so briefly. |
| subAgent parallel development | Use Codex multi-agent/subagent tools only after the first chapter is accepted. If unavailable, use mode B sequential development. |
| AskQuestion / choice cards | In Plan mode use `request_user_input`; otherwise ask concise numbered questions in chat and wait. |
| WebSearch | Browse only when current facts, products, tools, or asset sources may have changed. For pure local work, inspect files first. |
| Open browser / visual QA | Use the in-app browser skill or Playwright where available. For local Vite apps, start the dev server and give the URL. |
| Shell edits | Use `apply_patch` for hand edits. Use shell only for scaffolding, installs, format/build/test, and generated outputs. |

## PowerShell / Windows Notes

The bundled scaffold is Bash. On Windows, try Git Bash first:

```powershell
bash .\garden-skills\skills\web-video-presentation\scripts\scaffold.sh .\presentation --theme=paper-press
```

If `bash` is unavailable, create the Vite project manually and copy the template
files by reading `scripts/scaffold.sh` as the source of truth. Do not rewrite the
template logic from memory.

When deleting the scaffold demo on Windows, use PowerShell end-to-end:

```powershell
Remove-Item -LiteralPath .\presentation\src\chapters\01-example -Recurse -Force
```

Then patch `presentation/src/registry/chapters.ts` to remove the example import
and array item. Verify absolute paths before any recursive delete.

## Codex Checkpoint Policy

### Checkpoint Plan

After writing `article.md` if needed, `script.md`, and `outline.md`:

1. Run the `SCRIPT-STYLE.md` and `OUTLINE-FORMAT.md` checks.
2. Read all `themes/*/theme.json` dynamically and recommend 2-3 themes.
3. Ask for the five decisions in one compact message:
   script edits, outline edits, theme, asset plan, and development mode.
4. Do not scaffold until the theme and mode are clear.

Safe defaults only when the user explicitly asks Codex to choose:

- Theme: choose the strongest match from `bestFor`, explain the choice.
- Assets: placeholders unless real local assets are provided.
- Mode: A, chapter-by-chapter acceptance.

### First Chapter Checkpoint

Build chapter 1 to finished quality. Start the dev server and visually inspect
it before presenting it. Then stop and ask for acceptance. Do not proceed to
chapter 2 until the user says to continue, unless they explicitly requested a
single autonomous build and accepted the risks.

### Audio Checkpoint

After all chapters are complete, ask whether to synthesize audio. Do not require
TTS credentials just to finish the web presentation. If the user chooses audio,
read `references/AUDIO.md` and `templates/scripts/tts-providers/README.md`.

## Implementation Guardrails

- Each chapter folder owns exactly one `.tsx`, one `.css`, and one
  `narrations.ts`.
- `narrations.length` is the chapter step count. The maximum `step === N` in the
  component must be `narrations.length - 1`.
- Register chapters in `src/registry/chapters.ts` only after their files exist.
- Bump `src/hooks/useStepper.ts` `STORAGE_KEY` after chapter order or step counts
  change.
- Keep CSS class prefixes chapter-local to avoid cross-chapter leaks.
- Use theme tokens for color and font families. Hard-code layout geometry only
  when it expresses the chapter's visual idea.
- No pure-text chapters. Every chapter needs CSS/SVG/canvas/JS visual action.
- Lists reveal one item per step. Do not show a full list at once with staggered
  animation.

## Verification Checklist

Run what is available and proportionate:

```bash
cd presentation
npm run build
# or at minimum:
npx tsc --noEmit
```

For visual work, inspect at least:

- First chapter after it is built.
- Final full deck after all chapters are registered.
- `?auto=1` if audio has been synthesized, or if silent fallback timing matters.

Report any skipped checks honestly.
