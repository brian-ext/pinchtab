# Memory Monitoring Guide

Pinchtab can monitor Chrome's JavaScript heap usage across all tabs in each browser instance. This helps track memory consumption and detect leaks during long-running automation tasks.

## Enabling Memory Metrics

Memory monitoring is **off by default** because it requires per-tab CDP calls which can be heavy with many tabs open.

To enable:

1. Open the dashboard at `http://localhost:<port>/`
2. Go to **Settings** → **📈 Monitoring**
3. Toggle **Memory Metrics** on
4. Click **Apply Settings**

## What Gets Measured

For each running instance, Pinchtab aggregates metrics across all open tabs:

| Metric | Description |
|--------|-------------|
| `jsHeapUsedMB` | JavaScript heap memory currently in use |
| `jsHeapTotalMB` | Total JavaScript heap allocated |
| `documents` | Number of Document objects |
| `frames` | Number of Frame objects |
| `nodes` | DOM node count |
| `listeners` | Number of JS event listeners |

## API Endpoints

### Per-Tab Metrics

```
GET /tabs/{tabId}/metrics
```

Returns memory metrics for a specific tab:

```json
{
  "jsHeapUsedMB": 12.5,
  "jsHeapTotalMB": 24.0,
  "documents": 1,
  "frames": 1,
  "nodes": 1250,
  "listeners": 45
}
```

### Instance Metrics (Aggregated)

```
GET /metrics
```

Returns aggregated metrics across all tabs in the instance:

```json
{
  "metrics": { ... },
  "memory": {
    "jsHeapUsedMB": 85.2,
    "jsHeapTotalMB": 128.0,
    "documents": 5,
    "frames": 8,
    "nodes": 12500,
    "listeners": 320
  }
}
```

### All Instances (Orchestrator)

```
GET /instances/metrics
```

Returns memory metrics for all running instances:

```json
[
  {
    "instanceId": "inst_abc123",
    "profileName": "MyProfile",
    "jsHeapUsedMB": 85.2,
    "jsHeapTotalMB": 128.0,
    "documents": 5,
    "frames": 8,
    "nodes": 12500,
    "listeners": 320
  }
]
```

## Dashboard Visualization

When enabled, the Monitoring page shows:

- **Chart**: Solid lines for tab counts, dashed lines for memory (MB)
- **Instance List**: Shows memory next to tab count (e.g., `:9868 · 3 tabs · 85MB`)
- **Dual Y-Axis**: Left axis for tabs, right axis for memory in MB

Data is polled every 30 seconds and the chart retains the last 60 data points (~30 minutes of history).

## Performance Considerations

- Each tab requires a CDP `Performance.getMetrics()` call
- With many tabs (10+), this adds noticeable overhead
- Consider disabling for production workloads where performance matters
- Enable temporarily for debugging memory issues

## Troubleshooting

**Metrics show 0 or very low values**

This can happen if:
- No pages are loaded (empty tabs)
- Tabs were opened but not tracked by Pinchtab (external opens)

**Memory keeps growing**

Possible memory leak. Check:
- Event listeners not being removed
- DOM nodes accumulating
- Closures holding references
