# DEAD AIR: Kiro prompt pack (P0–P14)

These prompts are for the Kiroween 2026 entry. Paste each fenced block into Kiro. For the rationale, the rules digest and the verified issues (V1–V9), see [README.md](README.md).

Each prompt is labeled with four fields:

- **Mode:** how to start it in Kiro.
- **When:** the schedule week.
- **Priority:** MUST, HIGH, MED or STRETCH.
- **Done when:** the check that it worked.

From P2 on, the prompts assume the steering and hooks created in P1 exist, so they don't restate the project's truth rules.

**Re-anchor snippet.** Paste this whenever Kiro drifts:

```
Re-read .kiro/steering/competition.md and verification.md. Which judging criterion does this change serve, what evidence will prove it, and is anything you just wrote a claim we cannot support?
```

---

## P0: Kiro takeover and audit

**Mode:** chat, Kiro IDE · **When:** Week 1 · **Priority:** MUST

```
You are taking over DEAD AIR, a Kiroween 2026 "Resurrection" entry. A laptop speaker broadcasts a short human-written message as FSK audio, and any number of offline phones decode it from their microphones. Everything in this repo up to commit 8b1d7fe, including every .kiro file, was drafted with ChatGPT outside Kiro and is NOT evidence of Kiro use. From now on the real work happens here, in Kiro, and must leave a truthful trail.

Work in order. Stop and show me the results after step 4.

1. Provenance baseline. Create branch kiro/p0-takeover. Add an annotated tag pre-kiro-baseline on 8b1d7fe with the message "Last commit drafted outside Kiro (ChatGPT). All later work: Kiro sessions." Do not rewrite history.
2. Run `npm run check` and `npm run benchmark` and keep the exact output.
3. Check that every Kiro artifact actually loads. Record the results in docs/KIRO_AUDIT.md as a table: artifact | loads in Kiro? | problem | fix.
   - Steering (.kiro/steering/*.md): are the inclusion modes honored?
   - Hooks (.kiro/hooks/*.json): create ONE throwaway file-save hook through Kiro's own hook creation flow. Compare the JSON Kiro generates, especially the trigger value (PostFileSave vs FileSave), with our three hooks, and fix ours to match. Delete the throwaway.
   - Agents (.kiro/agents/*.json): is each one selectable? Add "skill://acoustic-evidence" to the resources of signal-scientist and failure-hunter, because agents don't load skills otherwise.
   - MCP (.kiro/settings/mcp.json): confirm acoustic-lab connects. Call inspect_frame and simulate_channel for BOTH profile=audible and profile=ultrasonic, and paste the JSON results into the audit.
   - Skill (.kiro/skills/acoustic-evidence): can it be found via /acoustic-evidence?
   - Workflows (.kiro/workflows/*.workflow.json): run validate_workflow on each and fix every reported error. Move the two human checklists (01-first-kiro-session.md, 02-finish-prize-entry.md) to docs/checklists/, because only *.workflow.json/yaml files are recipes.
   - Specs (.kiro/specs/*): put "LEGACY DRAFT (pre-Kiro)" at the top of each existing spec. New work will get Kiro-native specs.
4. Run the broadcast-reliability workflow with the brief "baseline audit only, no code changes". Summarize its findings.
5. Append the first real entries to docs/KIRO_EVIDENCE_LOG.md: date, Kiro surface and version, what ran, its output, and the commit. Update scripts/validate-config.mjs if your fixes change what it should check. Run `npm run check` again.
6. Commit with the trailer "Kiro-Prompt: P0", push the branch and the tag, and open a PR.

Follow .kiro/steering: no fabricated results. Do not change modem behavior in this prompt.
```

**Done when:** the audit table is complete; hooks, agents, skill and workflows load or validate in Kiro; MCP results for both profiles are logged; the tag is pushed.

---

## P1: Competition steering, evidence hooks, orchestrator agent

**Mode:** chat (steering and hooks) · **When:** Week 1 · **Priority:** MUST

