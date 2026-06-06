# agent-phrasing

*The space between the notes is where the meaning lives.*

---

In music, phrasing is how a performer groups notes into meaningful musical sentences. A phrase has a beginning (pick-up), a peak (climax), and an ending (resolution). The silence between phrases — the breath — gives the listener time to process what they heard before the next thought begins.

Agent output streams need the same thing. A constant stream of actions is noise. Phrased output — grouped into meaningful chunks with pauses between them — is signal.

This crate detects phrases in agent action streams using energy contour analysis. An action's "energy" represents its intensity or importance. The detector identifies natural phrase boundaries where energy dips below a threshold — the musical equivalent of a breath mark.

Provides: phrase detection via energy contours, phrase shape classification (arch/crescendo/decrescendo/flat), breathing room measurement between phrases, and full phrasing analysis with coverage metrics.

The insight: well-phrased agent output is easier to compose with. Other agents can process complete phrases rather than fragment streams. The breathing room between phrases is where the listener — human or agent — processes meaning. No breathing room = cognitive overload.

9 tests: phrase detection, multi-phrase streams, shape classification, breathing room, coverage analysis, edge cases.

Part of [SuperInstance](https://github.com/SuperInstance/SuperInstance).

License: MIT
