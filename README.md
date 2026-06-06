# agent-phrasing

**Energy contour phrase detection for agent output.**

Listen to a great speaker. They don't deliver information at a constant rate. There are builds, climaxes, pauses. The pause before the key point isn't dead air — it's the audience catching up. The crescendo before the reveal isn't padding — it's tension.

Musicians call this phrasing. A phrase is a complete musical thought with a beginning, middle, and end. The space between phrases is as important as the phrases themselves.

`agent-phrasing` detects when agent output naturally groups into phrases — chunks of high-energy work separated by recovery dips. Raw output streams are noise. Phrased output is signal.

## Why This Exists

Agent systems produce continuous streams of output. Without phrase detection, you're left with an undifferentiated firehose. Is the agent building toward something? Winding down? Treading water? You can't tell.

Phrase detection solves this by analyzing the energy contour of agent actions. When energy rises, a phrase is starting. When it dips below a threshold, the phrase ends. The result: structured, readable output instead of a wall of events.

## Core Idea

Actions have energy. Energy contours form phrases. Phrases have shape.

Four shapes cover most agent behavior:

- **Arch** — Energy peaks in the middle. The classic "warm up → execute → cool down" pattern. Most common.
- **Crescendo** — Energy builds throughout. A push toward a deadline or escalation.
- **Decrescendo** — Energy fades. Wrapping up, handing off, or losing steam.
- **Flat** — No phrasing detected. Either too short to classify or genuinely uniform.

The breakthrough insight: **the gap between phrases matters as much as the phrases.** `breathing_room()` measures whether a phrase ends cleanly enough for the next one to begin. A phrase that ends at high energy leaves no room — the next phrase crashes into it.

## Architecture

```
Action (id + weight + energy)
  │
  ├─ Phrase::detect(actions, dip_threshold)
  │    └─ Rises start phrases, dips end them
  │
  ├─ Phrase::shape()
  │    └─ Arch / Crescendo / Decrescendo / Flat
  │
  ├─ Phrase::breathing_room(next_phrase)
  │    └─ Gap between end_energy and next start_energy
  │
  └─ PhrasingAnalysis::analyze(actions, dip_threshold)
       ├─ phrase_count
       ├─ avg_phrase_length
       ├─ avg_breathing_room
       ├─ shape_distribution [arch, crescendo, decrescendo, flat]
       └─ phrase_coverage (fraction of actions in phrases)
```

## Usage

### Detect Phrases in an Action Stream

```rust
use agent_phrasing::{Action, Phrase};

let actions = vec![
    Action { id: 0, weight: 0.1, energy: 0.1 },  // quiet start
    Action { id: 1, weight: 0.5, energy: 0.6 },  // building
    Action { id: 2, weight: 0.9, energy: 0.9 },  // peak
    Action { id: 3, weight: 0.3, energy: 0.2 },  // dip below threshold
    Action { id: 4, weight: 0.4, energy: 0.5 },  // new phrase starts
    Action { id: 5, weight: 0.7, energy: 0.8 },  // building again
    Action { id: 6, weight: 0.2, energy: 0.1 },  // ending
];

let phrases = Phrase::detect(&actions, 0.3);
// Two phrases detected, split at the energy dip
```

### Analyze Phrase Shape

```rust
// An arch-shaped phrase
let phrase = Phrase {
    actions: vec![
        Action { id: 0, weight: 0.3, energy: 0.2 },
        Action { id: 1, weight: 0.8, energy: 0.9 },
        Action { id: 2, weight: 0.3, energy: 0.2 },
    ],
    start_energy: 0.2,
    peak_energy: 0.9,
    end_energy: 0.2,
};

match phrase.shape() {
    PhraseShape::Arch => println!("Classic warm-up, execute, cool-down"),
    PhraseShape::Crescendo => println!("Escalating intensity"),
    PhraseShape::Decrescendo => println!("Winding down"),
    PhraseShape::Flat => println!("Uniform or too short to classify"),
}
```

### Measure Breathing Room

```rust
let p1 = Phrase { 
    actions: vec![], start_energy: 0.5, peak_energy: 0.8, end_energy: 0.1 
};
let p2 = Phrase { 
    actions: vec![], start_energy: 0.6, peak_energy: 0.9, end_energy: 0.2 
};

let room = p1.breathing_room(&p2); // 0.5 — good gap
// p1 ended low (0.1), p2 starts higher (0.6) → clean separation
```

Breathing room is the difference between a good speaker and a rambler. A phrase that trails off softly gives the next phrase space to build. A phrase that ends at peak energy? The next one has nowhere to go but crash.

### Full Stream Analysis

```rust
use agent_phrasing::PhrasingAnalysis;

let analysis = PhrasingAnalysis::analyze(&actions, 0.3);
println!("Phrases detected: {}", analysis.phrase_count);
println!("Average length: {:.1} actions", analysis.avg_phrase_length);
println!("Breathing room: {:.2}", analysis.avg_breathing_room);
println!("Coverage: {:.0}%", analysis.phrase_coverage * 100.0);
println!("Shape distribution: {:?}", analysis.shape_distribution);
```

## API Reference

| Type | Purpose |
|------|---------|
| `Action` | Single action with weight and energy |
| `Phrase` | Group of actions forming a complete thought |
| `PhraseShape` | `Arch` / `Crescendo` / `Decrescendo` / `Flat` |
| `PhrasingAnalysis` | Full-stream analysis with aggregate stats |

### Key Methods

| Method | Returns |
|--------|---------|
| `Phrase::detect(actions, threshold)` | `Vec<Phrase>` — splits stream on energy dips |
| `Phrase::shape()` | `PhraseShape` — classifies the contour |
| `Phrase::breathing_room(next)` | `f64` — gap quality between phrases |
| `Phrase::total_weight()` | `f64` — summed importance |
| `PhrasingAnalysis::analyze(actions, threshold)` | Complete breakdown |

## The Deeper Idea

Phrase detection is really about *segmentation* — finding the natural boundaries in a continuous signal. The `dip_threshold` parameter is the key tuning knob. Set it too high and everything is one long phrase. Set it too low and every fluctuation creates a break.

The right threshold depends on your domain. For most agent workloads, 0.3 works well — it detects phrases where energy genuinely falls off, not just minor fluctuations. But for high-intensity domains (real-time trading, emergency response), you might want 0.5. For background tasks, 0.15.

The shape distribution tells you about the health of your agent's work patterns. A healthy system has mostly arches with some crescendos. A system with lots of flat phrases might be stuck. A system with only decrescendos might be running out of steam. The distribution is diagnostic.

## Related Crates

- **`agent-groove`** — Timing and feel for agent scheduling (the *rhythm*)
- **`agent-intonation`** — Accuracy measurement (how precisely agents match intent)
- **`agent-orchestration`** — Fleet dynamics as orchestral composition (who plays *loud*)
- **`agent-counterpoint`** — Species counterpoint for fleet coordination (how voices *relate*)
- **`agent-ensemble`** — The experiment proving musical coordination beats mechanical approaches

## License

MIT