**Before you run it:** copy the *Judging Criteria*, *Submission Requirements* and *Prizes / Bonus* sections from kiroween.devpost.com/rules and paste them where marked.

```
Set up steering and automation so every later Kiro session optimizes for the competition and documents its own work.

A. Steering. Use all four inclusion modes on purpose.
1. .kiro/steering/competition.md
   - Front matter: inclusion: auto; name: competition; description: "Kiroween 2026 rules, judging criteria, submission checklist and category framing. Use when prioritizing work or writing README, Devpost, video or blog content."
   - Body: the official text pasted below, verbatim, with a source line and retrieval date.
   - Then this checklist:
     - category: Resurrection
     - deadline: Fri Dec 4 2026, 2:00 PM PT (target Thu Dec 3)
     - public repo with MIT shown in the About panel; /.kiro committed
     - live HTTPS URL, free to use through Jan 29 2027
     - YouTube video under 3:00
     - Kiro write-up covering specs, hooks, steering, MCP, vibe coding, custom agents, workflows and skills
     - blog-post bonus; Ghost of Kiroween Past bonus if eligible
   - Close with: "Every feature must map to at least one judging criterion. Prefer work a judge can see or verify."
   <<<PASTE OFFICIAL RULES SECTIONS HERE>>>
2. .kiro/steering/dsp.md
   - Front matter: inclusion: fileMatch; fileMatchPattern: ["src/modem*.mjs","src/audio-worklet.js","src/dsp/**","mcp/**","scripts/benchmark.mjs","tests/*modem*.mjs","tests/profiles*.mjs"]
   - Rules:
     - Never decimate without band-limiting first.
     - Test both profiles at 44.1 and 48 kHz.
     - Every DSP change reports before/after benchmark and acoustic-lab results for both profiles.
     - State the CPU cost per second of audio.
     - Version wire-format changes and keep them backward compatible.
3. .kiro/steering/ui-design.md
   - Front matter: inclusion: fileMatch; fileMatchPattern: ["**/*.html","**/*.css","src/broadcast.mjs","src/ui/**"]
   - Rules:
     - Themes are CSS custom-property token sets. outbreak: near-black, burnt amber/orange call-to-action, phosphor-green receiver. veil: deep violet, lavender glow, teal accents.
     - WCAG AA contrast; touch targets of 44px or more; prefers-reduced-motion fallbacks.
     - Uppercase via CSS text-transform, not literal capitals in markup.
     - Every visual about signal or reception must come from real microphone data or real decoder state.
     - Banned copy: "silent", "inaudible", "offline mode ✓", "no Wi-Fi/cellular/Bluetooth ✓", and receiver counts or "delivered" claims on the Tower.
4. .kiro/steering/release-checklist.md
   - Front matter: inclusion: manual, so it is invoked as #release-checklist.
   - Content: the pre-merge and pre-submission gates.

B. Hooks. Create each through Kiro's hook flow so the JSON is valid, then commit.
1. Evidence logger.
   - Trigger: PostTaskExecution (a spec task finished). Action type: agent.
   - Prompt: "Append one entry to docs/KIRO_EVIDENCE_LOG.md: UTC time, spec and task, requirement IDs, files changed (git diff --stat), tests and benchmarks run with pass/fail counts, and Kiro vs human contribution. Facts only; if nothing was tested, say so."
2. Cache version.
   - Trigger: file save, matching any asset listed in CORE in service-worker.js except service-worker.js itself.
   - Command: node scripts/sw-version.mjs
   - Write that script: hash the CORE assets and rewrite the CACHE constant, so judges never get a stale offline build.
3. Claims lint.
   - Trigger: file save on README.md, docs/**/*.md and *.html.
   - Command: node scripts/lint-claims.mjs
   - Write that script. It fails on the banned copy above, on "PHYSICALLY-VERIFIED" without a link, and on "KIRO-EXECUTED" without an evidence-log reference.
   - Allow explicit negations ("not silent") and an inline claims-ok marker, so docs can still discuss limitations.
   - Add a unit test for it.
4. Keep the existing test-on-save hooks working.

C. Agent. Create .kiro/agents/release-captain.json, an orchestrator.
   - tools: include "subagent".
   - resources: all steering files plus docs/KIRO_EVIDENCE_LOG.md.
   - prompt: delegate in parallel (DSP checks to signal-scientist, adversarial checks to failure-hunter, UX to design-director), then merge the results into a PASS / FAIL / UNVERIFIED table.

Make each hook fire once with a real edit and record its output in the evidence log. Run npm run check. Commit with the trailer "Kiro-Prompt: P1" and open a PR.
```

