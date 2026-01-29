# Branch Differences - Failure Scenarios

This document describes the intentional failures injected into each branch for testing and demo purposes.

## Overview

Each branch contains different types of failures in the `checkoutservice` to demonstrate various error handling and observability scenarios.

## Branch Details

### 🔷 DEV Branch
**Purpose**: Clear stacktrace generation for debugging practice

**Failure Type**: Nil Pointer Dereference  
**Failure Rate**: 10% of checkout requests  
**Location**: `src/checkoutservice/main.go` - `PlaceOrder()` function

**What Happens**:
```go
// DEV: Simple nil pointer dereference to generate clear stacktraces
if rand.Float64() < 0.10 {
    log.Errorf("[PlaceOrder] SIMULATED FAILURE: Unexpected nil value in order processing for user_id=%q", req.UserId)
    var nilPointer *pb.PlaceOrderRequest
    _ = nilPointer.UserId // This will panic with nil pointer dereference
}
```

**Error Message**:
- Log: `SIMULATED FAILURE: Unexpected nil value in order processing`
- Panic: `runtime error: invalid memory address or nil pointer dereference`

**Use Cases**:
- ✅ Generates clean, easy-to-read stacktraces
- ✅ Practice debugging panic scenarios
- ✅ Test error monitoring and alerting
- ✅ Simple failure pattern for learning

**Image Tag**: `ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/checkoutservice:dev`

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

The `demo-sandbox-online-boutique/deploy/all-in-one.yaml` currently references the `:dev` tag:

```yaml
image: ghcr.io/runwhen-contrib/demo-sandbox-online-boutique-src/checkoutservice:dev
```

To use different branches:
- **DEV**: `:dev` (nil pointer failures, 10% rate)
- **PROD**: `:prod` (connection pool exhaustion, 5% rate)
- **TEST**: `:test` (no failures)
- **MAIN**: `:latest` (no failures)

### Switching Between Branches

```bash
# Deploy with dev failures (nil pointer)
sed -i 's/:prod/:dev/g' deploy/all-in-one.yaml
kubectl apply -f deploy/all-in-one.yaml

# Deploy with prod failures (connection pool)
sed -i 's/:dev/:prod/g' deploy/all-in-one.yaml
kubectl apply -f deploy/all-in-one.yaml

# Deploy without failures
sed -i 's/:prod/:test/g' deploy/all-in-one.yaml
kubectl apply -f deploy/all-in-one.yaml
```

## Observing Failures

Once deployed with the loadgenerator:

1. **Check logs**:
   ```bash
   kubectl logs -f deployment/checkoutservice | grep "SIMULATED FAILURE"
   ```

2. **Watch for panics**:
   ```bash
   kubectl logs -f deployment/checkoutservice | grep -A 20 "panic"
   ```

3. **Monitor error rate**:
   - DEV: Expect ~10% of checkout requests to fail
   - PROD: Expect ~5% of checkout requests to fail

4. **Stacktrace visibility**:
   - DEV: Clear Go stacktraces showing nil pointer dereference
   - PROD: Stacktraces with database connection context

## Testing Recommendations

1. **Start with TEST/MAIN** - Verify application works correctly
2. **Move to DEV** - Learn stacktrace analysis with simple errors
3. **Graduate to PROD** - Practice realistic production issue debugging

## Notes

- All failures are triggered during the `PlaceOrder` operation
- Loadgenerator will automatically trigger these failures during checkout
- Failures are logged before panicking for better observability
- Each panic generates a full Go stacktrace for debugging practice
