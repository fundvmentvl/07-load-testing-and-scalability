# Concurrent Reservation Service

## CEN4025 — Assignment 7: Load Testing and Scalability

### Project Overview

The Concurrent Reservation Service is a Java Spring Boot application originally developed for the CEN4025 midterm project and reused for Assignment 7 to evaluate application performance under increasing HTTP request workloads.

The application provides appointment availability and reservation management through an HTTP REST API. It also includes a TCP server for reservation operations.

Assignment 7 focuses on measuring response times, evaluating concurrent request handling, identifying scalability limitations, and documenting performance improvements.

### Technologies Used

- Java 26

- Spring Boot

- Apache Maven

- JUnit 5

- Windows PowerShell 5.1

- IntelliJ IDEA

- Git and GitHub

### Application Architecture

The application includes:

- **ReservationController:** Exposes HTTP REST endpoints.

- **ReservationService:** Implements reservation business logic and manages shared reservation state.

- **ReservationStore:** Provides an abstraction for reservation persistence.

- **JsonReservationStore / CsvReservationStore:** Support alternative persistence formats.

- **TCP Server:** Handles reservation requests through a TCP interface.

- **AsyncAuditService:** Supports asynchronous audit logging.

- **JUnit Tests:** Verify reservation functionality and concurrency behavior.

The HTTP API and TCP server use the same reservation service.

### HTTP API Endpoints

| Method | Endpoint | Description |

|---|---|---|

| GET | `/api/appointments` | Retrieve available appointment slots |

| POST | `/api/reservations` | Create a reservation |

| GET | `/api/reservations/{id}` | Retrieve a reservation by ID |

| DELETE | `/api/reservations/{id}` | Cancel a reservation |

The load tests currently target the read-only endpoint:

`http://localhost:8080/api/appointments`

Using a read-only endpoint allows repeated requests without consuming appointment slots or modifying reservation data.

### Running the Application

#### Prerequisites

- Compatible Java Development Kit

- Maven Wrapper

- Windows PowerShell

- IntelliJ IDEA or another Java IDE

#### Build and Test

From the project root:

```powershell

.\mvnw.cmd clean test

```

#### Start the Application

Run the Spring Boot application class in IntelliJ IDEA.

The application starts:

- HTTP server on port `8080`

- TCP server on port `9090`

#### Verify the HTTP Endpoint

```powershell

Invoke-RestMethod -Uri "http://localhost:8080/api/appointments" -Method Get

```

A successful response returns the available appointment slots.

---

## Performance and Load Testing

### Testing Objective

The objective of Assignment 7 is to evaluate how the Concurrent Reservation Service responds to increasing HTTP request demand.

The tests measure:

- Total HTTP requests

- Successful requests

- Failed requests

- Average response time

- Minimum response time

- Maximum response time

- Error rate

Additional workload levels will be used to investigate scalability and identify performance limitations.

### Testing Environment

| Component | Configuration |

|---|---|

| Operating system | Windows, 64-bit |

| Java | OpenJDK 26.0.2 |

| Maven | Apache Maven 3.9.16 |

| Development environment | IntelliJ IDEA |

| Load generator | Windows PowerShell 5.1 |

| Application host | Local machine |

| HTTP port | 8080 |

| Tested endpoint | `GET /api/appointments` |

| Network | Localhost |

The load generator and Spring Boot application run on the same computer.

This eliminates external network latency but means both processes share local CPU and memory resources.

### Test Script Directory

The project contains a `scripts` directory with PowerShell files used to reproduce the performance tests.

```text

07-load-testing-and-scalability/

├── scripts/

│   ├── baseline.ps1

│   └── low-load.ps1

├── src/

│   ├── main/

│   └── test/

├── pom.xml

├── mvnw

├── mvnw.cmd

└── README.md

```

### Test 1 — Sequential Baseline

**Script:** `scripts/baseline.ps1`