**Done when:** four steering files cover four inclusion modes; three new hooks have each fired once with the output logged; release-captain is selectable.

---

## P2: Bugfix spec: the near-ultrasonic decoder isn't frequency-selective (V1)

**Mode:** Spec → **Bug Fix** (from the "Let's build" screen, or the agent picker → Bug Fix) · **When:** Week 1 · **Priority:** MUST

```
Bug: the near-ultrasonic decoder accepts frames carried entirely by AUDIBLE tones.

Reproduce it yourself first; the lead comes from a pre-Kiro review.
- AcousticDecoder.readBit() in src/modem.mjs runs Goertzel on every 4th sample (stride=4, fs=sampleRate/4) with no anti-alias filter.
- As a result, 18.2/19.2 kHz fold to 5.8/4.8 kHz at 48 kHz, and to 3.85/2.85 kHz at 44.1 kHz.
- Repro: render a frame as plain FSK with zero=5800 Hz / one=4800 Hz at 48 kHz (or 6200/7200 Hz at 48 kHz, or 3850/2850 Hz at 44.1 kHz). Feed it to new AcousticDecoder(sr, cb, 'ultrasonic'). It decodes the packet.
- The same folding lets speech and room noise from 2.8–7.2 kHz into the "ultrasonic" channel. Using only every 4th sample also costs about 6 dB of SNR in theory.

Current behavior (defect):
- WHEN audible-band FSK at an alias pair is captured, THEN the ultrasonic decoder emits a CRC-valid packet.

Expected behavior:
- The ultrasonic decoder SHALL respond only to energy near 18.2/19.2 kHz and SHALL emit zero packets for audible alias frames.
- The audible decoder SHALL likewise ignore out-of-band energy.

Unchanged behavior:
- Both profiles SHALL CONTINUE TO round-trip at 44.1 and 48 kHz.
- The six-receiver test, the CRC/Hamming tests and the MCP tests SHALL CONTINUE TO pass.
- The AcousticDecoder(sampleRate, onPacket, profile) API and the wire format SHALL NOT change.
- CPU per second of audio SHALL stay within 2x of today. Measure it.

Design: evaluate these with measurements, not assumptions:
- a full-rate Goertzel for the ultrasonic profile;
- a complex heterodyne to baseband, then low-pass and decimate;
- a biquad band-pass before decimation.

Process:
- Run signal-scientist and failure-hunter as parallel sub-agents. One implements; the other attacks with wrong-carrier, speech-like interference, mixed-profile, and 44.1/48 kHz cases.
- Use the acoustic-lab MCP for before/after results on both profiles.
- Add a noise sweep that compares success rates before and after. Report the actual numbers.
- Property tests:
  (1) the bug reproduces on the current code;
  (2) after the fix, for random messages and every alias pair, the ultrasonic decoder returns no packets;
  (3) genuine ultrasonic frames still decode across a noise range.

Commit trailer: Kiro-Prompt: P2.
```

**Done when:** `bugfix.md`, `design.md` and tasks exist; the alias property test failed before the fix and passes after; before/after sweep numbers for both profiles are in the evidence log.

---

## P3: acoustic-lab MCP v2: realistic rooms and sweeps

**Mode:** Spec → Feature (Quick Spec is fine) · **When:** Weeks 1–2 · **Priority:** MUST

