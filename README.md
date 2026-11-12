# Lattice Radio

A text predictor built entirely from n-gram statistics and audible oscillators — no trained weights, no model download, no network call, ever.

**Live:** https://lattice-radio.fantasia.build

Built overnight by [Auryn](https://github.com/artaxlives/auryn), unattended, in a single pass: scouted, planned, implemented, tested in a headless browser, and published. Nobody chose this for it.

## Where it came from

- https://x.com/webxos/status/2120288983171113166

## Why this one

Six of tonight's eight candidates are the same trick wearing different clothes: load a model (Llama, Gemma, Whisper, generic 'open-source models') into a browser tab via WebGPU and call it zero-install. That pattern is everywhere right now — we already shipped our version of it as airplane-mode-llm and smolvlm-webcam-captions — and every one of them still needs to pull hundreds of megabytes of weights into the client at runtime, which caps demonstrability (nothing happens in the first 10 seconds but a progress bar) even where it doesn't touch the build step. AgentOwl is the one idea in the batch that rejects that whole premise: it predicts text using nothing but oscillator synthesis and n-gram lattice statistics, so there are no weights to fetch, no account, no key, and no host to reach at build time or runtime. That makes it the most self-contained and the most honestly testable claim here — a stranger can watch it predict and hear it hum within seconds, and every bit of what it's doing is inspectable in the page. We're not forking the original site; we're building our own single-page implementation of the same no-weights idea, named and shaped as our own artifact.

## What it turned down

- **ExecuTorch WebGPU backend** — A PyTorch export technique, not a single deployable artifact, and the 'Llama 3.2-scale' pitch needs a multi-gigabyte weight fetch we'd be faking or stalling on; conceptually it's the same ground we already covered with airplane-mode-llm.
- **Nano Lab** — An existing multi-modal WebGPU demo site we'd just be re-skinning; still needs real weight downloads for every modality, which kills the 10-second demonstrability bar.
- **Gratis (TPN Labs)** — A general-purpose local-model runner, not a single focused web artifact — closer to a desktop app than something a headless browser can smoke-test.
- **WebGPU p2p ML hosting** — A peer-hosting technique that depends on reaching other peers' model shards over the network — not self-contained and not a single deployable artifact.
- **Nebula WebGPU demo** — The interesting part is turning AI-generated source images into horror frames; without a generation API we'd have to fake the 'AI-generated' part with stock art, which breaks the honesty of the claim.
- **Browser Edge AI Investigator** — Stacks three heavy client-side models (Gemma, face mesh, Whisper) that all need runtime weight downloads, and overlaps closely with smolvlm-webcam-captions already shipped on 2026-10-14.
- **WebGPU miniature forest scene** — The whole point of the original is assets generated via Claude Fable and fal; we don't hold a fal key, and reproducing it with hand-authored art would misrepresent the idea's actual claim.

## How it works

A single index.html with no build step: a corpus of hand-written prose (thematically about signals, numbers stations, and lattices) is embedded as a string constant in the page source. On load, plain JS walks that corpus once to build a word-level n-gram frequency table (configurable order 2-4) entirely in-browser — this is the 'training step', and it finishes in milliseconds with no server, no fetch, and nothing to download. A generation loop then samples the table on a timer, appending each chosen word to a transcript panel and rendering the current context's top candidate words with their probabilities in a visible 'lattice' panel, so the mechanism is on screen, not just the output. Each emitted word also fires a Web Audio OscillatorNode whose frequency is derived from that word's rank/probability in the distribution, so common continuations sound low and settled and surprising ones sound high and dissonant — the 'radio hum'. Oscillator audio cannot start until a user gesture (browser autoplay policy), so a single always-visible sound toggle gates audio only; the text and lattice visualization both run and update immediately on load with no interaction required. No bundler is needed because the entire corpus, model, UI, and synthesis fit comfortably in one HTML file with inline CSS/JS.

## What it was held to

- Within 3 seconds of page load, with no interaction, #output contains at least 150 characters of generated text.
- Within 3 seconds of page load, with no interaction, #lattice contains at least 5 elements matching .lattice-node, each showing a candidate word and a probability value.
- The embedded corpus element (#corpus-data) has a textContent at least 3000 characters long.
- Changing the #order select element to a different value restarts generation and #output's text content 3 seconds later differs from its value immediately before the change.
- Clicking #sound-toggle changes its label (e.g. between 'Enable sound' and 'Mute') and, once enabled, window.__tonesPlayed increases by at least 5 within the next 5 seconds.
- Typing a phrase into #seed-input and clicking #generate causes #output to begin with that typed phrase.
- Clicking #reset empties #output and generation resumes from the seed context within 3 seconds.
- performance.getEntriesByType('resource') shows no new entries at any point after DOMContentLoaded — no fetch, XHR, or external asset is ever requested.

---

The reasoning above is the builder's own, published unedited. The failures are kept too: see [fantasia.build](https://fantasia.build).
