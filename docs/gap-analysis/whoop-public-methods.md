# What WHOOP says publicly about its methods

Baseline for the gap analysis: what WHOOP and third parties have published, before we read any decompiled code. Each claim is tagged with how much to trust it. Anything the APK study confirms or contradicts gets noted here with a link to the spec page.

Trust levels: **WHOOP** = stated by WHOOP staff or WHOOP's own site · **Study** = peer-reviewed paper · **Third-party** = blogs and reviews, unverified.

## HRV and Recovery

| Claim | Source | Trust |
| --- | --- | --- |
| HRV is RMSSD computed in overlapping 5-minute windows, stepped every 30 s, across the whole sleep | [WHOOP community post](https://community.whoop.com/t/hrv-extreme-spikes/6472/31) | WHOOP |
| Wake periods are removed, then windows are combined as a weighted average | same | WHOOP |
| Windows likely to be slow-wave sleep, and windows later in the night, get higher weights | same | WHOOP |
| Older method: HRV from the last 5 minutes of slow-wave sleep, best-quality segment if several | [WHOOP podcast 194](https://www.whoop.com/ie/en/thelocker/podcast-194-understanding-heart-rate-variability-and-its-performance-potential/), [RoadTrailRun 2017 review](https://www.roadtrailrun.com/2017/05/whoop-strap-20-review-novel-approaches.html?m=1) | WHOOP (outdated) |
| Recovery combines HRV, resting heart rate and sleep | WHOOP podcast 194 | WHOOP |
| Recovery uses ln(RMSSD) against a personal 14-day rolling baseline | [Kygo blog](https://www.kygo.app/post/whoop-stress-score-recovery-explained) | Third-party |

**Questions for the APK study:** window weights; how wake and artifacts are excluded; whether R-R cleaning happens on the phone or on the strap; baseline length; how RHR and respiratory rate enter Recovery.

## Strain

| Claim | Source | Trust |
| --- | --- | --- |
| 0–21 scale, for the day and for single workouts | [How WHOOP Strain works](https://www.whoop.com/kr/en/thelocker/how-does-whoop-strain-work-101/) | WHOOP |
| Logarithmic: each point is harder to earn than the last | same | WHOOP |
| Bands: 0–9 light, 10–13 moderate, 14–17 high, 18–21 all-out | [Strain Coach](https://www.whoop.com/thelocker/strain-coach/) | WHOOP |
| Built from cardiovascular load plus muscular load | How WHOOP Strain works | WHOOP |
| Core is weighted time in personalised HR zones based on max HR | [Sensai guide](https://www.sensai.fit/blog/whoop-strain-score-guide) | Third-party |
| Exact formula is not published | Sensai guide | Third-party |

**Questions for the APK study:** zone boundaries (%HRmax or %HR reserve); the log mapping; how muscular load comes from the accelerometer; whether any of this is computed on the phone.

## Sleep staging

| Claim | Source | Trust |
| --- | --- | --- |
| High agreement with polysomnography in a University of Arizona study (J Clin Sleep Med); sleep HR precision error about 1 bpm, n = 32 | [How well WHOOP measures sleep](https://ww2.whoop.com/thelocker/how-well-whoop-measures-sleep) | WHOOP |
| Best of three devices vs PSG for total, light and deep sleep; worst for REM | [JMIR mHealth 2024 systematic review](https://mhealth.jmir.org/2024/1/e52192/PDF) | Study |
| WHOOP 4.0 included in a 6-device, 62-adult PSG comparison | [SLEEP Advances 2025](https://academic.oup.com/sleepadvances/article/6/2/zpaf021/8090472) | Study (WHOOP-specific results not yet read) |
| Later algorithm update: +3% sleep/wake, +7% staging accuracy | How well WHOOP measures sleep | WHOOP |

**Questions for the APK study:** whether staging runs on the phone (a model file in the assets would be the giveaway) or in the cloud; what features it takes in (HR, HRV, motion, temperature).

## Unknowns to settle from the APK

- PPG sampling rate and any on-device signal filtering (no public source found).
- Packet formats beyond the 96-byte realtime frame documented by whoop-reader.
- Whether R-R intervals arrive already cleaned from the strap firmware.
