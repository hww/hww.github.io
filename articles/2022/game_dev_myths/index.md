# Myths and Misconceptions of a Game Programmer

### How Not to Get Lost in Three Pines

<div style="display: flex; align-items: center; gap: 20px; max-width: 700px; margin: 25px 0; padding: 15px; border: 1px solid #e1e4e8; border-radius: 8px; background-color: #fafbfc; flex-wrap: wrap;">
  <div style="flex-shrink: 0;">
    <img src="/pp/wendy.png" alt="Valeriya Pudova" width="70" style="border-radius: 50%; display: block; box-shadow: 0 2px 6px rgba(0,0,0,0.1);"/>
  </div>
  <div style="flex: 1; min-width: 250px; line-height: 1.5;">
    <div style="font-size: 1.2em; font-weight: 700; color: #24292e;">Valeriya Pudova</div>
    <p style="margin: 3px 0 0 0; font-size: 0.9em; color: #586069;">
      💬 Telegram: <a href="https://t.me" style="text-decoration: none; color: #0366d6; font-weight: 600;">@valery_h2w</a>
    </p>
    <p style="margin: 2px 0 0 0; font-size: 0.9em; color: #586069;">
      ✉️ Email: <a href="mailto:valery.hww@gmail.com" style="text-decoration: none; color: #0366d6;">valery.hww@gmail.com</a>
    </p>
  </div>
  <div style="flex-shrink: 0; font-size: 0.9em; color: #6a737d; font-weight: 500; background: #e1e4e8; padding: 4px 10px; border-radius: 12px;">
    📅 09.05.2022
  </div>
</div>

<details style="border: 1px solid #e1e4e8; border-radius: 6px; padding: 12px; margin-bottom: 25px; background-color: #ffffff;">
  <summary style="font-weight: 600; cursor: pointer; color: #24292e; user-select: none;">📌 Table of Contents (Expand)</summary>
  <div style="margin-top: 10px; line-height: 1.6;">
    <ul style="margin: 0; padding-left: 20px; font-size: 0.95em;">
      <li><a href="#introduction" style="text-decoration: none; color: #0366d6;">Introduction</a></li>
      <li><a href="#the-problem-of-three-great-misconceptions" style="text-decoration: none; color: #0366d6;">The Problem of Three Great Misconceptions</a></li>
      <li><a href="#idealization-of-the-model" style="text-decoration: none; color: #0366d6;">Idealization of the Model</a></li>
      <li><a href="#the-comfort-zone-trap" style="text-decoration: none; color: #0366d6;">The Comfort Zone Trap</a></li>
      <li><a href="#the-high-or-low-level-dilemma" style="text-decoration: none; color: #0366d6;">The High or Low Level Dilemma</a></li>
      <li><a href="#the-architects-mistake" style="text-decoration: none; color: #0366d6;">The Architect's Mistake</a></li>
      <li><a href="#neglect-of-the-special-case" style="text-decoration: none; color: #0366d6;">Neglect of the Special Case</a></li>
      <li><a href="#exaggeration-of-the-importance-of-oop" style="text-decoration: none; color: #0366d6;">Exaggeration of the Importance of OOP</a></li>
      <li><a href="#exaggeration-of-the-importance-of-compilation" style="text-decoration: none; color: #0366d6;">Exaggeration of the Importance of Compilation</a></li>
      <li><a href="#preference-for-exclusively-gui-tools" style="text-decoration: none; color: #0366d6;">Preference for Exclusively GUI Tools</a></li>
      <li><a href="#love-of-visual-programming" style="text-decoration: none; color: #0366d6;">Love of Visual Programming</a></li>
      <li><a href="#neglect-of-documentation" style="text-decoration: none; color: #0366d6;">Neglect of Documentation</a></li>
      <li><a href="#unjustified-skepticism" style="text-decoration: none; color: #0366d6;">Unjustified Skepticism</a></li>
      <li><a href="#lack-of-a-holistic-view-of-the-problem" style="text-decoration: none; color: #0366d6;">Lack of a Holistic View of the Problem</a></li>
      <li><a href="#on-which-day-of-the-week-does-monday-actually-begin" style="text-decoration: none; color: #0366d6;">On Which Day of the Week Does Monday Actually Begin?..</a></li>
      <li><a href="#given-or-development" style="text-decoration: none; color: #0366d6;">Given or Development?</a></li>
      <li><a href="#conclusion" style="text-decoration: none; color: #0366d6;">Conclusion</a></li>
    </ul>
  </div>
