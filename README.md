# Concurrent Reservation Service
## CEN4025 - Assignment 7: Load Testing and Scalability
### Project Overview

The Concurrent Reservation Service is a Java Spring Boot REST API that manages appointment availability and reservations.

Assignment 7 evaluates the application's performance, scalability, and correctness under concurrent HTTP workloads.

The project includes:

- Sequential baseline testing
- Concurrent load testing
- Throughput and scalability measurements
- Application resource monitoring
- Performance limitation investigation
- Availability retrieval optimization
- Post-optimization testing
- Automated and functional verification

The repository contains the application source code, automated tests, and PowerShell scripts used to reproduce the testing.

---

## 1. Technologies and Environment

| Component | Configuration |
|---|---|
| Programming Language | Java 26.0.2 |
| Framework | Spring Boot 4.1.1 |
| Build Tool | Maven Wrapper 3.9.16 |
| IDE | IntelliJ IDEA |
| Operating System | Windows 11, 64-bit |
| System Memory | Approximately 31.7 GiB |
| Testing Shell | Windows PowerShell 5.1 |
| HTTP Clients | Invoke-RestMethod and .NET HttpClient |
| Application Port | 8080 |
| Tested Endpoint | GET /api/appointments |

All tests were executed locally against the Spring Boot application running on localhost.

---

## 2. Build and Run Instructions

### Prerequisites

A compatible Java Development Kit must be installed and configured.

The repository includes Maven Wrapper, so a separate Maven installation is not required.

### Run Automated Tests

From the project root:

```powershell

.\mvnw.cmd clean test

```

Expected final test results:

```text

Tests run: 13, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS

```

### Start the Application

```powershell

.\mvnw.cmd spring-boot:run

```

Wait for Spring Boot to start successfully and for embedded Tomcat to initialize on port 8080.

### Verify the API

Open another PowerShell terminal:

```powershell

Invoke-RestMethod -Uri "http://localhost:8080/api/appointments" -Method Get

```

The final verification returned four appointment slots:

- SLOT-001

- SLOT-002

- SLOT-003

- SLOT-004

---

## 3. Load Testing Scripts

The `scripts` directory contains the PowerShell scripts used during the assignment.

Start the Spring Boot application before running these scripts.

Execute all commands from the repository root.

### Medium Load Test

```powershell

.\scripts\medium-load.ps1

```

Runs 20 background jobs with 2 requests per job.

Total: 40 requests.

### High Load Test

```powershell

.\scripts\high-load.ps1

```

Runs 20 background jobs with 5 requests per job.

Total: 100 requests.

### Concurrent Correctness Validation

```powershell

.\scripts\validate-load.ps1

```

Runs 20 background jobs with 2 requests per job.

Validates the appointment responses returned by the application.

Total: 40 requests.

### Throughput Test

```powershell

.\scripts\throughput-test.ps1

```

Measures request throughput using different numbers of PowerShell background jobs.

### Lightweight Scalability Test

```powershell

.\scripts\scalability-test.ps1

```

Measures throughput at concurrency levels of 1, 5, and 20.

Each configuration executes 100 HTTP GET requests using asynchronous .NET HTTP operations.

### Repeat Scalability Test Three Times

```powershell

1..3 | ForEach-Object {

    Write-Host "`n=== REPEAT TEST $_ ==="

    .\scripts\scalability-test.ps1

}

