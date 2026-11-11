---
slug: "benchmarks"
title: "Benchmarks"
description: "MIOSA's results in the independent ComputeSDK benchmarks. Burst time-to-interactive and DAX sandbox builds, every run, with links to the raw JSON."
section: "Optional"
order: 17035
---

MIOSA in the independent ComputeSDK sandbox benchmarks: burst startup and DAX builds. Every result is a committed JSON file, so each number can be checked against the repository.

Independent ComputeSDK benchmarks, updated Oct 23, 2026. Source: [github.com/computesdk/benchmarks](https://github.com/computesdk/benchmarks).

## Where MIOSA stands

- Burst startup (2026-10-23): #1 of 32. Median 57.09 ms, p95 68.81 ms, p99 72.20 ms, 100/100 succeeded. First measured 703.63 ms on 2026-09-04.
- DAX builds: #1 · Aug 21 & Aug 28. Best 38.4 s (#1 of 16 completed). Latest run (2026-10-23): #3 of 22 completed at 38.8 s. Fastest MIOSA build so far: 36.5 s (2026-10-16, #2).

On the other DAX runs MIOSA was #2, #3, #4 or #18, and on three dates the run failed.

MIOSA does not appear in the other ComputeSDK suites (browser, storage, AI gateway, and the retired sequential and staggered TTI tests), because it does not run those workloads.

Ranking is by the metric the suite uses at the top of its leaderboard: composite score for burst TTI (rewards speed and reliability together), and total build time for DAX. A provider with 0% success has no result to rank.

## Burst startup over time

Median milliseconds per run. Lower is better. The two failed runs (2026-08-21, 2026-09-10) are not plotted.

| Date | Median (ms) |
| --- | --- |
| 2026-08-28 | 698.6 |
| 2026-09-04 | 703.6 |
| 2026-09-11 | 222 |
| 2026-09-16 | 500.2 |
| 2026-09-18 | 399.1 |
| 2026-09-25 | 361.2 |
| 2026-10-02 | 163.4 |
| 2026-10-09 | 117.9 |
| 2026-10-16 | 94.8 |
| 2026-10-20 | 73.1 |
| 2026-10-23 | 57.1 |

## Leaderboards

### Burst TTI, 2026-10-23, all 32 providers

The latest run. Ranked by composite score. Success is out of 100 iterations. MIOSA's row is highlighted.

| Rank | Provider | Median (ms) | p95 (ms) | p99 (ms) | Success |
| --- | --- | --- | --- | --- | --- |
| 1 | miosa | 57.09 | 68.81 | 72.2 | 100% |
| 2 | isorun | 74.93 | 99.3 | 103.47 | 100% |
| 3 | promigence | 91.75 | 106.63 | 108.06 | 100% |
| 4 | gravixlayer | 108.71 | 116.39 | 116.99 | 100% |
| 5 | archil | 189.87 | 251.19 | 269.15 | 100% |
| 6 | arker | 316.5 | 335.32 | 343.75 | 100% |
| 7 | blaxel | 441.82 | 476.2 | 484.62 | 100% |
| 8 | superserve | 410.28 | 559.81 | 638.99 | 100% |
| 9 | createos | 466.18 | 576.2 | 578.24 | 100% |
| 10 | givemeanode | 501.73 | 694.41 | 696.06 | 100% |
| 11 | beam | 707.07 | 745.01 | 751.83 | 100% |
| 12 | cloudflare | 682.13 | 949.81 | 989.41 | 100% |
| 13 | tenki | 832.77 | 893.7 | 897.02 | 100% |
| 14 | buddy | 800.42 | 961.1 | 976.18 | 100% |
| 15 | vercel | 718.35 | 1334.62 | 1446.36 | 100% |
| 16 | declaw | 989.89 | 1268.52 | 1289.97 | 100% |
| 17 | modal | 1070.58 | 1204.89 | 1238.55 | 100% |
| 18 | mosaic | 1103.66 | 1343.16 | 1391.73 | 100% |
| 19 | daytona | 1094.64 | 1471.92 | 1597.83 | 100% |
| 20 | e2b | 1439.39 | 1680.93 | 1718.25 | 100% |
| 21 | tensorlake | 2268.59 | 2446.51 | 2456.54 | 100% |
| 22 | upstash | 1889.6 | 3058.58 | 3549.73 | 100% |
| 23 | novita | 2105.94 | 3169.2 | 3305.1 | 100% |
| 24 | microsandbox | 2390.26 | 2941.57 | 2961.64 | 100% |
| 25 | sail | 2860.14 | 3005.42 | 3089.63 | 100% |
| 26 | sandbox0 | 2836.82 | 3793.54 | 4183.35 | 100% |
| 27 | cloud-run | 3119.37 | 4402.74 | 4562.89 | 100% |
| 28 | freestyle | 3237.49 | 4707.02 | 4754.28 | 100% |
| 29 | codesandbox | 33771.52 | 55486.8 | 57741.31 | 100% |
| 30 | run-cloud | 27851.9 | 66122.11 | 76315.27 | 100% |
| - | hopx | - | - | - | 0% |
| - | northflank | - | - | - | 0% |

Two providers (hopx, northflank) recorded no successful iteration, so they have no median to rank.

### Burst TTI, 2026-10-09, all 30 providers

MIOSA's first #1. Ranked by composite score. Success is out of 100 iterations. MIOSA's row is highlighted.

| Rank | Provider | Median (ms) | p95 (ms) | p99 (ms) | Success |
| --- | --- | --- | --- | --- | --- |
| 1 | miosa | 117.93 | 130.72 | 132.13 | 100% |
| 2 | isorun | 137.98 | 149.15 | 152.78 | 100% |
| 3 | createos | 159.89 | 175.06 | 176.78 | 100% |
| 4 | archil | 190.16 | 389.57 | 439.69 | 100% |
| 5 | gravixlayer | 326.21 | 339.99 | 341.12 | 100% |
| 6 | arker | 351.41 | 366.06 | 367.49 | 100% |
| 7 | givemeanode | 434.51 | 603.96 | 628.11 | 100% |
| 8 | superserve | 524.51 | 644.05 | 726.64 | 100% |
| 9 | daytona | 646.02 | 847.83 | 873.93 | 100% |
| 10 | vercel | 665.97 | 822.52 | 905.89 | 100% |
| 11 | cloud-run | 618.06 | 907.9 | 962.07 | 100% |
| 12 | cloudflare | 635.45 | 731.49 | 844.36 | 99% |
| 13 | tenki | 863.7 | 917.71 | 947.62 | 100% |
| 14 | beam | 873.66 | 1129.49 | 1135.1 | 100% |
| 15 | modal | 1045.55 | 1260.25 | 1387.48 | 100% |
| 16 | declaw | 1099.21 | 1282.1 | 1298.2 | 100% |
| 17 | mosaic | 1174.99 | 1269.9 | 1348.19 | 100% |
| 18 | blaxel | 1054.22 | 1760.92 | 1809.14 | 100% |
| 19 | e2b | 1332.66 | 1601.8 | 1770.39 | 100% |
| 20 | buddy | 1394.11 | 1581.3 | 1619.79 | 100% |
| 21 | freestyle | 1437.56 | 1783.17 | 1854.07 | 100% |
| 22 | tensorlake | 2396.31 | 2579.3 | 2581.91 | 100% |
| 23 | sail | 2829.06 | 3942.07 | 4025.97 | 100% |
| 24 | sandbox0 | 4812.54 | 5768.08 | 5838.58 | 100% |
| 25 | upstash | 6780.05 | 9785.26 | 9881.62 | 100% |
| 26 | codesandbox | 8380.67 | 11486.07 | 11997.65 | 100% |
| 27 | run-cloud | 35454.87 | 61676.21 | 61699.58 | 42% |
| - | hopx | - | - | - | 0% |
| - | microsandbox | - | - | - | 0% |
| - | northflank | - | - | - | 0% |

Three providers (hopx, microsandbox, northflank) recorded no successful iteration, so they have no median to rank.

### DAX builds, 2026-08-28, all 30 providers

The run where MIOSA ranked first. Ranked by total build time among the 19 providers that completed. 11 providers recorded no successful build.

| Rank | Provider | Total (s) | Success |
| --- | --- | --- | --- |
| 1 | miosa | 39.3 | 100% |
| 2 | isorun | 39.5 | 100% |
| 3 | namespace | 40.2 | 100% |
| 4 | blaxel | 45.6 | 100% |
| 5 | createos | 49.2 | 100% |
| 6 | upstash | 50.4 | 100% |
| 7 | mosaic | 59.2 | 100% |
| 8 | arker | 62.6 | 100% |
| 9 | sail | 69.4 | 100% |
| 10 | vercel | 69.7 | 100% |
| 11 | e2b | 72.0 | 100% |
| 12 | tenki | 75.3 | 100% |
| 13 | codesandbox | 81.2 | 100% |
| 14 | tensorlake | 92.6 | 100% |
| 15 | modal | 97.7 | 100% |
| 16 | runloop | 114.4 | 100% |
| 17 | superserve | 119.8 | 100% |
| 18 | daytona | 141.3 | 100% |
| 19 | declaw | 172.6 | 100% |
| - | archil, beam, cloud-run, cloudflare, hopx, lightning, microsandbox, northflank, opencomputer, run-cloud, sandbox0 | - | 0% |

11 providers recorded no successful build, so they have no total to rank.

### DAX builds, 2026-10-23, all 33 providers

The latest run. Ranked by total build time among the 22 providers that completed. MIOSA ranked #3.

| Rank | Provider | Total (s) | Success |
| --- | --- | --- | --- |
| 1 | promigence | 31.6 | 100% |
| 2 | isorun | 37.3 | 100% |
| 3 | miosa | 38.8 | 100% |
| 4 | createos | 51.7 | 100% |
| 5 | gravixlayer | 52.6 | 100% |
| 6 | superserve | 54.5 | 100% |
| 7 | upstash | 55.0 | 100% |
| 8 | tenki | 55.3 | 100% |
| 9 | namespace | 61.5 | 100% |
| 10 | arker | 66.7 | 100% |
| 11 | novita | 70.4 | 100% |
| 12 | buddy | 72.2 | 100% |
| 13 | tensorlake | 73.1 | 100% |
| 14 | e2b | 75.0 | 100% |
| 15 | daytona | 79.5 | 100% |
| 16 | modal | 82.3 | 100% |
| 17 | sail | 88.9 | 100% |
| 18 | mosaic | 89.4 | 100% |
| 19 | beam | 91.4 | 100% |
| 20 | sandbox0 | 108.0 | 100% |
| 21 | cloudflare | 155.9 | 100% |
| 22 | declaw | 238.6 | 100% |
| - | archil, blaxel, cloud-run, codesandbox, freestyle, givemeanode, hopx, microsandbox, northflank, run-cloud, vercel | - | 0% |

11 providers recorded no successful build, so they have no total to rank.

### DAX builds, 2026-10-09, all 32 providers

MIOSA did not complete, so it has no rank here. 20 providers completed.

| Rank | Provider | Total (s) | Success |
| --- | --- | --- | --- |
| 1 | isorun | 34.5 | 100% |
| 2 | namespace | 40.1 | 100% |
| 3 | gravixlayer | 41.7 | 100% |
| 4 | givemeanode | 41.7 | 100% |
| 5 | blaxel | 45.9 | 100% |
| 6 | beam | 51.7 | 100% |
| 7 | createos | 52.8 | 100% |
| 8 | upstash | 54.7 | 100% |
| 9 | tensorlake | 59.3 | 100% |
| 10 | mosaic | 59.4 | 100% |
| 11 | arker | 64.2 | 100% |
| 12 | sandbox0 | 65.7 | 100% |
| 13 | daytona | 70.0 | 100% |
| 14 | e2b | 72.7 | 100% |
| 15 | vercel | 80.0 | 100% |
| 16 | tenki | 80.3 | 100% |
| 17 | sail | 102.5 | 100% |
| 18 | modal | 105.1 | 100% |
| 19 | superserve | 112.8 | 100% |
| 20 | declaw | 218.0 | 100% |
| - | miosa | - | 0% |
| - | archil, buddy, cloud-run, cloudflare, codesandbox, freestyle, hopx, microsandbox, northflank, opencomputer, run-cloud | - | 0% |

Providers with 0% success have no build time to rank.

## Method and reproduction

- **Independent and open source**: The benchmark code, runner and every result file are public. ComputeSDK states that sponsors cannot influence results. [github.com/computesdk/benchmarks](https://github.com/computesdk/benchmarks)
- **Same code, every provider**: The same open code against every provider, with fixed workloads and a fixed scoring ceiling. Each metric reports min, max, median, p95 and p99, and burst TTI also reports a composite score that rewards speed and reliability together.
- **Automated**: Runs execute on GitHub Actions on Namespace runners, and every result is committed to the repository as JSON. The repository lists the schedule as daily at 5pm Pacific, 00:00 UTC. In practice the burst TTI history is daily through 2026-07-13 and then weekly on Fridays, with extra runs on 2026-09-10 and 2026-09-16.
- **Live leaderboards and methodology**: Live leaderboards and the full methodology are published by ComputeSDK. [computesdk.com/benchmarks](https://www.computesdk.com/benchmarks)

```bash
git clone https://github.com/computesdk/benchmarks
cd benchmarks

# MIOSA's burst TTI row, latest run
jq '.results[] | select(.provider=="miosa")' results/burst_tti/2026-10-23.json

# Every provider, sorted by median
jq -r '.results[] | "\(.summary.ttiMs.median)\t\(.provider)"' results/burst_tti/2026-10-23.json | sort -n
```

- [Burst TTI results](https://github.com/computesdk/benchmarks/tree/main/results/burst_tti): Every dated burst time-to-interactive result file.
- [DAX results](https://github.com/computesdk/benchmarks/tree/main/results/sandbox-dax): Every dated DAX sandbox build result file.
