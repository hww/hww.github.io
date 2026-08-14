# Game Development

Clarification for creative team

Valeria Pudova

August 16, 2022

### 🔗 Связанные материалы
Данный документ является архитектурно-художественным дополнением к основному техническому исследованию. Если вас интересует низкоуровневый разбор, спецификация байт-кода, устройство кастомной виртуальной машины и реверс-инжиниринг рантайма, обратитесь к первой (технической) части:
👉 **[Читать первую часть: Программирование игрового процесса На примере игры The Last of Us](https://hww.github.io/articles/nd_gfs_en/index)**

## Introduction

In "Game Developing - The Last of Us Case Study" I only look at the technical aspects - leaving the creative team's perspective aside. Therefore, in what follows will give brief excerpts on the subject.
Animations

...Although the action scenery is important, the core of the game is the characters: their characters, movements, behavior in the game situation, etc. That is why animation, its plasticity, accuracy, and naturalness is a fundamental issue for the studio.

Animation can express the inner state of the character, his thoughts, feelings; through movement we understand the character - just as it happens in life. Animation is the essence of visual impact and attraction. The plasticity-naturalness of the movement attracts attention.

Animation doesn't have to follow the real behavior of objects and creatures exactly. On the contrary, you have to look for the character traits, the most expressive features, in the movement, and enhance them even more, make them more noticeable, more significant. It is necessary to balance between realism and expressiveness.

The more fascinating interactive animation in a game, the more realistic its world looks. And in this context, the background detail inevitably recedes to the second, even third, plan. A good example is the game Playdead - in which the environment (despite the amazing art direction) is simple, minimalistic, and a lot of unique animation - and because of that everything feels real, alive.

This is why the creation of good games, first of all, you need skilled animators.

# Processes

There can be different types of animations in games. In addition to animations created in animation systems, or mocap, tween animations, procedural geometry or particle animations, and simulations of physical objects are used. The designer needs to put it all together - mixing animations in various combinations and proportions, in layers or as a whole, to play independently or synchronize, to control these processes by events.

It is important that the resulting behavior remains natural, even when different animation systems are at work - for example, the running animation must be fully consistent with the linear movement of the physical body. Only then will the animation look authentic and natural. In the process, the developer will have to reproduce the result many times, changing the parameters again and again - so that the final result is perfect.

We can think of the game - in general - as a complex interactive animation, consisting of tracks and clips, as well as the rules of their playback.  All the characters in it act independently, each has its own goal and ways of achieving this goal, the mechanisms of making their own and the group's decisions.  To make such a complex animation, you need to write hundreds of thousands of lines of code. And most of this code will be unique to each case.

That's why the programmers' first priority is to create a framework that makes it easy to create thousands of competitive processes and synchronize them. Such processes should be energy efficient - almost free for the target platform. At the same time, changes to the code should take effect immediately and not require compiling and restarting the game.

A variant of such technical solutions is described in my article "Game Developing - The Last of Us Case Study".
Result

When I see a new game, the first thing I pay attention to the animations - how diverse and natural are they? Do they serve only a utilitarian function, or are they the foundation of an organic world that is constantly changing? How does a character's animation change as he walks along a cliff ledge? What does a character do when he or she remains motionless for a long time? What role does animation play in the interactions, and what role does it play in the drama? All of these details are the main indicators of quality for me. Of course, other components of the game are also important, but it's the animations that form the core of the experience.

If a game doesn't have a lot of animation, it just looks boring. Often in such games, the core is the code - which is a very complicated path with predictably weak results. In such cases, I recommend that the team change the style of work, to implement in code only what is impossible or inconvenient to do with animations. As a result, this not only makes the game more appealing, but also saves time in the development process.

A good example is the game "Ori and the Blind Forest" In it the wonderful interactions take place inside a living animated world, creating the narrative background of the game: every single movement tells us something about this world, about the creatures that inhabit it.
Conclusions

As a final note to this article, I would like to point out that a project as big as The Last of Us on Playstation 3 required the following resources:

    16 programmers, two of them tools programmers
    20 game designers
    120 animation artists
    6,000 DC source files.

There's something to think about.
Tags

#games #gamedesing #gamedev #indiegames #gamedeveloper #creative #animations #pipeline #scripting #unity3d