</details>

<blockquote>
  <strong>Abstract:</strong> A person constantly makes mistakes — he who does nothing makes no mistakes. And even the best of us — experienced professionals — can make annoying mistakes and be mistaken. This document describes the common misconceptions and erroneous myths that I have encountered in my professional activity while developing game projects.
</blockquote>


------------------------------

## Introduction

Working with different people, I noticed that many programmers over time develop a certain professional deformation. In another industry, it might play a positive role or remain insignificant, but in game development it becomes destructive. Because of it, the programmer falls into the trap of their own rigid attitudes and makes critical mistakes when making architectural and management decisions.

## The Importance of Programming Is Exaggerated

The world's best specialists have formed a cult image of the software developer in mass culture. As a result, within game studios, programmers are often perceived as the "elite" of the team, while all other content creators are pushed to the second and third plan.
To restore balance, it is enough to honestly answer the question: "What exactly does each team member do?"

* The artist does everything to make the game beautiful.
* The animator does everything to make the world move and come alive.
* The game designer makes this world fascinating.
* The programmer makes this world functional.

Hence the key conclusion follows: the main task of a game programmer is to create such development tools for designers that will reveal their creativity. The tools must allow them to test any hypotheses as quickly and autonomously as possible, so that through iterations they can achieve the most fascinating gameplay.

------------------------------

## The Problem of Three Great Misconceptions

There are three fundamental misconceptions most common among software developers.

## Misconception 1: The Software Environment Is a Platform

In reality, the platform is the hardware architecture of the target device. Each piece of hardware has its own constraints and specifics that require unique engineering solutions.

Example: A game is written not "for Unity or Unreal" but for a specific target platform (whether it is a mobile device, a console, or a PC). The runtime architecture must take into account the constraints of the physical processor, memory, and graphics chip of the specific device.

## Misconception 2: Code Is Written for Simulating the World

The classic approach teaches us to hide data inside objects and manage it through their own methods (encapsulation). For example, to create a class "Wolf" that itself stores its parameters (health, speed, mesh) and manages them itself. In game development, this habit inevitably leads to an architectural dead end.
As the game grows, the wolf's mechanics begin to multiply: it must run, hunt, sleep, burn, be afraid of torches. If all this logic is stuffed inside the wolf, the class will grow into a gigantic, unmanageable module.
It seems that the problem can be solved by a "clever substitution": split the wolf into dozens of narrow classes (Wolf_Patrolling, Wolf_Attacking, Wolf_Frightened) or simply replace a whole chair with a broken one at the moment of impact. But in practice this leads to two serious problems:

* The maintenance problem (Combinatorial explosion): What to do if the wolf is simultaneously wounded, burning, and running away? You will have to create a separate class for this crazy combination. The number of classes and components begins to grow exponentially, and the code becomes overgrown with crutches for seamless data transfer between them.
* The comprehensibility problem: When the logic of hundreds of particular game situations is encapsulated inside the objects themselves, it is impossible to see the big picture. Data is isolated in a heap of containers, and the developer loses control over which process and at what moment controls the world.

In reality, simulating an interactive world requires completely breaking classic encapsulation. An object in a game is not a living entity. It is a passive set of pure data (coordinates, radius, current version of the model).
And all the code — state machines, damage logic, AI — must be moved aside, in the form of external processes. The code does not belong to the wolf or the chair. External processes simply look at the world from above and transform this passive data in time when the right moment comes.

## Misconception 3: Code Is More Important Than Data

The classical school of programming teaches that system design begins with code — with the architecture of classes, interfaces, and interaction logic. Data is perceived as something secondary, which simply needs to be laid out in the "pockets" of objects. In the game industry, this approach turns reality upside down and leads to serious consequences.

* When a programmer thinks only about code, they create redundant levels of abstraction to make the system "beautiful." In practice, this results in two fundamental problems: The performance problem (Cache Misses): Hardware does not care about the beauty of the class hierarchy. The processor cannot execute an "abstract wolf" — it can efficiently process dense, contiguous arrays of data in RAM. When data is encapsulated inside thousands of individual objects, it is scattered chaotically through memory. The processor spends colossal time on empty waits (cache misses), picking out the coordinates and health of creatures from different corners of memory.
* The logic complication problem: Trying to isolate code from data, the developer is forced to write tons of service code: getters, setters, factories, and adapters. Architecture is written for the sake of architecture itself. Instead of direct transformation of parameters, we get a cumbersome superstructure that only prevents the code from performing its only task.