```

This repeats the benchmark to evaluate measurement variability.

---

## 4. Sequential Baseline Results

The initial baseline executed 20 sequential HTTP GET requests.

| Metric | Result |
|---|---:|
| Total Requests | 20 |
| Successful Requests | 20 |
| Average Response Time | 99.484 ms |
| Minimum Response Time | 64.37 ms |
| Maximum Response Time | 188.28 ms |
| Error Rate | 0% |

All sequential requests completed successfully.

---

## 5. Concurrent Load Testing Results

| Workload | Requests | Successful | Average Response Time | Error Rate |
|---|---:|---:|---:|---:|
| Initial Concurrent Load | 20 | 20 | 1358.756 ms | 0% |
| Medium Load | 40 | 40 | 790.612 ms | 0% |
| High Load | 100 | 100 | 374.319 ms | 0% |
All tested workloads completed without observed request failures.

The tests used PowerShell background jobs. Process initialization and scheduling overhead may influence measured response times.

The results should not be interpreted as a controlled server-side latency scaling curve.

---

## 6. Throughput Testing Results

The background-job throughput test executed 100 requests per configuration.

| Background Jobs | Successful Requests | Duration | Throughput |
|---|---:|---:|---:|
| 1 | 100 | 2.544 sec | 39.31 req/sec |
| 5 | 100 | 7.072 sec | 14.14 req/sec |
| 20 | 100 | 32.303 sec | 3.10 req/sec |

All requests succeeded.

Measured throughput decreased as background-job concurrency increased.

PowerShell background-job startup and scheduling overhead are potential contributors to this behavior.

---

## 7. Lightweight Scalability Baseline

The lightweight scalability script used asynchronous HTTP requests to reduce background-job overhead.

| Concurrency | Requests | Duration | Throughput |
|---|---:|---:|---:|
| 1 | 100 | 0.079 sec | 1266.38 req/sec |
| 5 | 100 | 0.019 sec | 5155.99 req/sec |
| 20 | 100 | 0.015 sec | 6631.70 req/sec |
All 300 requests completed successfully with a 0% error rate.

These measurements represent short-duration localhost tests and should not be interpreted as sustained production throughput.

---

## 8. Application Resource Monitoring

Resource measurements were collected before and after a high-load test.

| Resource | Before | After |
|---|---:|---:|
| Cumulative CPU Time | 24.594 sec | 24.922 sec |
| Working Set Memory | 93.53 MB | 95.54 MB |
| Thread Count | 53 | 53 |

Observed differences:

- CPU time increased by 0.328 seconds.
- Memory increased by approximately 2.01 MB.
- Thread count remained unchanged.

The accompanying high-load test completed 100 requests successfully with an average response time of 289.145 ms.

These measurements represent before-and-after snapshots rather than peak resource utilization.

---

## 9. Performance Limitation Investigation

The original `ReservationService.getAvailableSlots()` implementation checked each appointment slot against the reservations collection.

For every available slot candidate, the service scanned existing reservations to determine whether the slot was reserved.

The original approach had approximate time complexity:

O(S * R)

Where:

- S = Number of appointment slots

- R = Number of reservations

This repeated scanning could become increasingly expensive as the number of appointment slots and reservations grows.

The method also used a synchronization lock to protect shared reservation data.

The nested collection search was identified as a potential algorithmic limitation, not a conclusively demonstrated cause of the observed throughput behavior.

---

## 10. Performance Optimization

The `ReservationService.getAvailableSlots()` method was optimized to build a set of reserved appointment identifiers once per request.

The service then uses set membership checks to filter available appointment slots.

The optimized approach has approximate expected time complexity:

O(S + R)

The modification:

- Eliminates repeated scans of the reservations collection.
- Uses a HashSet-backed lookup through Java collectors.
- Preserves the existing synchronization lock.
- Maintains the original REST endpoint.
- Preserves the expected response structure.
- Does not change reservation creation behavior.

The synchronization lock was retained to protect the shared HashMap-based reservation state.

Git version history preserves the original and modified implementations.

---

## 11. Post-Optimization Performance Results

The lightweight scalability test was repeated after implementing the optimization.

| Concurrency | Before Optimization | After Optimization |
|---|---:|---:|
| 1 | 1266.38 req/sec | 313.21 req/sec |
| 5 | 5155.99 req/sec | 722.37 req/sec |
| 20 | 6631.70 req/sec | 850.72 req/sec |

All 300 post-optimization requests succeeded.

The initial post-optimization results showed lower throughput at all three concurrency levels.

The optimization therefore did not demonstrate a measured throughput improvement in this test.

Because the benchmark durations were very short, additional measurements were collected.

---

## 12. Repeated Post-Optimization Testing

Three additional scalability tests were executed.

Each test included 100 requests at concurrency levels of 1, 5, and 20.

| Concurrency | Repeat 1 | Repeat 2 | Repeat 3 | Average |
|---|---:|---:|---:|---:|
| 1 | 950.02 | 382.83 | 393.19 | 575.35 |
| 5 | 2405.31 | 1293.07 | 1291.15 | 1663.18 |
| 20 | 937.02 | 1513.78 | 1453.94 | 1301.58 |

All throughput values are requests per second.

Summary:

- Total Requests: 900
- Successful Requests: 900
- Failed Requests: 0
- Error Rate: 0%

The results varied considerably between repetitions.

The short benchmark duration, JVM warm-up, operating system scheduling, and local system conditions may have influenced the measurements.

Although the optimization reduced the theoretical complexity of appointment filtering, the tests did not establish an improvement in measured throughput.

The results also do not establish that the code modification alone caused the lower throughput.

---

## 13. Final Concurrent Correctness Validation

After implementing the optimization, the concurrent correctness validation script was executed again.

| Metric | Result |
|---|---:|
| Background Jobs | 20 |
| Requests per Job | 2 |
| Total Requests | 40 |
| Valid Responses | 40 |
| Invalid Responses | 0 |
| Validation Error Rate | 0% |

All 40 requests passed the configured response validation checks.

This verifies expected appointment retrieval behavior under the tested concurrent workload.

Because this test uses read-only GET requests, it does not independently establish correctness during simultaneous reservation creation.

---

## 14. Final Automated Testing

The final verification command was:

```powershell

