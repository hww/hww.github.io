# Text or Nodes? Two Ways AAA Studios Let Designers Build Games

*What Naughty Dog and Kojima Productions can teach an indie studio*

*By Valeria Pudova*

---

Indie developers sometimes treat AAA games as an ideal that can never be reached, and that attitude is a risk in itself. I would argue the opposite: projects like *Death Stranding* and *The Last of Us* should be studied by everyone, and their methods borrowed for our own work.

This article compares the development approaches of *Kojima Productions* and *Naughty Dog*. The key finding is that there are **two main ways to describe game logic and data: visual and textual.** Both work. Both studios shipped excellent games with them, and the two studios are often seen as opposites. But beneath that difference, their architectures share the same foundation, and that foundation is what an indie team should copy. At the end, I will also say honestly which side I lean toward, and why.

## The vision and how well it was realized

To compare the two games without emotion, look at what the creators set out to do and how closely the result matches it. By that measure, both games are flawless:

- **The Last of Us** is a game-as-cinema, where the direction of each moment matters most. Everything serves one purpose: revealing the characters and driving the drama forward. The player is an observer, deeply absorbed in the relationships between the heroes, in tense encounters, and sometimes frightening events.
- **Death Stranding** is a game-as-journey. Its characters are deliberately abstract. The goal is to speak directly to the player: place them in a set of circumstances, give them tools, make them act, and then confront them, in freedom, with obstacles and opposing forces.

Finishing on time is not enough to achieve this. A project succeeds in proportion to the time spent on the important details players love, and that requires development **tooling** that is efficient, flexible and scalable. The tools are what buy the team that time.

---

## Two answers to the same question

Both teams got the result they wanted, so their technology clearly helped. But they chose opposite ways of letting designers define data and logic:

- **Naughty Dog: text.** Game designers use a specialized text-based **DC script language**, a *Domain Specific Language* (DSL) that works equally well for describing complex data structures and for implementing logic.
- **Kojima Productions: visual nodes.** They use the **Decima** engine (from *Guerrilla Games*, the studio behind *Horizon Zero Dawn*), where data structures and logic are declared mainly through **graphical node-based programming**.

---

## What both approaches have in common

Despite the fundamental difference in how designers enter their work (text versus graphics), the two architectures are strikingly similar. Four principles hold in both:

1. **Two layers.** At the core are two independent layers: a high-performance base in **C++**, and high-level game logic in a dynamic, domain-specific language.
2. **Designer autonomy.** A game designer can create any data structure or behavior they need **without a programmer** and without touching the engine's source code.
3. **Seamless iteration (hot reload).** Changes can be made and tested instantly **without restarting the game**, even while debugging on a remote target platform or console.
4. **Parallel production.** The tools are built from the start for a large, distributed team to work at once, and they technically minimize conflicts when assets are merged.

These four points are the real lesson. Whether you choose text or nodes, adopting them can multiply the speed and quality of any indie studio's work.

---

## Where they differ: text versus visuals

Neither approach is wrong, and I do not claim that graphical programming is inferior in every case. Still, I personally lean toward Naughty Dog's text-based approach, for these reasons:

- **Ease of use over the long term.** The one clear advantage of visual nodes is that a designer learns them faster. But *easier to learn* is not the same as *easier and more efficient to use* over years of production. Most programs in the world are written as text, and that is not an accident.
- **Readability.** For me, text is simply easier to read and to work with.
- **Decomposition.** It is easy to split a complex system into components such as modules.
- **Merging.** Text merges cleanly when several people change the same material, which matters a great deal for parallel production.
- **Flexibility.** This is the most important point. A text-based DSL is far more flexible than any visual tool, especially when combined with domain-specific and **data-oriented programming**. It also scales better as the system grows.

To be fair, the comparison I would most like to make is one of budgets and person-hours spent on both projects. Reliable figures exist only for *Naughty Dog*; the numbers for *Kojima Productions* are contradictory in open sources. Industry data suggests that Kojima's project needed considerably more resources for a comparable result, but this is not conclusive, and I treat it as a hint rather than proof.

---

## Conclusions

Two principles lie at the heart of AAA-level development methods that an indie studio can apply:

1. **Making a game is an iterative process.** Tools should remove, or at least minimize, the need for designers and content creators to compile code or restart the game client.
2. **Making a game is a collective process.** When dozens of people work on one project at the same time, the architecture of the tools must minimize merge conflicts and waiting time.

Text or nodes is a choice you can make freely, and I have explained why I choose text. These principles, however, are **universal and apply regardless of engine or target platform.**

By studying and adopting the practices of industry leaders, you save resources today and raise your team's professionalism, which determines the final result: a responsive, polished and commercially successful game.

---

## References

- **\[1\]** *More than half of Death Stranding’s team defected from Konami* (analysis of the Kojima Productions team composition).
- **\[2\]** *Outernauts: From AAA Console to AAA Flash Games*, Joe Valenzuela, Insomniac Games (applying AAA methodology to small projects).
- **\[3\]** *Unsynced: The Last of Us Melee System*, Anthony Newman, Naughty Dog, Inc. (specification of the DSL and runtime data systems).