In reality, data is the game, and code is just a tool for changing it. The world model is determined not by how functions are written but by how data structures are organized in memory. Effective simulation begins with designing tables, arrays, and data streams. Only when the data lies correctly does the code become simple, straightforward, and work at maximum processor speed.

## Misconception 4: Game World Objects Interact with Each Other Directly

When we see how one ball in a game hits another and pushes it away, our brain suggests the "obvious" OOP solution: the object Ball_1 detected the collision and called a method on the object Ball_2. This assumption generates the main problem of game dev — loss of control and unpredictability of the simulation.

If objects communicate directly, the code turns into spaghetti: all classes must know about all others. Chaos intensifies if the runtime has a non-deterministic execution order (as in Unity). Setting Execution Order does not save — it gives local guarantees for script types, but the update order of the object instances themselves on the scene remains random. Player actions add uncertainty, and the system goes haywire. The error may float: manifest once in a hundred hours, reproduce on one computer and be absent on another. Such bugs are extremely difficult to localize and fix. The game becomes non-deterministic; it cannot be repeated and tested.

In reality, the interactive world is built on a rigid hierarchy of external control. Objects know nothing about each other and do not interact with anyone. Instead, there are isolated subsystems — "little orchestras" — controlled by their conductors:

* The melee orchestra knows everything about the positions of fighters within a three-meter radius and predictably distributes their attacks so that they do not strike simultaneously.
* The visibility orchestra spins the mill wings only when the player looks at it and completely freezes its data, saving processor resources, when it is behind an obstacle or behind their back.

Control in the game is achieved not when we write "smart behavior of objects" but when we deprive objects of will, subordinating their passive data to external, isolated situation managers.

------------------------------

## Idealization of the Model

In the real world, the human brain is accustomed to strict classification: "a table is just a table." But in the virtual world, there is often nothing in common between objects of the same type at all.

Example: A static table (part of the background geometry), a physical table (an object with mass that can be pushed), and a destructible table (having a durability scale and shattering into fragments) are implemented absolutely differently from the engine architecture point of view.

Drawing direct parallels between objects of real life and virtual space is nothing more than a mental hint to simplify the programmer's thinking. For full-fledged modeling of the game world, multiple times more specific data is required that has no analogues in reality: states of animation trees, collider configurations, customization parameters, damage distribution logic.
The lower we descend through the levels of abstraction, the more chaotic the task looks, until we reach the level of bits and processor registers. The model of the virtual world cannot be as simple as we perceive reality. An attempt to build an "ideal academic model" at the upper level inevitably leads to critical complication of the software complex.
Ultimately, effective modeling of game space always leads to the creation of a monolithic structure of different-type but execution-flow-connected data. In the virtual world, decisions that seem to the player to be the height of realism are performed on the engine side by banal replacement of one object with another (for example, a whole vase in the frame is instantly replaced by a set of fragment meshes upon taking damage) or by simply launching the required animation.
Designers are incredibly inventive in methods of visual simplification of models, whereas programmers tend to complicate them with redundant code. At such moments, it is necessary to remember the fundamental KISS principle in time.

"Everything should be made as simple as possible, but not simpler."
— Albert Einstein

------------------------------

## The Comfort Zone Trap

According to my observations, leading engineers in the industry freely know several programming languages, use different IDEs, and work in several operating systems.

At the same time, young but talented developers often with inexplicable persistence lock themselves within one language, one familiar development environment, and one OS. They justify their reluctance to expand their engineering horizons by the imaginary "perfection" of the chosen toolkit.

It is important for such developers to realize: even if a newly learned language or paradigm is never useful in their current commercial practice, the very process of learning will restructure their thinking and allow them to reach a completely different professional level.

------------------------------

## The High or Low Level Dilemma

Programmers often split into two irreconcilable camps:

   1. The high-level approach: One must abstract from hardware, focus on pure business logic, entrusting all optimization and dirty work to the compiler and system libraries.
   2. The low-level approach: Code must be created with a rigid understanding of the platform that will execute it, that is, at the level of memory management, cache lines, and registers.

In the reality of game dev, both statements are absolutely true. A top-class architect must continuously balance between these approaches, choosing the most effective tool for a specific subsystem to achieve the optimal result.

------------------------------

## The Architect's Mistake