```
Our synthetic tests only add white noise, so a green CI says little about real rooms. A crude prep probe (one seed) suggested both profiles fail at about RT60 0.6 s reverb and beyond about 1000 ppm clock offset. Verify this properly.

Extend mcp/acoustic-lab.mjs. Keep it dependency-free, stdio JSON-RPC, seeded and compute-bounded.

1. Add optional impairments to simulate_channel:
   - rt60: 0–1.2 s, using a seeded exponential-decay room impulse response with early reflections.
   - clockPpm: ±3000, via fractional resampling.
   - crossRate: render at 44.1 kHz and decode at 48 kHz, and the reverse.
   - lowpassHz / highpassHz: speaker and microphone band limits.
   - agc: a simple compressor.
   - interference: none | speech-like | hum | babble.
   - snrDb: replaces the raw noise amplitude; keep noise for compatibility.
   - startOffset: random sub-sample alignment.
2. New tool sweep:
   - Grid over profiles × one or two impairment axes × seeds, with a cap on total trials.
   - Returns the success rate with Wilson 95% intervals, the median FEC corrections and the airtime.
   - Labels results SYNTHETIC_ONLY.
3. New tool compare: run the same sweep against two git refs or two decoder options (for example before and after P2), so an improvement can be proven.
4. Update scripts/benchmark.mjs so it covers both profiles plus a default room preset, and writes docs/BENCHMARKS.md (a generated table that includes the commit SHA).
5. Add autoApprove entries for the new read-only tools in .kiro/settings/mcp.json, and extend tests/mcp.test.mjs to cover argument validation and determinism with a fixed seed.

Then call sweep from this session for both profiles with rt60 ∈ {0, 0.3, 0.6, 0.9} and clockPpm ∈ {0, 300, 1000}. Paste the tables into the evidence log, and list the worst failure modes as input for the modem v2 spec (P6). Optionally run the sweep in a Kiro Web cloud session and link it.
```

**Done when:** Kiro has called `sweep` from chat, with the tables logged; `docs/BENCHMARKS.md` is generated; MCP tests are green.

---

## P4: Receiver resilience (vibe coding)

**Mode:** Vibe (chat, no spec). Note in the write-up that this part was vibe-coded. · **When:** Week 1 · **Priority:** MUST

```
Quick fixes for real phones in src/broadcast.mjs, plus shared receiver code if you extract any. No spec needed.

1. Request a Screen Wake Lock while listening. Re-acquire it on visibilitychange and release it on stop. If the API is unsupported, show "KEEP SCREEN ON".
2. If the page is hidden, or the AudioContext becomes suspended or interrupted (iOS), show "LISTENING PAUSED". Resume when the user returns, calling context.resume() from a user gesture if required.
3. After getUserMedia, read track.getSettings(). Show sampleRate, echoCancellation, noiseSuppression and autoGainControl in an expandable Diagnostics panel. Warn loudly if noise suppression or echo cancellation stayed ON, because they erase FSK tones.
4. Map getUserMedia errors (NotAllowedError, NotFoundError, NotReadableError, OverconstrainedError, SecurityError) to plain-language recovery steps for iOS Safari and Android Chrome.
5. Keep the status words honest: LISTENING, PAUSED, STOPPED.

Unit-test any pure logic you extract. Commit trailer: Kiro-Prompt: P4.
```

**Done when:** on a real phone the screen stays awake while listening, and the Diagnostics panel shows the actual track settings.

---

## P5: Field-trial loop (diagnostic clips → MCP → device matrix)

**Mode:** Spec → Feature · **When:** Week 2 · **Priority:** MUST

