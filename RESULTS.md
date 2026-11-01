# K6 lab results

## Part 0

- Step 0.1, `http_req_duration`: 254.64 ms (avg, min, median, max, p(90), and p(95) are equal for this single request). One iteration and one request; the status check passed.
- Step 0.2: 80 requests at 3.864824 requests/s, with p(95) = 258.06 ms. All 80 checks passed. Each user waits for the HTTP response and then sleeps for one second before requesting again. The average iteration took about 1.29 seconds, including initial connection overhead, so five users cannot sustain five requests per second.
- Step 0.3: Five users shared exactly 20 iterations and made 20 requests. All 20 checks passed. Unlike the duration-based run, this run ended after the specified amount of work was completed.
- Step 0.4: All three checks passed: status 200, a pizza name, and every ingredient vegetarian.
- Step 0.5: The request without a token returned the expected 401. The check passed and `http_req_failed` was 0.00%, because `http.expectedStatuses(401)` treats this refusal as the correct response.
- Step 0.6: `k6 run 06-thresholds.js` exited with **0**. All three thresholds passed: checks = 100.00%, failures = 0.00%, and p(95) = 367.35 ms < 1500 ms. `k6 run -e P95=300 06-thresholds.js` exited with **99**: p(95) = 363.49 ms exceeded 300 ms, while the checks and failure-rate thresholds still passed.
- Step 0.7: With 3 users for 30 seconds, all requirement rules passed: p(95) = 371.28 ms < 1000 ms, p(99) = 374.79 ms < 1500 ms, and `http_req_failed` = 0.00% < 1%. The exit code was **0**. I changed only the p(95) requirement from 1000 ms to 1 ms using `-e P95=1`; p(95) = 364.61 ms failed that rule and the exit code was **99**. The p(99) and failure-rate rules still passed. The script keeps the original requirement as its default. I displayed p(99) using `--summary-trend-stats 'avg,min,med,max,p(90),p(95),p(99)'`.
- Step 0.8: The exclusions used lowercase `pepperoni` and `knife`, but `explore.js` showed the API's exact names are `Pepperoni` and `Knife`. The matching is case-sensitive, so the original exclusions did not prevent those choices. In the original 5-user, 20-second run, 14 of 75 pizzas failed the pepperoni check and 24 of 75 failed the knife check; only 187 of 225 checks passed (83.11%). These failure counts can overlap on the same pizza. Every HTTP status was still 200, so status-only checks would have missed the wrong recommendations. A single manual request might randomly return neither excluded item and hide the bug. Repeated requests sampled enough random choices to expose it. After fixing the request's capitalization, the same load produced 75 pizzas and **225/225 passing checks (100.00%)**. Both runs exited with 0 because this script has checks but no thresholds.

## Part A results

Runs were made on 1 November 2026 using K6 v2.3.0. Response times below are in milliseconds, rounded to two decimal places from the measured metrics; K6's terminal display truncates some final decimal places. Req/s is the overall `http_reqs` rate. Users is the highest configured number of virtual users.

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | --------- | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.498363 | 322.94 ms | 324.59 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.346876 | 361.39 ms | 374.71 ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.174993 | 357.47 ms | 464.42 ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 0.991417 | 348.40 ms | 366.53 ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.998389 | 362.25 ms | 398.52 ms | 0.00% | 100.00% |

The overall p(95) decreased from 361.39 ms in A2 to 357.47 ms in A3 (about 1.1% lower), while the request rate increased from 1.346876 to 2.174993 requests/s. Both tests had 0.00% failed requests and 100.00% passing checks.

In A4, the spike ramp begins at 30 seconds, reaches 5 users at 35 seconds, holds until 65 seconds, and returns to 1 user at 70 seconds. The report shows no sustained response-time rise during the spike. Request samples give p(95) = 358.03 ms before the spike (0-30 s), 343.89 ms during the 5-user hold (35-65 s), and 339.97 ms in the first 30 seconds after ramp-down (70-100 s). No recovery delay was observed. The first complete 10-second interval after ramp-down was already within the earlier range, so the report supports recovery within its 10-second sampling resolution, rather than an exact sub-second recovery time. There were no failed requests.