Programmers by nature are born architects. Where it is required to design a complex logical system, they perform the task brilliantly. Experience, background, and engineering culture force them to painstakingly erect a monumental and slender building of new architecture.
But the key problem is that the game as a whole is not a classic static information system. The game as a whole is a complex interactive animation working in real time. And animation and interactive response are not at all the area where the laws of building rigid monolithic systems apply.
The situation is complicated by the fact that the game designer at the early stages of prototyping physically cannot know exactly which gameplay solution will work. The designer always works in a cyclic, iterative mode: "made it — spun it in the runtime — looked — improved/discarded."
Instead of giving the designer a simple, "dirty," but fast solution (minimally viable prototype) for instant hypothesis testing, the programmer takes weeks to build a capital building of "ideal architecture," irretrievably losing precious production time.

------------------------------

## Neglect of the Special Case

Let us imagine a situation: a programmer writes performant code and sincerely cares about the target platform. They receive a gameplay task that, according to the game designer, seems unique, isolated, and "occurring in the game only once."
To save time, the developer changes their strict approach and solves it in a fast but architecturally inefficient way (for example, through direct references or redundant heap allocations). However, as the project develops, it suddenly turns out that in a new location — and what is much worse, right in the current game frame — hundreds and thousands of objects begin to use this "unique" script.

Game dev rule: Meet a task once — you will meet it many more times.

Solve any local task as if there will be a colossal number of similar objects in the scene. Always keep in mind and use Object Pools, Homogeneous Data Arrays, and the principles of Data-Oriented Programming (DOD).

------------------------------

## Exaggeration of the Importance of OOP

Modern multi-paradigm programming languages allow flexible combination of approaches. We can design a performant module in a functional or Data-Oriented style. However, Object-Oriented Programming (OOP) is so popularized that developers by default choose exclusively it, even where it is inappropriate.
With such a blind choice, compactness, hierarchy, and readability of code are put at the forefront, that is, exclusively convenience for the programmer themselves. Of course, there is sense in this, because according to the golden rule of code cleanliness: "Programs are written first of all for people who read them, and only secondly — for machines that execute them."
But game dev is an industry of extreme performance. A good engineer always soberly evaluates which programming paradigm is most effective for computations in a specific task, rather than writing the entire project according to OOP templates simply by habit.

## Exaggeration of the Importance of Encapsulation

At one time, my own understanding of OOP was shifted toward the idea that encapsulation is the main and unshakable pillar of this paradigm. Subsequently, I noticed that this distortion is massively widespread among developers.
What is noteworthy: by exaggerating the value of data isolation (encapsulation), programmers almost always underestimate the power and significance of polymorphism. This attitude looks maximally paradoxical, since it is dynamic polymorphism that is the most valuable, flexible, and applicable capability of OOP in the context of creating game systems.

------------------------------

## Exaggeration of the Importance of Compilation

I have been fortunate to work with outstanding programmers, many of whom were head and shoulders above me in certain areas. But even these brilliant and talented people periodically showed unjustified skepticism toward the dynamic style of programming and interactive development processes.

This is explainable: developers are accustomed to believing that the main sacred duty of the compiler is to catch the maximum of potential type or memory errors at the build stage and report them in time. This strong property of compiled languages deforms perception so much that any step away from static compilation toward dynamic interpretation is perceived by engineers as a fall into the abyss.

Such fear of interpreters looks very strange, given the historical fact: the vast majority of the world's AAA hits were created with deep integration of embedded scripting engines and interpreters (such as Lua, Lisp/Scheme, or custom bytecode machines).

The designer vitally needs a flexible runtime tool that allows instantly modifying logic and parameters right "on the fly," in a running game, without restarting and multi-minute recompilation of the project. Only in this way can quality timings and complex animation be quickly tuned.

------------------------------

## Preference for Exclusively GUI Tools

The industry has firmly established the opinion that a development tool is convenient and viable only when it has a graphical user interface (GUI). In practice, however, programmers spend a colossal amount of production time writing cumbersome custom GUI editors, which in the end still turn out to be inconvenient and overloaded.

A question for reflection: If designing a quality and ergonomic GUI is so simple, why does the vast majority of professional software in the world still sin with non-obvious and overloaded interfaces?

Programmers would never create redundant GUI windows if they themselves regularly used the utilities they write for game designers. To make a truly convenient graphical interface, it is necessary to write tons of accompanying system code: advanced search, contextual replacement, undo/redo systems, input validation, and so on. This eats up huge R&D resources.
In many internal pipelines, it is much more reasonable and efficient to use structured text configuration files (schemas). After all, for the text format, ideal graphical interfaces that have been developing for decades have already been created in the world — professional text editors (Emacs, VS Code, Vim), where all imaginable automation tools are available out of the box.

