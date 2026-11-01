# K6 lab results

## Part 0

- Step 0.1, `http_req_duration`: 254.64 ms (avg, min, median, max, p(90), and p(95) are equal for this single request). One iteration and one request; the status check passed.
- Step 0.2: 80 requests at 3.864824 requests/s, with p(95) = 258.06 ms. All 80 checks passed. Each user waits for the HTTP response and then sleeps for one second before requesting again. The average iteration took about 1.29 seconds, including initial connection overhead, so five users cannot sustain five requests per second.
- Step 0.3: Five users shared exactly 20 iterations and made 20 requests. All 20 checks passed. Unlike the duration-based run, this run ended after the specified amount of work was completed.

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | --------- | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | | | | | |
| A2 | Average load | 3 | | | | | |
| A3 | Stress | 5 | | | | | |
| A4 | Spike | 5 | | | | | |
| A5 | Soak | 2 | | | | | |

## Part B
