# Real-Time Simulation Specifics in Unity3D

**Valeriya Pudova**
December 4, 2020

---

## Introduction

An analysis of the specifics of programming real-time processes in a discrete simulation system using Unity3D as an example.

The purpose of this document is to confirm or refute that the `Update` or `FixedUpdate` methods do not guarantee the accuracy of the internal clock (`Time`). The document should answer two questions:

1. Can the `FixedUpdate` method be guaranteed to be called at equal time intervals?
2. Can `FixedUpdate` be called a predictable number of times over a certain real-time interval?

Below, the questions are formulated more concretely, and answers are given to them.

---

## Is `FixedUpdate` called in real time?

If `fixedDeltaTime` equals 10 ms, will the method be called strictly every 10 milliseconds?

**No, it will not.**
The physics update time (TFPU — time for physics update), depending on the situation, may be less than or greater than `fixedDeltaTime`.

Will it be guaranteed that over real time `T`, the `FixedUpdate` method will be called according to the expression:

```
Nfu = T / Fdt
```

where:
- `Nfu` — the number of `FixedUpdate` calls
- `T` — real time
- `Fdt` — the value of `fixedDeltaTime`

**No, it will not.**
In a situation where TFPU is greater than `fixedDeltaTime`, the physics engine will not be able to reach time `T`, even if it loads the processor exclusively. Since physics tries to catch up with real time and continuously executes `Nfus` times in a row while `fixedTime` is less than `T`, this leads to freezing of rendering for an indefinite time (*Frame-skipping mode*).

To minimize this effect, there is a `maximumDeltaTime` parameter. It does not allow physics to execute in a row longer than a specified value. When the product `fixedDeltaTime * Nfus` reaches a value greater than or equal to `maximumDeltaTime`, physics "defers" the update, skipping a fragment of real time (*Wall of molasses mode*).

In other words, **time in the virtual world cannot be guaranteed to be synchronized with real time**. This is clearly stated in the Unity3D manual:

> *If frame will take longer than specified value, then less physics and other fixed frame rate updates will be performed. That will temporarily slow down the gameplay, but will allow frame rate to catch up, avoiding nasty stuttering.*

---

## Can `Time.time` be desynchronized from `Time.fixedTime`?

**It cannot.** One depends on the other, which can be described by a simplified formula:

```
Tvn = Tvn-1 + Fdt * Nfus
```

where:
- `Tvn` — time in frame `n`
- `Tvn-1` — time in frame `n-1`
- `Fdt` — the value of `fixedDeltaTime`
- `Nfus` — the number of `FixedUpdate` calls between `Tn` and `Tn-1`

---

## Conclusion

Unity3D does not manipulate the `timeScale` value and does not slow down time. Instead, when TFPU is greater than `Fdt` and if `Fdt * Nfus` exceeds `maximumDeltaTime`, Unity3D forcibly interrupts the physics update, giving preference to other operations (e.g., rendering). This indirectly slows down time inside Unity3D.

- The system **cannot guarantee** that the `FixedUpdate` method will be called at equal time intervals.
- The system **cannot call** `FixedUpdate` a predictable number of times over a certain real-time interval.

When the system experiences a shortage of computational resources, one of two possible scenarios is used (sometimes called):

- **Frame-skipping mode** — stuttering of rendering at normal animation speed.
- **Wall of molasses mode** — slowing down animations of physical objects at normal rendering speed.

The same is true for other engines.

---

## References

- [1] [Timesteps and Achieving Smooth Motion in Unity](https://www.google.com/url?q=https://www.kinematicsoup.com/news/2016/8/9/rrypp5tkubynjwxhxjzd42s3o034o8&sa=D&source=editors&ust=1786723807161712&usg=AOvVaw1PwaMuaDVtfwChZXjupB9v)