------------------------------

## Love of Visual Programming

Programmers and game designers often show amazing solidarity in the belief that graphical programming (visual logic graphs, blueprints) is an unambiguous path to success. The motives of the parties are different:

* For programmers, the presence of graphs is a legitimate way to fully delegate routine, uninteresting gameplay tasks to designers.
* For designers, this is long-awaited autonomy and total control over the process. With such a system, they feel confident, knowing that they can quickly assemble a working scene without the participation of a programmer.

In part, this works exactly like that. However, blind implementation of visual programming at the forefront has serious long-term consequences:

* The readability problem: As game mechanics scale and become more complex, a neat graph inevitably turns into unreadable "spaghetti" of connections.
* The performance problem: The code automatically generated under the hood of the graph often works critically slowly and inefficiently consumes RAM.
* The limitation problem: The toolkit of a visual language is always conceptually poorer, more rigid, and slower to develop than the syntax of text code.
* The collaboration problem: Graphs are most often serialized into heavy binary or complexly structured files. They cannot be normally merged in version control systems, which is why simultaneous collaboration of several people on one graph turns into a nightmare.

An effective alternative is the integration of a lightweight text interpreter (e.g., Lua), or a competent hybrid approach: the visual graph is used exclusively as a high-level finite state machine (FSM), and all the internal heavy logic of the nodes is written by programmers in text.

------------------------------

## Neglect of Documentation

The vast majority of programmers I have worked with sincerely disliked and did not know how to create quality technical documentation. Moreover, they spent a lot of energy inventing elaborate pretexts just to avoid writing it.
But practice shows a rigid regularity: as soon as an engineer overcomes this barrier and begins to systematically maintain quality technical documentation of their own modules, their professional and architectural growth accelerates multiple times.

If you strive for rapid professional development — train yourself to document architecture so that it is convenient, easy, and pleasant for the whole team to use.

## The Trap of "I Don't Want to Share What I Know Myself"

Sometimes experienced and strong developers bring colossal benefit to the project with their code but flatly refuse to spend their time on mentoring and teaching junior colleagues. The reasons for selfish concealment of expertise may be psychological, but I offer three theses for a radical change of attitude toward this issue:

   1. Team result: Creating a modern game is a strictly group process. The final quality of the product and the stability of the runtime will be determined by the qualification of the weakest and most inexperienced developer in the team. By teaching colleagues, you directly insure yourself against night debugging.
   2. Professional duty: Our personal baggage of knowledge and deep experience was formed not only thanks to perseverance but also because at one time the older generation of engineers openly shared knowledge with us. Teaching young specialists is our direct duty to preserve engineering culture.
   3. Feedback effect: By teaching others and breaking down complex concepts into simple components, the teacher themselves begins to understand their own subject of development more deeply and clearly.

------------------------------

## Unjustified Skepticism

In R&D practice, non-trivial tasks regularly arise that cannot be solved on the fly. This is an absolutely normal work process, but the reaction of specialists to such challenges differs radically.
Very often programmers (including senior status ones) turn on a defensive reaction and begin to unconditionally deny the very physical possibility of solving the task, without even attempting a primary analysis (RND). They are ready to spend days writing reports and proofs of why the task is "impossible." But the paradox is that in the end a beautiful engineering solution is always found.
In commercial development, it is multiple times more comfortable and effective to work with those people who are initially aimed at finding ways to solve, and not at collecting proofs of the impossibility of its implementation. Positively and flexibly minded developers show colossal productivity over the distance and develop as professionals many times faster than skeptics.

## Culture of Distrust and Fear of Mistakes

There are no people in the industry who never make mistakes. But there are people who are organically incapable of admitting their mistakes. The fear of dropping their inflated "authority" in the eyes of colleagues can be so great that it forces a person to persist in their delusion, making mistake after mistake and blocking the development of the project.
An engineer who openly admits: "I don't know this / I don't understand this right now" learns and grows at cosmic speed. They are not afraid to trust the expertise of colleagues in order to get to the bottom of the problem together and come to an elegant solution. Each of us needs to continuously learn to listen, hear, and trust our team.

------------------------------

## Lack of a Holistic View of the Problem

A classic situation: a programmer receives a bug report, looks into the subsystem code, instantly finds a local obvious cause, and hurries to send a quick bugfix to the repository.
In the absence of a holistic view of the system, this almost always leads to two outcomes:

   1. The original bug was not actually fixed, or was fixed only partially, for a narrow special case.
   2. The local fix violated the logic of related modules, which caused a cascade of new critical bugs in completely different parts of the project.

