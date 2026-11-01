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

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | --------- | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | | | | | |
| A2 | Average load | 3 | | | | | |
| A3 | Stress | 5 | | | | | |
| A4 | Spike | 5 | | | | | |
| A5 | Soak | 2 | | | | | |

## Part B