In A5, the full 30-second ramp-up, 10-minute hold at 2 users, and 30-second ramp-down completed. I compared equal two-minute windows within the steady hold: 30-150 seconds and 510-630 seconds. The early window had 122 requests, average duration 275.98 ms, p(95) 347.90 ms, and 0 failed requests. The late window had 125 requests, average duration 277.16 ms, p(95) 355.97 ms, and 0 failed requests. Late p(95) was 8.07 ms (about 2.3%) higher. The report shows fluctuations, including a temporary rise around minutes 7-8 that later fell, rather than a continuous increase. All five Part A runs exited with 0 and passed every configured threshold.

## Part B

1. I filled in the Part A results table above with the measured request rate, overall p(95), maximum duration, request failure rate, and check success rate for all five runs. Every Part A run had 0.00% failed requests and 100.00% passing checks.

2. In Step 0.2, five virtual users made 80 requests at 3.864824 requests/s. Each user waited for its HTTP response and then slept for one second. The average iteration took about 1.29 seconds, including initial connection overhead. Because each user starts its next request only after finishing that iteration, the rate stays below five requests/s. Five users describes concurrency, not five requests arriving every second.

3. I did not observe an increase in overall p(95) from average load to stress. It decreased from 361.39 ms at 3 users to 357.47 ms at 5 users, about 1.1%, while throughput increased from 1.346876 to 2.174993 requests/s. Both runs passed the home-page and recommendation thresholds and had no failed requests. These small loads appear to be inside the system's operational envelope. They do not identify its upper limit or prove how it would behave under much heavier load. The small difference could come from random response variation, network conditions, or other users on the shared site.

4. I observed no sustained latency rise during the spike and no detectable recovery delay afterward. The five-user hold was at 35-65 seconds and ramp-down to one user finished at 70 seconds. Before the spike, p(95) was 358.03 ms; during the hold it was 343.89 ms; during 70-100 seconds it was 339.97 ms. The first complete 10-second interval after ramp-down was already within the earlier range. I can therefore say there was no delay visible at the report's 10-second resolution, but I cannot claim an exact sub-second recovery time from this small sample.

5. The late soak window was slightly slower: p(95) increased from 347.90 ms in seconds 30-150 to 355.97 ms in seconds 510-630, about 2.3%. The average changed only from 275.98 ms to 277.16 ms. Both windows had 0.00% failed requests, as did the whole run. The report shows a temporary bump followed by lower latency, so I did not see a clear progressive slowdown or rising error rate. This short run cannot rule out long-term defects. A soak test lasting several hours could reveal a memory leak that gradually exhausts available memory and eventually causes slow responses or failures.

6. An average can hide a small group of very slow requests. A p(95) threshold checks the response time experienced by the slower part of the user population: at least 95% of requests must be below the chosen limit. In A2, for example, the average was 279.59 ms but p(95) was 361.39 ms. A requirement about 95% of requests is therefore checked more directly with p(95) than with the average.

7. First, an internet run includes changing network distance, routing, congestion, and connection setup; a controlled local network can have much lower and more stable latency. Second, QuickPizza is shared with other learners, so its competing workload is outside my control; a dedicated lab can control background traffic, server resources, and configuration. Either difference can change the measurements even when the scripts are identical.

8. Hundreds of virtual users could overwhelm a shared site, interrupt other people's use, and create unexpected resource costs. I need ownership or written permission before testing at that scale. Calling traffic a test does not give me permission to disrupt someone else's service. For QuickPizza, I followed the limit of five users and used only two for the longer soak test.

9. For an everyday messaging app, my vague requirement is: "My chat list should load quickly." A testable version is: "In the app's controlled test environment, with five virtual users repeatedly requesting the conversation-list API for two minutes and pausing one second between iterations, 95% of requests must finish within 1000 ms, 99% within 1500 ms, and fewer than 1% may fail." For a script that requests only that API, these K6 options check the requirement:

   ```javascript
   export const options = {
     vus: 5,
     duration: '2m',
     thresholds: {
       http_req_duration: ['p(95)<1000', 'p(99)<1500'],
       http_req_failed: ['rate<0.01'],
     },
   };
   ```
