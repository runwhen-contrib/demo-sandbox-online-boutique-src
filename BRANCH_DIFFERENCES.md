# Branch Differences - Failure Scenarios

This document describes the intentional failures injected into each branch for testing and demo purposes.

## Overview

Each branch contains different types of failures to demonstrate various error handling and observability scenarios. The DEV branch injects a crash into `adservice`, while the PROD branch injects a different failure into `checkoutservice`.

## Branch Details

### 🔷 DEV Branch
**Purpose**: Clear stacktrace generation for debugging practice

**Failure Type**: NullPointerException  
**Failure Rate**: 10% of ad requests  
**Service**: `adservice` (Java)  
**Location**: `src/adservice/src/main/java/hipstershop/AdService.java` - `getAds()` method

**What Happens**:
```java
// DEV: Throws NullPointerException and terminates the JVM to simulate a fatal crash
if (Math.random() < 0.10) {
    logger.fatal("SIMULATED FAILURE: Unexpected null reference in ad catalog lookup for context_keys=" + req.getContextKeysList());
    NullPointerException npe = new NullPointerException("Ad catalog reference is null during ad retrieval");
    logger.fatal("Fatal error in getAds — terminating process", npe);
    System.exit(1);
}
```

**Error Message**:
- Log: `FATAL - SIMULATED FAILURE: Unexpected null reference in ad catalog lookup`
- Log: `Fatal error in getAds — terminating process` with `java.lang.NullPointerException` stacktrace
- JVM terminates with exit code 1

**Use Cases**:
- ✅ Generates clean, easy-to-read Java stacktraces
- ✅ Practice debugging crash scenarios in a different service than prod
- ✅ Test error monitoring and alerting
- ✅ Simple failure pattern for learning — distinct from the prod checkoutservice issue

**Image Tag**: `ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/adservice:dev`

---

### 🔴 PROD Branch
**Purpose**: Realistic production issue - connection exhaustion

**Failure Type**: Database Connection Pool Exhaustion  
**Failure Rate**: 5% of checkout requests  
**Location**: `src/checkoutservice/main.go` - `PlaceOrder()` function

**What Happens**:
```go
// PROD: Database connection pool exhaustion - realistic production issue
if rand.Float64() < 0.05 {
    log.Errorf("[PlaceOrder] SIMULATED FAILURE: Database connection pool exhausted for user_id=%q", req.UserId)
    panic(fmt.Sprintf("FATAL: Database connection pool exhausted - unable to process order for user %s. Connection timeout after 30s. Active connections: 100/100. Please check database connection settings and increase pool size.", req.UserId))
}
```

**Error Message**:
- Log: `SIMULATED FAILURE: Database connection pool exhausted`
- Panic: `FATAL: Database connection pool exhausted - unable to process order for user X. Connection timeout after 30s. Active connections: 100/100. Please check database connection settings and increase pool size.`

**Use Cases**:
- ✅ Simulates realistic production infrastructure issue
- ✅ Tests connection pool monitoring
- ✅ Practice incident response for resource exhaustion
- ✅ Demonstrates actionable error messages

**Image Tag**: `ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/checkoutservice:prod`

---

### 🟢 TEST Branch
**Purpose**: Baseline / Clean version

**Failure Type**: None  
**Failure Rate**: 0%  
**Location**: No failures injected

**Use Cases**:
- ✅ Baseline for comparison
- ✅ Verify application works without errors
- ✅ Performance testing without artificial failures
- ✅ Integration testing

**Image Tag**: `ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/checkoutservice:test`

---

### ⚪ MAIN Branch
**Purpose**: Production-ready baseline

**Failure Type**: None  
**Failure Rate**: 0%  
**Location**: No failures injected

**Use Cases**:
- ✅ Source of truth for production code
- ✅ Merge target for all branches
- ✅ Clean codebase reference

**Image Tag**: `ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/checkoutservice:latest`

---

## Deployment

### Using with Kubernetes

The `demo-sandbox-online-boutique/deploy/all-in-one.yaml` references branch-specific image tags per service. Each branch of the Flux repo pulls the matching tag:

- **DEV branch**: `adservice:dev` (NullPointerException, 10% rate) — all other services use `:dev` clean images
- **PROD branch**: `checkoutservice:prod` (connection pool exhaustion, 5% rate)
- **TEST branch**: all `:test` images (no failures)
- **MAIN branch**: all `:latest` images (no failures)

## Observing Failures

Once deployed with the loadgenerator:

1. **Check adservice logs (DEV)**:
   ```bash
   kubectl logs -f deployment/adservice -n online-boutique-dev | grep "SIMULATED FAILURE"
   ```

2. **Watch for exceptions (DEV)**:
   ```bash
   kubectl logs -f deployment/adservice -n online-boutique-dev | grep -A 20 "NullPointerException"
   ```

3. **Check checkoutservice logs (PROD)**:
   ```bash
   kubectl logs -f deployment/checkoutservice -n online-boutique-prod | grep "SIMULATED FAILURE"
   ```

4. **Monitor error rate**:
   - DEV (adservice): Expect ~10% of ad requests to fail
   - PROD (checkoutservice): Expect ~5% of checkout requests to fail

5. **Stacktrace visibility**:
   - DEV: Clear Java stacktraces showing NullPointerException in adservice
   - PROD: Go stacktraces with database connection context in checkoutservice

## Testing Recommendations

1. **Start with TEST/MAIN** - Verify application works correctly
2. **Move to DEV** - Learn stacktrace analysis with adservice crash (simple error, different service than prod)
3. **Graduate to PROD** - Practice realistic production issue debugging with checkoutservice

## Notes

- DEV failures are triggered during ad retrieval (every page load requests ads)
- PROD failures are triggered during the `PlaceOrder` checkout operation
- Loadgenerator drives traffic that triggers both failure types
- Failures are logged before crashing for better observability
- DEV uses a different service (adservice) than PROD (checkoutservice) to avoid confusion between scenarios
