# Kiroween 2026 · DEAD AIR: competition prep

Prepared **2026-10-31** with Claude Code, outside Kiro, as planning input for [JaredVogler2/Kiroween2026](https://github.com/JaredVogler2/Kiroween2026) at commit `8b1d7fe`. **Nothing in this folder is evidence of Kiro use.** It lives in this prompt library rather than in the competition repo on purpose: everything committed to the judged repo from now on should come from Kiro sessions.

| File | What it is |
|---|---|
| `README.md` (this file) | Rules digest, project assessment, ranked enhancements, schedule, guardrails, submission checklist |
| [`KIRO_PROMPTS.md`](KIRO_PROMPTS.md) | 15 sequenced copy-paste prompts (P0–P14) that make Kiro do the work |

---

## 1. Rules and judging: what we know

Devpost blocks automated access, so the full text of [kiroween.devpost.com/rules](https://kiroween.devpost.com/rules) couldn't be read. The table below separates confirmed facts from inferred ones. **Prompt P1 asks you to paste the official criteria into a Kiro steering file**, so Kiro works from the real text rather than this summary.

| Topic | What we know | Source / confidence |
|---|---|---|
| Deadline | **Fri Dec 4, 2026, 2:00 PM PT** | Your screenshot ✔ |
| Eligibility | Only projects created during the submission period, which opened Oct 30. The repo's first commit is Oct 31, 02:13 ET ✔ | Screenshot ✔ · git log |
| Categories | Resurrection, Frankenstein, Skeleton Crew, Costume Contest. One category per entry | Screenshot ✔ · 2025 rule |
| Judging | **Stage 1** is pass/fail: fits a category, uses Kiro, meets the requirements. **Stage 2** scores equally weighted criteria; the rules text asks whether **a variety of Kiro tools were used impactfully**. The 2025 criteria were **Potential Value** (useful, accessible, impactful), **Implementation** (how effectively Kiro was used) and **Quality & Design** (creativity, originality, polish) | Rules-page snippets · [Kiro 2025 blog](https://kiro.dev/blog/kiroween-2025/) ✔. A third-party summary claims five criteria at 20 % each (unverified) |
| Category prizes | The highest-scoring eligible entry in each category wins, using the same criteria | Rules-page snippet |
| Ghost of Kiroween Past | $2,000 for one winner. Open to entrants who **registered for Kiroween 2025** (Devpost verifies) | Rules-page snippet |
| Blog-post bonus | A public post on a third-party platform, published during the submission period, stating it was written for Kiroween 2026. Goes to the top 100 scoring eligible entries; prize is 2,000 Kiro credits | Rules-page snippet |
| Required | Public repo with an OSI license shown in the repo's About panel. `/.kiro` at the repo root, not gitignored. A working app URL that stays free to test through judging. A public YouTube video **under 3 minutes**. A text description that includes how Kiro was used. English. Teams of up to 3. Pre-existing code disclosed | 2025 rules · third-party 2026 summary |
| Judging window | Dec 16, 2026 to Jan 29, 2027. **Keep the app live until then** | Rules-page snippet |

### What this means for DEAD AIR

1. **Kiro usage is currently zero, and that is the biggest scoring risk.** Every file, including all of `.kiro/`, was drafted with ChatGPT, and the README says so, as it should. Don't hide that. Instead, have Kiro do most of the remaining engineering in a way judges can see: Kiro-native specs (including a bugfix spec), hooks that actually fire, MCP calls that drive decisions, agents and workflows that actually run, and an evidence log written as the work happens. **Target:** by Dec 4, most shipped code traces back to Kiro tasks made after a `pre-kiro-baseline` tag.
2. **"Variety of Kiro tools, used impactfully."** The plan uses about 12 Kiro features. Each one is tied to a real outcome, such as a bug fixed, a measured speed-up or an automated evidence trail, rather than added for show.
3. **Potential Value** needs proof that real phones decode the signal, plus real-world uses: shelters during outages, classrooms and events without Wi-Fi, text versions of PA announcements for Deaf and hard-of-hearing visitors, and privacy (no accounts, no servers).
4. **Quality & Design:** the concept boards set the bar. Add live visuals driven by the real signal, and give judges a way to verify the app in under 60 seconds on one device.
5. **Resurrection fit:** spell out the lineage. Bell 103 modems sent data as audio tones (1962); then came acoustic couplers, then Kansas City Standard cassette data (1975), and now DEAD AIR.

---

## 2. Where the project stands (commit `8b1d7fe`)

**Strengths**
- The concept is clear and fits Resurrection well. Its truthfulness rules (no fake receipts, no LLM, no invented receiver counts) set it apart.
- The FSK modem has no dependencies: Hamming(7,4), CRC-16, plus audible and near-ultrasonic profiles. AudioWorklet capture runs with echo cancellation, noise suppression and AGC **off**, which is correct for modems. Received text renders via `textContent`. There is an offline PWA cache and a Vercel config.
- The `.kiro/` scaffold already uses current Kiro formats: steering (including `auto`), v1 JSON hooks, custom agents, `*.workflow.json` recipes, a skill, and a working stdio MCP server.
- All 21 tests pass, and the benchmark and CI run.

**Verified issues.** These were reproduced with throwaway scripts outside the repo; nothing in the repo was changed.

| # | Finding | Evidence | Why it matters |
|---|---|---|---|
| V1 | **The near-ultrasonic decoder isn't frequency-selective.** `AcousticDecoder.readBit()` runs Goertzel on every 4th sample (`stride=4`) with no anti-alias filter, so it detects 18.2/19.2 kHz only through their *aliases* | A frame sent entirely with **audible** tones decoded on the ultrasonic decoder in 3 of 3 cases: 4.8/5.8 kHz and 6.2/7.2 kHz at 48 kHz, and 2.85/3.85 kHz at 44.1 kHz | Speech and room noise at 2.8–7.2 kHz leak into the "ultrasonic" channel, with a theoretical SNR loss of about 6 dB. Until this is fixed, a result like "near-ultrasonic works on phone X" proves nothing |
| V2 | **The hero line from the concept boards doesn't fit** | "IF YOU CAN READ THIS, YOU ARE NOT ALONE." is 40 characters, but the textarea has `maxlength=36`. Airtime would be 23.4 s; the 17-character default already takes 13.1 s | The signature demo moment is impossible as built, and long airtime eats into a 3-minute video |
| V3 | **Reverb has never been tested** | A crude synthetic room probe (single seed): both profiles decode at RT60 0.15–0.3 s and **fail at 0.6 s**, which is typical of a living room or classroom. Clock offsets up to 1000 ppm decode; 3000 ppm fails. Repo tests add only white noise, and the benchmark covers only the audible profile | Real rooms may fail while CI stays green |
| V4 | **No physical-device result exists** | Every trial in `docs/DEVICE_TEST_MATRIX.md` is NOT RUN | The core claim is unproven. Test on real phones in week 1 |
| V5 | **Phones can auto-lock mid-message** | There is no Screen Wake Lock and no visibility handling | Reception fails without any visible error during demos and judging |
| V6 | **The service-worker cache version is set by hand** (`dead-air-v0.5.0`) | `service-worker.js` | Judges' phones can keep serving a stale offline build |
| V7 | **`index.html` duplicates `broadcast.html`**, identical apart from `<title>`. The footer has a typo, "KIROW EEN" | diff | The two copies drift apart, and the typo looks careless |
| V8 | **The concept boards over-claim:** "OFFLINE MODE ✓ (No Wi-Fi, No Cellular, No Bluetooth)" and "Near-Ultrasonic (**Silent**)" | A web page can't check radio state, and 18 kHz isn't silent to everyone | Keep the look but change the words: "BROWSER OFFLINE" plus a self-attested radio checklist, and "Quiet · experimental" |
| V9 | **Some `.kiro/` details need confirming inside Kiro.** The hook docs' example uses `PostFileSave`, but the trigger reference lists "File Save". Agents load the `acoustic-evidence` skill only if `skill://acoustic-evidence` is in their `resources`. Human checklists sit in `.kiro/workflows/`, where only `*.workflow.json/yaml` files count as recipes | [kiro.dev docs](https://kiro.dev/docs/hooks/) | A config that doesn't load does nothing |

---

## 3. Enhancement plan (ranked)

| Priority | Enhancement | Criteria it lifts | Kiro features shown | Prompt |
|---|---|---|---|---|
| MUST | Kiro takeover: baseline tag, prove every `.kiro` artifact loads in Kiro, first real evidence | Stage 1 · Implementation | Agent chat, MCP, `validate_workflow`, workflow run | P0 |
| MUST | Competition, DSP and UI steering. Hooks for evidence logging, cache-version bumps and claim linting. An orchestrator agent | Implementation | Steering (always/auto/fileMatch/manual), hooks (PostTaskExecution, file save), sub-agents | P1 |
| MUST | Fix V1, writing the failing test first | Quality · Implementation | **Bugfix spec**, property-based tests, parallel sub-agents | P2 |
| MUST | A realistic channel lab (reverb, clock drift, band limits, AGC, speech interference), parameter sweeps, and a benchmark covering both profiles | Implementation · Quality | **MCP** (custom server v2), Quick Spec | P3 |
| MUST | Receiver resilience: wake lock, audio resume, mic-permission and track-settings diagnostics | Quality | **Vibe coding** | P4 |
| MUST | Field-trial loop: local diagnostic WAV → `analyze_recording` MCP tool → device matrix | Value · Implementation | Feature spec, MCP, **Agent Skill** | P5 |
| HIGH | Modem v2 "FAST": compact header, text packing, M-ary FSK or shorter symbols with guard time, stronger FEC, better sync. Goal: the hero line in ≤ 8 s | Value · Quality | Design-first spec, **workflow `repeat` loop**, checkpoints | P6 |
| HIGH | Receiver theater: a real state machine (detected → locked → decoding % → verified/corrupted) plus a live spectrogram | Quality & Design | Feature spec, custom agent | P7 |
| MED | Calibration wizard ("Spirit Sweep" in the Veil skin), built on real measurements | Quality · Value | Feature spec, MCP | P8 |
| HIGH | Landing page and console redesigned to match the concept boards. Truthful copy. One page instead of two copies | Quality & Design | Feature spec, vibe polish, browser-automation screenshots | P9 |
| MUST | **Judge ladder:** decode a real field recording (no mic) → single-device loopback → full room | Value · Quality | Feature spec | P10 |
| MED | Beacon mode, quick phrases, phrase codebook | Value | Vibe coding | P11 |
| STRETCH | Survivor Roll Call (acoustic ACKs) and Tape mode (WAV export and decode) | Value · Resurrection | Feature specs | P12 |
| MED | Release gate as a parallel workflow, plus real browser E2E tests using a fake-mic WAV | Implementation · Quality | **Workflow `parallel`**, orchestrator agent | P13 |
| MUST | Submission kit: README, Devpost write-up, video shot list, blog post | All | Skill, agent | P14 |

**Minimum path if time runs short:** P0 → P1 → P2 → P4 → P5 → P7 → P9 → P10 → P14.

---

## 4. Schedule (Sat Oct 31 → Fri Dec 4)

| Week | Dates | Work |
|---|---|---|
| 1 | Nov 1–7 | P0, P1, P2, P4. Deploy over HTTPS. First real laptop→phone audible trial (P-01) |
| 2 | Nov 8–14 | P3, P5. Run trials P-01 to P-04. Decide whether near-ultrasonic stays a headline feature or becomes "experimental R&D" |
| 3 | Nov 15–21 | P6, P7 |
| 4 | Nov 22–28 (Thanksgiving is Thu Nov 26) | P9, P10, then P8 and P11 if time allows. Capture the real field clip for the judge ladder |
| 5 | Nov 29–Dec 4 | **Feature freeze Mon Nov 30.** P13, P14. Record the video, publish the blog, and **submit Thu Dec 3** to keep a day of buffer |

---

## 5. Kiro feature coverage (for the write-up)

| Kiro feature | Used in | What judges can inspect |
|---|---|---|
| Feature specs (requirements-first and design-first) | P5–P10, P12 | New `.kiro/specs/*` folders with task status |
| Bugfix spec | P2 | `bugfix.md` (current / expected / unchanged behavior) plus property tests that go from failing to passing |
| Correctness / property-based tests (IDE) | P2, P6 | The generated property-based tests |
| Steering: always · auto · fileMatch · manual | P1 | `.kiro/steering/` |
| Hooks: file save, PostTaskExecution | P1 | `.kiro/hooks/`, plus evidence-log entries the hook wrote |
| MCP: custom `acoustic-lab` v2 | P0, P3, P5 | Tool calls and result tables quoted in the evidence log |
| MCP / Power: browser automation (if available) | P9, P13 | Screenshot QA loop |
| Custom agents and parallel sub-agents | P1, P2, P6, P13 | `.kiro/agents/`, including the orchestrator |
| Workflows: step · repeat · parallel, checked with `validate_workflow` | P0, P6, P13 | `.kiro/workflows/` plus run outputs |
| Agent Skills | P5, P14 | `.kiro/skills/field-trial` and `devpost-writeup` |
| Vibe coding | P4, P11, polish in P9 | Transcript excerpts in the write-up |
| Checkpoints / rewind | P6 | The story of a dead-end experiment |
| Kiro Web cloud session (optional) | P3 and P6 sweeps | Session link |

---

## 6. Guardrails

- **Keep the truth rules:** no fake receipts, receiver counts, Kiro evidence or device results. Mention them in the write-up, because they are a strength.
- **Disclose the ChatGPT-drafted baseline once, plainly** (tag plus commit) in the README and on Devpost, then let the Kiro work speak for itself. Don't scatter disclaimers through the UI.
- **Never use the real EAS/EBS attention tones (853 + 960 Hz)** for the post-apocalyptic mood. FCC rules (47 CFR §11.45) prohibit transmitting the Attention Signal, or a simulation of it, outside real alerts and authorized tests, and the tones alarm viewers. Design an original attention chime instead.
- **Video and blog:** no copyrighted music. Keep third-party logos out of frame where you can. Caption the radios-off state on screen.
- **Hearing safety:** use moderate volume and keep speakers away from ears. Near-ultrasonic can still be audible, especially to children and pets.
- **Privacy:** mic audio never leaves the device. Diagnostic clips are local downloads the user starts. Don't commit clips that contain identifiable voices.

---

## 7. Submission checklist (target Thu Dec 3)

- [ ] Repo **public**, MIT license shown in the About panel, `/.kiro` committed at the root
- [ ] HTTPS deploy, tested on iOS Safari and Android Chrome, **live and free through Jan 29, 2027**
- [ ] Video under 3:00, public on YouTube, with real devices in the same frame as their radios-off settings and a Kiro segment
- [ ] Devpost: category **Resurrection**; features; a Kiro write-up for each feature with concrete examples and numbers; testing instructions (the judge ladder); provenance disclosure
- [ ] Blog post published on a third-party platform, mentioning Kiroween 2026, with its URL in the form
- [ ] **Ghost of Kiroween Past:** if you registered for Kiroween 2025, make sure the entry claims it
- [ ] Teammates (up to 3) added on Devpost
- [ ] Release-gate workflow green; evidence log and device matrix up to date

---

## 8. How to run the prompt pack

1. Open `Kiroween2026` in **Kiro IDE 1.x** (Correctness/PBT is IDE-only). Trust the workspace, because project agents load only in trusted workspaces.
2. Run the prompts in order.
   - **Spec** prompts: start a new Spec (Feature or Bug Fix), paste the prompt as the description, review the requirements and design gates, then run the tasks.
   - **Vibe** prompts: paste into chat.
   - **Agent/Workflow** prompts: paste into chat, and Kiro runs the named agents and recipes.
3. Attach the two concept boards to the chat for P7–P10.
4. Use one branch and one PR per prompt (for example `kiro/p2-ultrasonic-selectivity`). Add the commit trailer `Kiro-Prompt: P2` so provenance can be audited.
5. After each prompt, check that the hook wrote its evidence-log entry. Add the human-only parts yourself, such as device trials and judgment calls.

## Sources

- [kiroween.devpost.com/rules](https://kiroween.devpost.com/rules) (search snippets only; direct access is blocked) · [Kiro: "This is Kiroween" (2025)](https://kiro.dev/blog/kiroween-2025/) · [Kiroween 2025 on Devpost](https://kiroween-2025.devpost.com/) · [Reconscribe 2026 guide](https://reconscribe.com/kiroween-2026-hackathon/) (written before the rules were posted; partly outdated)
- Kiro docs: [hooks](https://kiro.dev/docs/hooks/) · [hook triggers](https://kiro.dev/docs/hooks/types/) · [steering](https://kiro.dev/docs/steering/) · [specs](https://kiro.dev/docs/specs/) · [bugfix specs](https://kiro.dev/docs/specs/bugfix-specs/) · [custom agents](https://kiro.dev/docs/custom-agents/) · [sub-agents](https://kiro.dev/docs/custom-agents/subagents/) · [workflows](https://kiro.dev/docs/workflows/authoring/) · [skills](https://kiro.dev/docs/skills/) · [powers](https://kiro.dev/docs/powers/)
