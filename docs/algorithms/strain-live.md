# Live strain (on-phone)

**Status:** spec, from static analysis of WHOOP Android 5.474.0 (Oct 2026) · **Confidence:** high for the steps and constants, medium where marked.

The WHOOP Android app calculates strain on the phone during recorded activities. It does not work like the zone-based TRIMP that NOOP uses. Each heart-rate sample is smoothed, turned into a fraction of heart-rate reserve, weighted on a steep logistic curve, and summed. The total is divided by the load of a theoretical full day at max heart rate, and a monotone curve maps that 0–1 value onto the familiar 0–21 scale.

The server's Strain record has the same two fields: a 0–1 `rawScore` and a 0–21 `score`. That suggests the phone runs the same method as WHOOP's backend, or a close copy of it.

**Attribution and status:** derived from a decompile of the official app, under the "facts vs code" rule in [CONTRIBUTING.md](../CONTRIBUTING.md). This page describes behaviour and parameters only. No WHOOP code, literals table or assets are reproduced. Every number here is an **unvalidated candidate** until it is checked against independent data under [VALIDATION_PROTOCOL.md](../VALIDATION_PROTOCOL.md). Nothing here may back a shipped metric before then.

## Inputs

| Input | Notes |
| --- | --- |
| Heart-rate samples | (timestamp ms, bpm), about 1 Hz |
| Resting HR | from the user profile |
| Max HR | the profile's lifetime max, raised if the session goes higher |
| Age | days since birth ÷ 365.242, clamped to 12–120 years |
| Sex, fitness level | used for the estimated-max formula below |
| Carry-over state | last processed timestamp plus the most recent ~60 samples, so a session is scored in batches |

## Steps

1. **Merge and order.** Combine the carried-over samples with the new batch. Drop exact duplicates and sort by time.
2. **Smooth: rolling 25th percentile.** Each sample gets a window of 30 samples before and 30 after.
   - If two neighbouring samples in the window are ≥ 30 s apart, the window is cut at that gap so it never spans a dropout.
   - The smoothed value is the 25th percentile of the window (linear interpolation).
   - The last 30 samples of a batch wait until the next batch arrives, so live strain lags by about 30 s.
   - A low percentile throws away short upward spikes, which are the typical optical-sensor artifact, but it biases sustained HR slightly low.
3. **Intensity.** r = (HR − resting) ÷ (max + 3 − resting), clamped to 0–1. **Anything at or below r = 0.30 counts as zero.**
4. **Time weight.** Each sample counts for the seconds until the next sample, **capped at 3 s**.
5. **Load per second.** load = 16 ÷ (1 + e^((1 − r) / 0.15)), times the time weight. That's 8 per second at r = 1, about 0.55 at r = 0.5 and about 0.16 just above the 0.30 floor. Load climbs steeply with intensity, roughly exponentially below max.
6. **Raw strain.** Sum the load and divide by 691,200, which is 8 × 86,400: a full day spent at max HR. Clamp to 0–1.
7. **Scale to 0–21.** A fixed, monotone lookup in 0.1 steps. Anchor points (rounded):

   | Raw | Strain |
   | --- | --- |
   | 0.0036 | 10 |
   | 0.0087 | 14 |
   | 0.020 | 18 |
   | 0.030 | 20 |
   | 0.70 | 21 |

   The curve is close to logarithmic through the middle and almost flat at the top. Going from 20 to 21 needs about 23 times more raw load than reaching 20. That matches WHOOP's public "each point is harder" claim.

## Estimated max HR

An estimate is reported alongside the score:

```latex
HR_{max} \approx 204.4 - 0.273 \times age + s + f
```

- **s (sex):** male −2.0, female 0, non-binary −1.0.
- **f (fitness level):** collegiate 0, serious enthusiast −0.7, tactical/professional −3.0, recreational enthusiast −3.3, inactive −3.7.

The score itself uses the profile max, raised to the session max if exceeded. The estimate seems to be used for display and sanity checks: resting HR must sit below it plus 10.

## Open questions

- **Medium confidence:** the scale lookup decompiles badly (the loop's control flow is mangled). The reading above, "first bin whose lower edge ≤ raw < next edge", is the only one consistent with the table, but bytecode should confirm it.
- Is a session's strain the sum of per-batch raw values, or is the whole session recomputed each time? The carry-over state allows either.
- How does day strain combine workouts with the rest of the day on the server? It is probably the same function over the whole day's HR, but nothing on the phone confirms that.
- Muscular load: WHOOP's marketing mentions it, but this path uses heart rate only. The carry-over state has accelerometer fields that are never read here.