**Purpose:** Establish application response times under a low-demand workload.

**Workload:**

- 20 HTTP GET requests

- Requests executed sequentially

- No intentional concurrent clients

Run:

```powershell

.\scripts\baseline.ps1

```

#### Recorded Baseline Results

| Metric | Result |

|---|---:|

| Total requests | 20 |

| Successful requests | 20 |

| Failed requests | 0 |

| Average response time | 99.484 ms |

| Minimum response time | 64.37 ms |

| Maximum response time | 188.28 ms |

| Error rate | 0% |

All 20 baseline requests completed successfully.

These results establish the initial low-demand reference point for subsequent load tests.

### Test 2 — Concurrent Request Load

**Script:** `scripts/low-load.ps1`

**Purpose:** Evaluate how the application handles overlapping HTTP requests.

**Workload:**

- 20 HTTP GET requests

- 20 PowerShell background jobs

- Requests allowed to overlap

- Windows PowerShell 5.1-compatible execution

Run:

```powershell

.\scripts\low-load.ps1

```

#### Recorded Concurrent-Load Results

| Metric | Result |

|---|---:|

| Background jobs | 20 |

| Total requests | 20 |

| Successful requests | 20 |

| Failed requests | 0 |

| Average response time | 1,358.756 ms |

| Minimum response time | 601.26 ms |

| Maximum response time | 2,004.27 ms |

| Error rate | 0% |

All 20 concurrent-load requests completed successfully.

Average measured response time increased compared with the sequential baseline. However, the PowerShell background-job approach introduces additional client initialization and scheduling overhead.

Therefore, the difference should not be interpreted as server-side processing time alone.

### Running the Scripts

1. Start the Spring Boot application.

2. Confirm that port `8080` is available.

3. Open Windows PowerShell in the project root.

4. Execute the baseline test.

5. Execute the concurrent-load test.

6. Review the printed results and performance summaries.

Commands:

```powershell

.\scripts\baseline.ps1

.\scripts\low-load.ps1

```

If script execution is restricted, the following command can temporarily permit scripts in the current PowerShell session:

```powershell

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

```

### Measurement Method

Each script uses the .NET `Stopwatch` class to measure elapsed time around an HTTP request.

The scripts record individual request durations and calculate summary statistics.

The baseline script executes requests sequentially.

The concurrent-load script uses PowerShell background jobs to create overlapping requests.

These methods are useful for basic application load testing, but background-job startup overhead and local machine resource contention can affect measured response times.

The current scripts do not independently report aggregate throughput or server CPU and memory utilization. These metrics require additional measurement during the remaining scalability analysis.

### Current Performance Comparison

| Metric | Sequential Baseline | Concurrent Load |

|---|---:|---:|

| Total requests | 20 | 20 |

| Average response time | 99.484 ms | 1,358.756 ms |

| Minimum response time | 64.37 ms | 601.26 ms |

| Maximum response time | 188.28 ms | 2,004.27 ms |

| Failed requests | 0 | 0 |

| Error rate | 0% | 0% |

Both workloads completed without observed HTTP request failures.

The concurrent test showed higher client-observed response times, but additional controlled workload levels are needed before drawing conclusions about application scalability.

---

## Automated Testing

The project includes JUnit tests covering reservation service behavior.

The existing tests verify scenarios such as:

- Available appointment retrieval

- Successful reservation creation

- Invalid appointment handling

- Duplicate booking prevention

- Reservation cancellation

- Invalid customer input

- Concurrent booking protection

- Reservation persistence behavior

Run the tests using:

```powershell

.\mvnw.cmd clean test

```

Automated testing helps verify that performance-related changes do not introduce functional regressions.


## Version Control

Git and GitHub are used to track application changes, performance scripts, and documentation.

Recommended commit for the initial testing scripts:

```powershell

git add scripts README.md

git commit -m "Add reproducible baseline and concurrent load test scripts"

git push origin main

```