```
Create a feature spec "field-trial-loop" so physical results flow back into Kiro as evidence.

Requirements to cover:

1. Receiver "Save diagnostic clip".
   - The user starts it. It saves up to 30 s of rolling capture of the raw mic PCM the decoders saw.
   - Export as a local 16-bit WAV download with a JSON sidecar: UTC time, user agent, AudioContext sampleRate, track settings, active profiles, and decoder events.
   - Nothing is uploaded. The UI copy says clips may contain room audio.
2. New MCP tool analyze_recording({path}).
   - Decodes a WAV with both profiles.
   - Reports packets, CRC and FEC results; per-symbol tone power ratio and an SNR estimate; estimated clock offset (ppm) from preamble timing; spectrum peaks; band energy from 1–20 kHz.
   - Labels results FIELD_RECORDING (physical evidence), distinct from SYNTHETIC.
   - Only accepts paths under field-recordings/.
3. New skill .kiro/skills/field-trial/SKILL.md (name: field-trial).
   - The protocol for trials P-01 to P-08 in docs/DEVICE_TEST_MATRIX.md: setup photo; proof radios are off; distance, volume and orientation; success AND failure recorded for each phone; one clip saved per failure; analyze_recording run on it; result appended to the matrix.
   - Add skill://field-trial to the resources of signal-scientist and failure-hunter.
4. field-recordings/README.md: consent and privacy rules. Don't commit clips with identifiable voices; keep clips to 30 s or less.

After implementation I will run P-01 to P-04 on real devices. Then run analyze_recording on every failed clip and propose decoder changes ranked by expected impact. Do not implement them yet; they feed P6.
```

**Done when:** at least four physical trials are in the matrix, failures included, each failure with a clip analysis; ranked proposals are logged.

---

## P6: Modem v2 "FAST" profile

**Mode:** Spec → Feature (**Design-First**), plus a workflow · **When:** Weeks 2–3 · **Priority:** HIGH

```
Create a Design-First feature spec "modem-v2-fast".

Goal: the concept-board line "IF YOU CAN READ THIS, YOU ARE NOT ALONE." (40 characters; today 23.4 s and over the 36-character UI cap) airs in 8 s or less, stretch 5 s or less, on the audible band. Add a near-ultrasonic variant. Don't lose reliability.

Targets, measured with the P3 acoustic-lab sweep and the P5 field results:
- 95% or better synthetic success for both profiles at RT60 ≤ 0.6 s, ±300 ppm clock offset, and SNR ≥ 10 dB.
- No regression against v1 in physical trials.

Options to evaluate by measurement. Pick and justify.
- A compact header, for example 4 bytes: version/flags, a 12–16-bit ID, and length.
- 6-bit text packing for the uppercase character set, falling back to UTF-8.
- M-ary FSK (4, 8 or 16 orthogonally spaced tones), or shorter symbols with a guard interval against reverb tails.
- Alternating tone sets between adjacent symbols.
- Bit interleaving plus Reed–Solomon over GF(256), or convolutional coding with soft decisions.
- A Barker-13 or chirp sync instead of the 1010… preamble, whose two-symbol shifts look identical.
- Soft-decision demodulation from tone power ratios.

Constraints:
- Keep v1 as the ROBUST profile. v2 frames carry a new version number so receivers can run v1 and v2 decoders side by side.
- Stay dependency-free.
- If CPU use rises, move DSP off the main thread (AudioWorklet or a Worker with a shared ring buffer).
- Raise the UI cap to 64 characters only if the airtime targets hold.
- Show a live airtime estimate in the transmitter UI.

Process:
- Write .kiro/workflows/modem-optimize.workflow.json with a repeat node (maxIterations: 3; onMaxIterations: stop and report). In each loop, signal-scientist proposes and implements one variant, then failure-hunter runs the sweep and attacks it. Loop until the targets are met.
- Run validate_workflow first, then run the workflow.
- Use checkpoints to rewind dead ends, and note them in the evidence log.
- Write property-based tests for codec round-trips: random payloads, plus random bit-error patterns within the code's correction capability.
```

**Done when:** a measured airtime table compares v1 and v2 for both profiles; the workflow has run with its output logged; at least two real phones decode v2.

---

## P7: Receiver theater: real state machine and live spectrogram

**Mode:** Spec → Feature. Use the design-director agent for the UI tasks and attach the concept boards. · **When:** Week 3 · **Priority:** HIGH