A superficial understanding of the problem is extremely subjective and never guarantees the architectural correctness of the fix. The developer is required to deeply delve into the problem at the level of data flows, and after its isolation — to conduct total, thorough testing of adjacent systems.

If after your edits the original code of the project begins to rapidly degrade (turning into a fragile and easily broken structure) — this is an alarming signal. It is worth seriously thinking about the root causes: either you have ceased to be interested in this project, or due to critical failures in the company's management processes, your work has turned into monotonous and burnout-inducing routine.

------------------------------

## On Which Day of the Week Does Monday Actually Begin?

One of the weakest sides of most engineers is the inability to accurately estimate the time costs of completing tasks (time management). Constant shifts in deadlines and chaotic re-estimations during work are a clear sign of specialist immaturity.
If you are aimed at serious and quality professional growth, introduce the following rigid techniques into your daily engineering practice:

* Having finished the workday, think in detail not only about what exactly you will do tomorrow but also how exactly you will implement it (which structures you will touch, which interfaces you will write).
* Being in the process of solving the current task, already in advance carefully look at the architectural requirements of the next task in the pool.
* Never artificially inflate deadlines "with a margin" for the sake of laziness and do not underestimate them for the sake of management approval — strive for maximum mathematical accuracy of estimation.
* Keep your word: try to fulfill your obligations to the team at any cost (within a reasonable balance of health).
* Keep a personal engineering diary. For each task set before yourself, strictly record four parameters:

1. Your initial predicted deadline.
   2. The real actual time spent by the fact of closing the task.
   3. The detailed reason for the delay (what exactly you did not take into account at the planning stage — unforeseen allocations, someone else's bug, bad refactoring).
   4. The physical volume of the result: the number of created/modified files, classes, pure lines of code.

A top-class engineer is visible not by beautiful reasoning but by how they plan their labor and how they relate to their technical obligations to the team. For a professional truly passionate about their craft, a fiery enthusiast, Monday always begins on Saturday (as in the famous utopia of the Strugatsky brothers).

------------------------------

## Given or Development?

If a specialist is completely satisfied with the current level of their competencies and passively waits only for regular extraction of financial profit from their once-developed skills — their professional development instantly and irrevocably stops.

True development requires colossal conscious efforts. There is no mental sense in them for someone who believes that they have already "achieved everything" and is ideal. Only those specialists who possess healthy engineering self-criticism and consider their current level of knowledge insufficient for the challenges of the future develop qualitatively and continuously.

Always orient yourself toward the vector of permanent development, and not toward the static given of the current status.

------------------------------

## Conclusion

Summing up, we can formulate the key set of goals and vectors for the continuous and effective professional growth of a game programmer:

   1. Expand your engineering horizons: Systematically study new paradigms, programming languages, and tools, stepping out of your comfort zone.
   2. Remember the client: Do absolutely everything in your power to make the operational work of game designers as productive, autonomous, and creative as possible.
   3. Maintain a balance of levels: Be able to jewelry-balance between high-level abstractions of logic and low-level optimization for the target hardware.
   4. Manage complexity: Never idealize the architectural model and do not complicate the solution beyond what is necessary. Sacredly observe the KISS principle.
   5. Focus on the essence: First and foremost, be responsible for designing clean and performant data of the model, and not for the volumes of code serving them.
   6. Choose the paradigm consciously: Soberly prioritize and combine the capabilities of languages (OOP, DOD, functional style) for a specific physical task.
   7. Save R&D resources: Design development tools efficiently and quickly, minimizing time costs for creating heavy custom GUIs where structured text can suffice.
   8. Write documentation: Create exhaustive, understandable technical documentation — both for crystal planning of architecture and for quick onboarding of colleagues.
   9. Share expertise: Do not lock knowledge on yourself, actively teach and mentor your team.
   10. Be solution-oriented: Always look for engineering ways to implement any complex problem, and do not waste energy on proving its unsolvability.
   11. Trust the team: Learn to listen to colleagues and trust their expertise — this is a fundamental marker of an engineer of the highest level.
   12. Delve holistically: Always deeply analyze the root causes of bugs at the scale of the entire system and subject fixes to total testing.
   13. Respect deadlines: Learn to precisely estimate work schedules and ironcladly fulfill your obligations to production.

But the most important quality of a good programmer is the unceasing orientation toward development. Such a specialist never gets stuck at what has been achieved; they are always organically open to the new and sincerely interested in deep personal and engineering growth.