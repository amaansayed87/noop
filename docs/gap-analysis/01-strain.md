# Gap 01: Strain (WHOOP live strain vs NOOP Effort)

**Summary.** The two methods differ at every stage. WHOOP smooths heart rate, weights intensity on a continuous curve and divides by a fixed physical ceiling. NOOP scores raw heart rate in step zones (Edwards) or with an exponential (Banister) and maps the total through a log curve whose denominator is fitted to WHOOP exports. Five of these differences are worth testing as upgrades.

- WHOOP method: [../algorithms/strain-live.md](../algorithms/strain-live.md)
- NOOP method: `android/.../analytics/StrainScorer.kt` and its Swift twin in `Packages/StrandAnalytics`

## Side by side

| Stage | WHOOP (on-phone live strain) | NOOP `StrainScorer` | Why it matters |
| --- | --- | --- | --- |
| HR cleaning | Rolling 25th percentile over ±30 samples; window cut at gaps ≥ 30 s | None: raw bpm goes straight into TRIMP | Optical spikes (cadence lock, loose strap) add load in NOOP but are mostly removed in WHOOP |
| Intensity floor | Below 30% HRR counts as nothing | Edwards: below 50% HRR counts as nothing. Banister: subtracts a sedentary rate at 10% HRR | Walking and light activity count very differently |
| Intensity weight | Smooth logistic in %HRR, ≈ 8/s at max | Edwards: 5 steps (1–5) at 50/60/70/80/90% HRR. Banister: x·0.64·e^(b·x) | Steps make a 1-bpm change at a zone edge jump the score |
| Time credit per sample | Up to 3 s | Up to 2 min (chosen for the 5/MG's ~30 s cadence) | WHOOP's cap would under-credit sparse 5/MG streams; NOOP's rule is deliberate there |
| Max HR | Profile lifetime max, raised when exceeded; estimate 204.4 − 0.273·age + sex + fitness offsets | Tanaka (208 − 0.7·age), or the observed 99.5th percentile over ≥ 600 samples | Every %HRR depends on it; Tanaka ignores sex and fitness |
| Normalisation | ÷ load of 24 h at max (a fixed physical ceiling) | ln(TRIMP+1) ÷ ln(D), D = 7201 (Edwards), fitted | WHOOP's scale is anchored to physiology; NOOP's is anchored to a fit |
| Output scale | 0–21 by monotone lookup | 0–100 "Effort" (0–21 × 100/21 for WHOOP comparisons) | Different curve shapes even after rescaling |
| Data floor | Scores with any data (warns under 3 samples) | ≥ 600 samples, or ≥ 20 spanning ≥ 10 min | NOOP is more conservative |

## Candidate upgrades to test

Each upgrade gets one hypothesis, tested on its own, so no result depends on another change.

| # | Change | Hypothesis | Risk |
| --- | --- | --- | --- |
| U1 | Robust HR pre-filter (rolling low percentile, gap-aware) before TRIMP | Cuts artifact-driven strain inflation without lowering real efforts by more than 0.3 strain | A low percentile biases sustained HR low; the median may be a better trade-off |
| U2 | Continuous intensity weight instead of Edwards steps | Removes zone-edge jumps; daily strain becomes a smooth function of HR | Changes every historical score; needs migration thinking |
| U3 | Physical-ceiling normalisation plus our own fitted monotone curve | Gives a scale that does not drift when the fitted D changes, and lines up with WHOOP's 0–21 | The curve must be our own fit, not WHOOP's table |
| U4 | Sex- and fitness-aware max-HR prior, raised by observed peaks | Better %HRR for users with no max test and few hard sessions | Needs a fitness-level input NOOP doesn't collect yet |
| U5 | Cadence-aware time cap: 3 s for dense 4.0 streams, NOOP's 2 min for sparse 5/MG | Stops one sample being credited across a short dropout on 4.0 | Must keep NOOP's 5/MG fix (#482) intact |

## How we test without a strap

1. **Python reference models.** Write clean-room Python versions of both pipelines in `research/` from the two specs. They live in notebooks, not in app code.
2. **Public wrist-HR data.** Use exercise and daily-life recordings with wrist optical HR and a chest-strap or ECG reference, such as PPG-DaLiA (UCI) and PhysioNet's wrist-PPG-during-exercise set. The reference lets us measure the artifact effect directly.
3. **Artifact injection.** Add synthetic spikes and dropouts to clean segments and measure how far each pipeline's strain moves.
4. **WHOOP-equivalent as proxy ground truth.** Because the live-strain method is now specified, we can measure how far NOOP's Effort sits from a WHOOP-style score on the same input, before owning a strap.
5. **Buy-gate criterion.** If U1–U3 together move NOOP measurably closer to the WHOOP-style score and reduce artifact sensitivity, buy a used WHOOP 4.0 and validate on real data under [VALIDATION_PROTOCOL.md](../VALIDATION_PROTOCOL.md).

## Already at parity: calories

NOOP's calorie model (`WorkoutDetector.kt` `Calories`) uses the same published equations as WHOOP's on-phone model: revised Harris–Benedict for resting energy (Roza & Shizgal 1984) and Keytel 2005 for exercise. It also uses the same 30% HRR switch point. NOOP goes further, with Keytel's VO2max-adjusted form and a 50% HRR gate for whole-day estimates. One detail is left to check. WHOOP's current model interpolates linearly from the resting rate at the switch point to the Keytel rate at max HR, rather than jumping to Keytel at the switch point.

## Out of reach: Recovery, sleep, HRV

These are computed on WHOOP's servers. The app receives finished values: Recovery score, RMSSD, resting HR and a calibrating flag. The app has no ML model files, and none of WHOOP's own code loads one, so sleep staging also runs on the server. Their gap analysis uses WHOOP's public statements ([whoop-public-methods.md](whoop-public-methods.md)) and published literature instead.