.\mvnw.cmd clean test

```

Results:

```text

Tests run: 13, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS

```

The application was then started using:

```powershell

.\mvnw.cmd spring-boot:run

```

Spring Boot started successfully, and embedded Tomcat initialized on port 8080.

A final HTTP GET request to `/api/appointments` returned all four expected appointment slots.

These results confirm that the optimized implementation continued to pass the existing automated tests and function correctly for the verified endpoint.

---

## 15. Findings and Limitations

The testing produced several findings:

1. All measured sequential and concurrent workloads completed without observed HTTP request failures.
2. PowerShell background-job overhead influenced throughput measurements.
3. Lightweight asynchronous tests provided higher throughput measurements but used very short execution durations.
4. The original availability retrieval method contained a potential algorithmic scalability limitation.
5. The optimization reduced repeated reservation searches while preserving synchronization.
6. Post-optimization testing did not establish a measurable throughput improvement.
7. Repeated testing revealed substantial measurement variability.
8. All 13 automated tests passed after the optimization.
9. Final concurrent validation confirmed 40 valid responses with zero failures.

The tests were conducted locally using a small appointment dataset.

Longer-duration tests, larger reservation datasets, controlled JVM warm-up, and repeated measurements of both implementation versions would be required for a stronger performance assessment.

---

## 166. Conclusion

Assignment 7 evaluated the Concurrent Reservation Service through baseline testing, progressively increasing concurrent workloads, throughput analysis, resource monitoring, performance optimization, and correctness verification.

The service completed the measured HTTP workloads without observed request failures.

A potential algorithmic limitation was identified in the appointment availability retrieval logic and addressed through set-based filtering while preserving synchronization.

Although the modification improved the theoretical efficiency of availability filtering, the measured results did not establish a throughput improvement.

Repeated testing showed that short-duration localhost benchmarks can produce variable results.

Final automated and functional verification passed, demonstrating that the optimized application preserved the behavior covered by the existing tests and validation activities.
 