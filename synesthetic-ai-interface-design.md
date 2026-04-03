# Synesthetic Perception as a Design Methodology for AI Interfaces

**Sean "Mook" DiMarco**  
Creative Technologist | Boston, MA | sfdimarco@gmail.com  
April 2026

---

## Abstract

This paper proposes that **grapheme-color synesthesia** — a neurological condition in which digits and letters involuntarily elicit color percepts — constitutes a legitimate and novel design methodology for AI interfaces. Rather than treating synesthetic perception as a curiosity or accessibility edge case, I argue it provides a form of multi-channel data processing that is structurally analogous to how high-dimensional AI systems encode information. Designers with synesthesia may have native access to perceptual feedback channels that most developers approximate through visualization tools. The implications for AI-assisted interface design, prompt engineering, and human-AI collaboration are explored.

---

## 1. Introduction

Most interface design begins with aesthetics: mood boards, brand colors, visual hierarchy conventions. Synesthetic interface design begins differently. When I open a code editor, I am not reading tokens — I am reading a color field. When a variable name is inconsistent with its value, it does not just look wrong. It looks wrong the way a sour note sounds wrong: immediately, involuntarily, before any reasoning has occurred.

This paper documents the SYN color system I have developed from my own grapheme-color synesthesia, and proposes that this system — when applied deliberately to AI interface design — produces perceptual coherence that cannot be achieved by purely analytical design processes.

I am not arguing that synesthesia is required for good design. I am arguing that it provides an unusually direct signal about the relationship between information structure and perceptual response — and that this signal is relevant to AI systems in particular, because AI systems are themselves high-dimensional pattern encoders.

---

## 2. The SYN Color System

My digit-to-color percepts are stable, consistent, and involuntary:

| Digit | Perceived Color | SYN Hex   |
|-------|----------------|-----------|
| 0     | Cyan           | `#00bcd4` |
| 1     | Red            | `#e53935` |
| 2     | Blue           | `#1565c0` |
| 3     | Yellow         | `#f9a825` |
| 4     | Green          | `#2e7d32` |
| 5     | Orange         | `#ef6c00` |
| 6     | Purple         | `#6a1b9a` |
| 7     | Pink           | `#c2185b` |
| 8     | Near-white     | `#f5f5f5` |
| 9     | Near-black     | `#212121` |

These percepts extend to letters via digit-encoding (A encodes from its shape similarity to certain digits, etc.). The result is that any piece of text I read carries a simultaneous color signal.

Applied to interface design, this creates a constraint that is actually a feature: **every color decision is cross-referenced against a stable perceptual standard**. If a UI element assigned to "important" uses a color that my perceptual system associates with something low-weight, the dissonance is felt before it is analyzed.

---

## 3. AI Systems as Color Spaces

Large language models and image generation models encode meaning in high-dimensional vector spaces. Tokens, concepts, and relationships are represented as positions in a latent space where proximity implies semantic relationship.

The interesting parallel: synesthetic perception can be understood as a natural dimensionality reduction — a mapping from abstract token space (words, digits) to a lower-dimensional but rich perceptual space (color, form, weight). When I navigate a codebase, I am performing a kind of spatial query on colored information. When something is "out of place," I detect it through the same mechanism that detects a wrong color in a painting.

This is not just metaphor. Synesthetic cognition involves real cross-activation of neural pathways — the same regions that process color receive input from regions that process graphemes. It is a literal multi-channel integration system.

AI systems designed for humans would benefit from respecting this channel, rather than defaulting to monochrome, text-dense interfaces that compress all information into a single stream.

---

## 4. Field Research: Handing AI to a 4th Grader

The most useful research method I have developed is simple: **build an AI tool, then watch what breaks when a 9-year-old uses it.**

Children do not accommodate broken interfaces. They do not rationalize confusion or attribute fault to themselves. When a tool confuses a child, the tool is broken. Period.

Over 10 years of classroom teaching across EMPOW Studios (25 schools) and The Chestnut Hill School, I have observed that:

1. **Visual feedback loops accelerate learning** — tools that show the consequence of an action immediately (p5.js live preview, Blockly code generation) produce faster comprehension than tools with delayed feedback.

2. **Perceptual consistency matters more than feature richness** — children abandon tools that feel visually inconsistent faster than tools with fewer features.

3. **Color carries information** — color-coded interfaces require less working memory than text-labeled ones at early reading levels. This is not a preference — it is a cognitive efficiency.

These observations align with the SYN framework: consistent color-to-meaning mapping reduces cognitive load, regardless of whether the user is synesthetic.

---

## 5. AI Collaboration as Co-Research

Working with Claude and Gemini, I have developed a methodology that treats AI collaboration as a design research process rather than a tool usage process:

- **Gemini for visual grammar**: The Mowersville production backgrounds were built by developing a visual grammar with Gemini that it could hold consistently across six scenes. This required iterative negotiation of visual rules — not just prompting, but co-authoring a constraint system that the AI could apply reliably.

- **Claude for system architecture**: When designing complex interactive systems, I treat Claude's responses as early prototypes — not final answers, but first-pass structural proposals that I then revise against my own perceptual standards.

The critical observation: **I can detect when a model is pattern-matching instead of reasoning** before I can articulate why. The response "feels" structurally flat — like a sequence of correctly-shaped tokens that don't add up to a coherent whole. This is the synesthetic signal applied to language rather than digits.

---

## 6. Implications for AI Interface Design

1. **Color systems should be grounded, not arbitrary.** Randomly assigned "brand colors" carry no cognitive weight. Color systems derived from consistent internal logic — whether synesthetic, semantic, or hierarchical — produce interfaces that users can internalize faster.

2. **Perceptual feedback channels beyond text are undertapped.** Most AI interfaces default to chat-window patterns: text in, text out. Audio, color, spatial layout, and motion all carry information. Synesthetic designers are more likely to reach for these channels naturally.

3. **Diverse perception is a design asset.** ADHD, autism, and synesthesia are not obstacles to navigated around in UX. They are alternate signal-processing architectures. Designing from inside those architectures produces tools that work for more people, because the constraints are more demanding.

---

## 7. Conclusion

Synesthetic perception is not a disability, an ability, or a gimmick. It is a cognitive architecture — one that happens to be well-suited to the problems of AI interface design: high-dimensional information, multi-channel representation, and the question of what makes an AI response feel right before you can say why.

The SYN color system is one implementation of this. The broader claim is that **what your nervous system tells you before your reasoning catches up is valid design data** — and that AI tools built by people who attend to that signal will be different, and better, than tools built purely analytically.

---

## About the Author

Sean "Mook" DiMarco is a Creative Technologist, Animator, and Educator based in Boston, MA. He has taught K–8 students to code for 10 years, built AI tools deployed in 25+ schools via EMPOW Studios, and holds a BFA in Animation from Lesley University. He has grapheme-color synesthesia, ADHD, and autism. He is looking for a hybrid EdTech / Creative AI role.

[sfdimarco.github.io](https://sfdimarco.github.io) · [sfdimarco@gmail.com](mailto:sfdimarco@gmail.com) · [LinkedIn](https://www.linkedin.com/in/sean-dimarco/)