```
Create a feature spec "receiver-theater". Make the 5–20 s of reception suspenseful and truthful.

1. Decoder events (throttled):
   - signal: level per profile, from carrier-band energy
   - lock: preamble found
   - progress: bytes decoded / total, once the header is decoded
   - packet: CRC valid
   - corrupt: reason is CRC, header or timeout
   The DSP code stays UI-free and tested.
2. Receiver states: LISTENING → SIGNAL DETECTED → LOCKED → DECODING n% → VERIFIED, or → SIGNAL LOST / CORRUPTED. Show failures honestly. Use large type, aria-live announcements, and haptics where supported.
3. A live spectrogram (waterfall) from the real microphone stream.
   - Markers at the active carriers (1.3/2.3 kHz), with a zoomable 17–20 kHz band.
   - Outbreak skin: phosphor-green radar and waterfall. Veil skin: "spirit box" violet mist.
   - Reduced-motion fallback: static level bars.
   - Pause rendering when hidden. Budget: at most 2 ms per frame on a mid-range phone.
4. Reveal: the text appears only after the CRC passes, with a materialize animation, the meta line "CRC VERIFIED • ACOUSTIC CHANNEL • <profile>", and the FEC count.
5. Transmitter mirror: show the outgoing waveform/spectrum and progress while playing.

Every visual must come from real mic samples or real decoder state. Add a test that fails if any receiver UI path can render text without a CRC-valid packet.
```

**Done when:** a real phone shows every state, including a deliberately provoked CORRUPTED state; the "no text without CRC" test passes.

---

## P8: Calibration wizard / "Spirit Sweep"

**Mode:** Spec → Feature · **When:** Weeks 3–4 · **Priority:** MED

```
Create a feature spec "channel-calibration" that matches the concept board's Channel Calibration panel. The Veil skin calls it "Spirit Sweep". It is measurement-only.

1. The Tower plays a self-synchronizing sweep: a start chirp, then stepped tones of about 250 ms each (1, 2, 4, 8, 12, 15, 16, 17, 18, 18.5, 19, 19.5, 20 kHz), then a short test packet in each profile.
2. The receiver detects the chirp, measures each step's power against its noise floor (SNR per band), checks for clipping, and decodes the test packets.
3. Verdict for each device, shown with the chart: either "Near-ultrasonic usable (18.2 kHz: x dB, 19.2 kHz: y dB)" or "Use audible". Never show OK without a measurement. Store the last result locally on each device.
4. The speaker and mic checks are real level measurements, not fixed checkmarks.
5. Unit-test the detector by running the sweep through the acoustic-lab room model.
```

**Done when:** two different phones produce different, measured verdicts, recorded in the device matrix.

---

## P9: Landing page and console redesign to match the concept boards

**Mode:** Spec → Feature, then polish details in vibe mode · **When:** Week 4 · **Priority:** HIGH

```
Create a feature spec "dead-air-experience". Bring the UI up to the concept boards (attached) while keeping every claim true.

1. Structure.
   - index.html becomes a cinematic landing page: the DEAD AIR title; two story cards (THE LAST BROADCAST, a post-apocalyptic emergency network; THE VEIL, a paranormal transmission experience); and buttons for How It Works, Run Demo (which opens the judge ladder, P10) and Settings.
   - broadcast.html?scene=outbreak|veil becomes the console, with a sidebar: Broadcast, Calibration/Spirit Sweep, Status/Evidence Log, Help.
   - Remove the duplicated markup between index.html and broadcast.html.
2. Art direction.
   - Theme token sets from .kiro/steering/ui-design.md.
   - Self-hosted OFL fonts (WOFF2), cached by the service worker.
   - Hero art from my own concept images, converted to WebP/AVIF at 250 KB or less each, with CSS fallbacks.
   - No UI text baked into images. Tasteful scanlines and noise. Honor prefers-reduced-motion.
3. Truthful copy, changed from the boards:
   - "BROWSER OFFLINE" plus a self-attested radio checklist, instead of "OFFLINE MODE ✓ (No Wi-Fi, No Cellular, No Bluetooth)".
   - "Near-Ultrasonic (Quiet · experimental)" instead of "Silent".
   - Fix the footer typo "KIROW EEN".
4. How It Works page.
   - A laptop → sound → phones diagram.
   - A heritage strip: 1962 Bell 103 modem (data as tones) → acoustic couplers → 1975 Kansas City Standard cassette data → 2026 DEAD AIR.
   - Real-world uses: outage shelters; classrooms and events without Wi-Fi; captions of PA announcements for Deaf and hard-of-hearing visitors; tours and theater.
   - Privacy: no accounts, servers or tracking.
5. Visual QA loop. Capture screenshots of both scenes at 390×844 and 1440×900, using a browser-automation MCP server or Power if one is available, otherwise Playwright. Compare them with the boards and iterate. Note which details were polished in vibe mode.
```

**Done when:** screenshots of both scenes at phone and desktop sizes are committed under `docs/screenshots/`; the claims lint passes; Lighthouse accessibility is 95 or higher.

---

## P10: Judge ladder: proof in 60 seconds on one device

**Mode:** Spec → Feature · **When:** Week 4 · **Priority:** MUST

```
Create a feature spec "judge-ladder". A judge with one laptop and two minutes must be able to see the real decoder work. "Run Demo" opens three rungs:

1. Zero-hardware proof (30 s or less, no permissions).
   - Decode a bundled REAL field recording, captured in P5 (pick a successful clip), live in the browser through the same decoder, with the P7 states, spectrogram and reveal.
   - Show the clip's device, distance, room and date, labeled "Recorded on a real phone · decoded now in your browser".
   - Never present a synthetic render as a recording. If you also offer a synthetic sample, label it SYNTHETIC.
2. One device, real air path.
   - Acoustic loopback: the laptop listens with its microphone while playing a short packet through its speakers (or use two tabs).
   - Explain that headphones must be unplugged, and show diagnostics on failure.
3. Full room.
   - Laptop plus phones. Show a locally generated QR code so phones can open the receiver while still online, then the radios-off checklist.

Cache the sample clip for offline use. Update docs/JUDGE_QUICKSTART.md and add a 60-second path at the top of the README.
```

**Done when:** rung 1 works on a fresh browser with no microphone; rung 2 works on at least one Mac and one Windows laptop, with results recorded.

---

## P11: Beacon mode, quick phrases, phrase codebook (vibe coding)

**Mode:** Vibe · **When:** Week 4 · **Priority:** MED

```
Add these to the Tower in vibe mode:

1. Beacon mode.
   - Repeat the current message every 15/30/60/120 s until stopped, reusing the same packet ID so receivers dedupe.
   - Show a cycle counter. Auto-stop after 10 minutes. Show a volume reminder.
   - Story copy: "The tower repeats every 60 seconds for late arrivals."
2. Quick phrases per scene.
   - Outbreak: SHELTER IN PLACE · WATER AT NORTH GATE · MEDIC NEEDED · WE ARE STILL HERE
   - Veil: ARE YOU STILL HERE? · WE HEAR YOU · THE DOOR WAS NEVER LOCKED
3. Optional codebook, like radio Q-codes and telegraph codebooks.
   - Send a known phrase as a 1-byte code. Airtime drops to header + 1 byte: about 6 s with v1 framing, far less with v2.
   - Receivers render the phrase from the cached codebook and mark it "CODEBOOK".
   - Version the codebook inside the packet so mismatched builds fail safely.

Write tests for codebook encode/decode and for the beacon scheduling logic.
```

**Done when:** a phone that starts listening mid-beacon receives the next cycle; codebook tests are green.

---

## P12: Stretch goals (only if P0–P11 are done)

**Mode:** Spec → Feature, one spec each · **Priority:** STRETCH

```
A) Spec "survivor-roll-call".
   - After a broadcast, the Tower listens for 10–20 s.
   - A receiver that got a CRC-valid message can tap "I HEAR YOU" (or "NEED HELP"). This sends a tiny acoustic ACK (message ID, a random 8-bit tag, status) in a random slot: slotted ALOHA with 4–6 slots and one retry.
   - The Tower lists each verified ACK it decodes, e.g. "3 ACKs HEARD (not a count of all receivers)". Veil skin: "the investigators answer".
   - This needs mic permission on the Tower. Handle collisions honestly.
B) Spec "tape-mode".
   - Export any broadcast as a WAV file, so it can be recorded to a voice memo or even a cassette, and decode audio files offline. This nods to 1975 Kansas City Standard cassette data.
   - Share the decoder path with the judge ladder.
```

---

## P13: Release gate and real browser E2E tests

**Mode:** chat (workflow and tests) · **When:** Weeks 4–5 · **Priority:** MED

```
1. Browser E2E tests.
   - Add @playwright/test as a devDependency only; the app itself stays dependency-free.
   - Launch Chromium with --use-fake-ui-for-media-stream --use-fake-device-for-media-stream --use-file-for-fake-audio-capture=<wav>, where the WAV is rendered by our own encoder.
   - Open the receiver and assert that the CRC-verified reveal shows the exact text, for both profiles at 48 kHz, plus a negative case (an alias frame produces no reveal).
   - Check that Chromium's fake capture actually preserves 18–19 kHz content. If it resamples or filters, document that and record the ultrasonic E2E test as a known limitation.
   - Add a CI job.
2. .kiro/workflows/release-gate.workflow.json.
   - A parallel node with four branches:
     - DSP: signal-scientist runs npm run check, the benchmark and an acoustic-lab sweep summary.
     - UX: design-director takes screenshots of both scenes at phone and desktop sizes and runs accessibility checks.
     - Truth: failure-hunter runs lint-claims and cross-checks the docs against the evidence log and device matrix.
     - Submission: release-captain runs the checklist from #release-checklist and the competition steering.
   - joinPolicy: all. Then a final step writes docs/RELEASE_GATE.md as a PASS / FAIL / UNVERIFIED table.
   - Run validate_workflow, then run it, fix failures and re-run.
```

**Done when:** CI runs the E2E job; `docs/RELEASE_GATE.md` shows no FAIL rows.

---

## P14: Submission kit

**Mode:** chat plus a new skill · **When:** Week 5 (feature freeze Mon Nov 30) · **Priority:** MUST

```
Create .kiro/skills/devpost-writeup/SKILL.md (name: devpost-writeup). It turns docs/KIRO_EVIDENCE_LOG.md, docs/BENCHMARKS.md, docs/DEVICE_TEST_MATRIX.md and the specs into submission text. Use it to produce:

1. A README.md rewrite:
   - hero image or GIF, one-line pitch, live URL, and the 60-second judge path;
   - a how-it-works diagram;
   - a "Built with Kiro" section that links each spec, hook, steering file, agent, workflow, skill and MCP tool to a concrete outcome;
   - honest Limitations;
   - one short Provenance paragraph: ChatGPT-drafted baseline → tag pre-kiro-baseline; everything after it was done in Kiro, per the evidence log.
   Remove "The repository is presently private".
2. docs/DEVPOST_SUBMISSION.md:
   - Sections: Inspiration · What it does · How we built it · Challenges · Accomplishments · What we learned · What's next · Category: Resurrection · Testing instructions.
   - "How we built it" goes feature by feature through Kiro: what we did, a real example, and the measured effect (airtime before/after, the bugfix-spec story, sweep tables, the hooks that wrote the evidence log).
3. docs/VIDEO_SHOT_LIST.md: 2:50 or less, time-coded.
   - Cold open.
   - Radios-off settings in the same frame as the phones.
   - One continuous real multi-phone receive.
   - Near-ultrasonic segment with an honest caveat.
   - The Veil.
   - Kiro segment with real screen captures: a spec task run, a hook firing, an MCP sweep, the workflow.
   - Close.
   No copyrighted music; add captions.
4. docs/BLOG_DRAFT.md for dev.to or Medium: the build story. It must say it was written for Kiroween 2026. I will publish it before Dec 4 and add the URL to Devpost.

Only state what the evidence log, benchmarks and device matrix support. Run the release-gate workflow last.
```

**Done when:** the README, Devpost text, shot list and blog draft exist and pass the claims lint; the release gate is green; the video is recorded; you submit by Thu Dec 3.
