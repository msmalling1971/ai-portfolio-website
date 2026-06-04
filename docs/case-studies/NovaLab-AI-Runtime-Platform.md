# NovaLab AI Runtime Platform

Date: June 2026

## Objective

Build and validate a local AI runtime platform that can serve an OpenAI-compatible model API, expose runtime metrics, and support observability and load testing from a dedicated monitoring container.

This project is hands-on AI infrastructure work: model serving, GPU-backed inference, platform observability, dashboard validation, and controlled benchmark testing.

## Executive Summary

The NovaLab AI Runtime Platform extends the local AI infrastructure lab beyond basic model access into runtime serving, monitoring, dashboarding, and load testing.

The work completed so far includes:

- A dedicated vLLM runtime container serving `Qwen/Qwen2.5-7B-Instruct`
- RTX 3090 GPU passthrough with NVIDIA 595.80 and CUDA 13.2
- An OpenAI-compatible API exposed on port `8000`
- vLLM metrics exposed at `/metrics`
- A dedicated observability container running Prometheus, Grafana, Node Exporter, and Locust
- A Grafana dashboard for runtime, host, throughput, latency, and queue visibility
- Locust benchmark testing at controlled user counts

The goal was not just to prove that a local model could run. The goal was to understand whether the runtime could be observed, measured, tested, and explained like real infrastructure.

## Environment Summary

### CT202: `ai-vllm-01`

- Debian 13
- RTX 3090 GPU passthrough
- NVIDIA 595.80
- CUDA 13.2
- Python virtual environment
- vLLM 0.22
- Model: `Qwen/Qwen2.5-7B-Instruct`
- OpenAI-compatible API
- Metrics endpoint: `http://192.168.50.104:8000/metrics`

Runtime API endpoint:

```text
http://192.168.50.104:8000/v1
```

## Observability Stack

### CT203: `ai-observability-01`

- Debian 13
- Prometheus
- Grafana
- Node Exporter
- Locust
- Grafana Dashboard: `NovaLab AI Runtime Dashboard v1`
- Locust Web UI: `http://192.168.50.152:8089`

## Architecture

The current runtime platform separates model serving from observability and load testing.

```text
Open WebUI
    |
    | OpenAI-compatible API
    v
CT202: ai-vllm-01
    - Debian 13
    - vLLM 0.22
    - Qwen/Qwen2.5-7B-Instruct
    - RTX 3090 passthrough
    - NVIDIA 595.80
    - CUDA 13.2
    - /v1 API
    - /metrics

CT203: ai-observability-01
    - Debian 13
    - Prometheus scrapes CT202 /metrics
    - Grafana visualizes Prometheus data
    - Node Exporter exposes CT203 host metrics
    - Locust runs API load tests
```

## Major Milestones

- vLLM successfully served `Qwen/Qwen2.5-7B-Instruct`.
- Open WebUI connected to vLLM using `http://192.168.50.104:8000/v1`.
- Prometheus installed and running on CT203.
- Grafana installed and connected to Prometheus.
- Prometheus scraping vLLM metrics from `http://192.168.50.104:8000/metrics`.
- Created the `NovaLab AI Runtime Dashboard v1` dashboard.
- Locust installed and running from CT203.
- Locust web UI available at `http://192.168.50.152:8089`.

## Understanding the Dashboard

The dashboard is designed to show three layers at once:

- Observability server health
- AI runtime behavior
- Saturation indicators

That matters because a benchmark is only useful if the platform is being measured from more than one angle. A model can return answers and still be close to a bottleneck. The dashboard is meant to show whether the monitoring box is healthy, whether vLLM is processing work cleanly, and whether requests are starting to queue.

### CT203 CPU Usage %

Explains CPU usage on the observability server. This helps confirm Prometheus, Grafana, Node Exporter, and Locust are not becoming the bottleneck during testing.

### CT203 Memory Usage %

Explains memory usage on the observability server. This shows whether the monitoring/load-testing box has enough RAM headroom.

### CT203 Disk Usage %

Explains disk usage on the observability server. Prometheus stores time-series data locally, so disk usage matters over time.

### Qwen 2.5 7B Generation Tokens/sec

Explains output token throughput. This is the main AI runtime throughput metric and shows how many response tokens the model generates per second.

PromQL:

```promql
rate(vllm:generation_tokens_total[1m])
```

### Qwen 2.5 7B Prompt Tokens/sec

Explains input token throughput. This shows how many prompt/input tokens are being processed per second before responses are generated.

PromQL:

```promql
rate(vllm:prompt_tokens_total[1m])
```

### vLLM num_requests_running

Explains active requests currently being processed by vLLM. This helps show current concurrency.

### vLLM num_requests_waiting

Explains queued requests waiting for GPU/runtime resources. This is one of the most important saturation indicators. Zero means requests are not backing up. Anything above zero means a queue is forming.

### Average Request Latency

Explains average request completion time. This is one of the main user-facing performance metrics.

### P95 Request Latency

Explains the response time experienced by 95% of requests. This is often more useful than average latency because averages can hide slow outliers.

### KV Cache Usage %

Explains how much of the model's working memory cache is being used. Think of it as the AI runtime's short-term memory workspace. Low usage means there is plenty of context/cache headroom. High usage means the runtime may be getting closer to memory pressure.

PromQL:

```promql
vllm:kv_cache_usage_perc * 100
```

Simple interpretation guide:

- 0-30%: plenty of headroom
- 30-70%: normal operating range
- 70-90%: watch closely
- 90%+: possible bottleneck territory

## What I Watch First

When I am trying to understand whether the runtime is approaching saturation, these are the first dashboard signals I look at:

1. Requests Waiting
2. Average Latency
3. P95 Latency
4. Generation Tokens/sec
5. KV Cache Usage %

Requests Waiting is the first saturation warning. If it stays at zero, requests are still being accepted and processed without backing up. If it rises above zero, the runtime is starting to queue work.

Average Latency and P95 Latency show whether users would feel the platform slowing down. Average latency gives the general shape of response time, while P95 helps catch slower requests that the average can hide.

Generation Tokens/sec shows whether output throughput is scaling as load increases. If users increase but throughput flattens, something is likely becoming a limit.

KV Cache Usage % shows how much of the runtime's short-term memory workspace is being consumed. For me, this is a practical way to watch whether context/cache pressure is becoming part of the performance story.

Together, these panels give a quick operating view: are requests queueing, are responses slowing down, is throughput still climbing, and is the runtime running out of working memory headroom?

## Important PromQL Queries

### Tokens/sec

```promql
rate(vllm:generation_tokens_total[1m])
```

### Prompt tokens/sec

```promql
rate(vllm:prompt_tokens_total[1m])
```

### Requests running

```promql
vllm:num_requests_running
```

### Requests waiting

```promql
vllm:num_requests_waiting
```

### Average request latency

```promql
vllm:e2e_request_latency_seconds_sum / vllm:e2e_request_latency_seconds_count
```

### P95 request latency

```promql
histogram_quantile(0.95, rate(vllm:e2e_request_latency_seconds_bucket[5m]))
```

### KV cache usage

```promql
vllm:kv_cache_usage_perc * 100
```

## Important PromQL Fix

The original `Generated Tokens/sec` dashboard panel was wrong because it used the raw counter:

```promql
vllm:generation_tokens_total{}
```

That query showed lifetime total generated tokens, not tokens per second.

The corrected query is:

```promql
rate(vllm:generation_tokens_total[1m])
```

The prompt tokens panel was fixed with the same pattern:

```promql
rate(vllm:prompt_tokens_total[1m])
```

Raw counters continuously increase. They are useful, but they do not directly show throughput. For per-second throughput, the dashboard needs `rate()` over a time window.

This was an important observability lesson: a metric can be valid and still answer the wrong operational question.

## Implementation Notes: Dashboard v1.1

Dashboard v1.1 added the missing host and GPU telemetry needed to separate the observability server from the AI runtime server. The goal was practical: make it clear what CT203 is doing as the monitoring/load-testing host, and what CT202 is doing as the vLLM/GPU runtime host.

### vLLM runtime startup command

CT202 serves `Qwen/Qwen2.5-7B-Instruct` through vLLM on port `8000`:

```bash
source ~/vllm-env/bin/activate
vllm serve Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 8000 \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.90 \
  --max-model-len 8192
```

### Prometheus scrape jobs

CT203 runs Prometheus and Grafana. Prometheus scrapes the vLLM runtime on CT202 and the CT202 Node Exporter endpoint:

```text
vLLM scrape target:
192.168.50.104:8000

CT202 node exporter scrape target:
192.168.50.104:9100
```

### Node Exporter textfile collector

The Node Exporter textfile collector is enabled on CT202 so custom GPU metrics can be written as Prometheus-formatted files:

```text
/etc/default/prometheus-node-exporter
ARGS="--collector.textfile.directory=/var/lib/node_exporter/textfile_collector"
```

### Custom GPU metrics exporter

The `gpu-metrics.sh` script uses `nvidia-smi` to collect GPU utilization, temperature, power draw, and VRAM usage. It writes the current values to `gpu.prom`, which Node Exporter exposes through the textfile collector.

```bash
#!/usr/bin/env bash
set -euo pipefail

OUT="/var/lib/node_exporter/textfile_collector/gpu.prom"
TMP="${OUT}.$$"

read -r GPU_UTIL GPU_TEMP GPU_POWER GPU_MEM_USED < <(
  nvidia-smi --query-gpu=utilization.gpu,temperature.gpu,power.draw,memory.used \
    --format=csv,noheader,nounits | head -n 1 | tr -d ','
)

cat > "$TMP" <<EOF
# HELP gpu_utilization_percent GPU utilization percentage from nvidia-smi.
# TYPE gpu_utilization_percent gauge
gpu_utilization_percent $GPU_UTIL
# HELP gpu_temperature_celsius GPU temperature in Celsius from nvidia-smi.
# TYPE gpu_temperature_celsius gauge
gpu_temperature_celsius $GPU_TEMP
# HELP gpu_power_watts GPU power draw in watts from nvidia-smi.
# TYPE gpu_power_watts gauge
gpu_power_watts $GPU_POWER
# HELP gpu_memory_used_mb GPU memory used in MB from nvidia-smi.
# TYPE gpu_memory_used_mb gauge
gpu_memory_used_mb $GPU_MEM_USED
EOF

mv "$TMP" "$OUT"
```

### systemd automation

The exporter is automated with `gpu-metrics.service` and `gpu-metrics.timer`. The timer runs every 15 seconds so Grafana can show near-real-time GPU state without running `nvidia-smi` from Prometheus directly.

`gpu-metrics.service`:

```ini
[Unit]
Description=Collect NVIDIA GPU metrics for Node Exporter

[Service]
Type=oneshot
ExecStart=/usr/local/bin/gpu-metrics.sh
```

`gpu-metrics.timer`:

```ini
[Unit]
Description=Run GPU metrics collection every 15 seconds

[Timer]
OnBootSec=15s
OnUnitActiveSec=15s
Unit=gpu-metrics.service

[Install]
WantedBy=timers.target
```

### Grafana Dashboard v1.1

Dashboard v1.1 now separates CT203 observability host metrics from CT202 AI runtime host metrics. The GPU section includes utilization, temperature, power draw, and VRAM usage.

With `Qwen/Qwen2.5-7B-Instruct` loaded and the runtime idle/ready, the RTX 3090 showed about `21,996 MB` of VRAM used. That baseline matters because it distinguishes model residency from benchmark-driven memory changes.

## Locust Test Methodology

Locust was installed in a Python virtual environment on CT203:

```text
~/locust-env
```

Verified version:

```text
locust 2.44.1
```

Benchmark target:

```text
http://192.168.50.104:8000
```

Endpoint tested:

```text
POST /v1/chat/completions
```

Model:

```text
Qwen/Qwen2.5-7B-Instruct
```

Tests were run with controlled user counts and spawn rates. Screenshots were captured from both Locust and Grafana for evidence.

The Locust test should exercise the same OpenAI-compatible API path used by application clients:

```text
POST /v1/chat/completions
model: Qwen/Qwen2.5-7B-Instruct
```

## Benchmark Results

| Test | Users | Spawn Rate | Requests | Failures | Generation Tokens/sec | Prompt Tokens/sec | Avg Latency | P95 Latency | Requests Running | Requests Waiting | Current RPS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Baseline | 1 | 1 | 594 | 0 | ~33.07 | ~6.58 | ~4.18 sec / 4178 ms | ~4.20 sec Locust / ~4.88 sec Grafana | 1 | 0 | Not captured |
| Concurrency Test | 5 | 1 | 997 | 0 | ~159.31 | ~29.6 | ~4.22 sec / 4222.52 ms | ~4.30 sec Locust / ~4.88 sec Grafana | 5 | 0 | Not captured |
| 25-user benchmark | 25 | 5 | 1244 | 0 | ~751.76 | ~141 | ~4.54 sec / 4535.21 ms | ~4.60 sec Locust / ~4.88 sec Grafana | 19 | 0 | ~3.3 |
| 50-user benchmark | 50 | 10 | 3203 | 0 | ~1439.93 | ~262 | ~4.76 sec Grafana / 4922.37 ms Locust | ~4.89 sec Grafana / 5.0 sec Locust | 30 | 0 | ~6.9 |

## Benchmark Interpretation

The RTX 3090 scaled cleanly from one concurrent user to five concurrent users. Generation throughput increased from roughly 33 tokens/sec to roughly 159 tokens/sec, while average latency stayed almost flat at around 4.2 seconds. Prompt token throughput also scaled from roughly 6.58 tokens/sec to roughly 29.6 tokens/sec.

The most important finding was that `vLLM num_requests_waiting` remained at zero during the 5-user test. That means requests were not backing up in the queue, and the runtime had not reached saturation.

The later 25-user and 50-user tests showed the platform continuing to scale under higher concurrency. Generation throughput increased to roughly 752 tokens/sec at 25 users and roughly 1,440 tokens/sec at 50 users. Prompt token throughput also increased to roughly 141 tokens/sec and roughly 262 tokens/sec.

Average latency increased, but stayed in a usable range: roughly 4.54 seconds at 25 users and roughly 4.76 to 4.92 seconds at 50 users depending on whether Grafana or Locust was the reference. P95 latency stayed around 4.88 to 5.0 seconds at the 50-user level.

The most important finding across all tests was that `vLLM num_requests_waiting` remained at zero. Even when the runtime was processing 30 active requests during the 50-user benchmark, vLLM did not report queued requests.

These are still lab benchmark results, not final production performance claims. Their value is that they prove the runtime path, dashboard path, GPU path, and load testing path are working together.

## GPU Observability Findings

- GPU: EVGA RTX 3090 FTW3 Ultra
- NVIDIA Driver: 595.80
- CUDA: 13.2
- GPU Utilization during 50-user run: 100%
- Power Draw: 249W / 250W
- Temperature: 56 degrees C
- VRAM Usage: 21998 MiB / 24576 MiB
- Performance State: P2
- No benchmark failures
- No vLLM queued requests
- Note: `nvidia-smi` showed "No running processes found," but GPU utilization, power draw, and VRAM allocation confirmed active inference workload.

During the sustained 50-user benchmark, the RTX 3090 reached 100% utilization and nearly full power draw while maintaining only 56 degrees C. Despite the GPU being fully utilized, vLLM request queue depth remained at zero and Locust reported zero failures.

This indicates the platform handled the workload cleanly and that the GPU was thermally stable under sustained inference load.

## Screenshot Placeholders

The following screenshots were captured with Snagit and should be added to the repo when ready:

```text
screenshots/benchmark-1user-locust.png
screenshots/benchmark-1user-grafana.png
screenshots/benchmark-5users-locust.png
screenshots/benchmark-5users-grafana.png
content/images/project-screenshots/ai-infrastructure-platform/benchmark-25users-grafana.png
content/images/project-screenshots/ai-infrastructure-platform/benchmark-25users-locust.png
content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-grafana.png
content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-locust.png
content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-nvidia-smi.png
```

Suggested usage:

- `screenshots/benchmark-1user-locust.png`: Baseline Locust test evidence.
- `screenshots/benchmark-1user-grafana.png`: Baseline Grafana dashboard evidence.
- `screenshots/benchmark-5users-locust.png`: 5-user Locust concurrency test evidence.
- `screenshots/benchmark-5users-grafana.png`: 5-user Grafana dashboard evidence.
- `content/images/project-screenshots/ai-infrastructure-platform/benchmark-25users-grafana.png`: 25-user Grafana benchmark evidence.
- `content/images/project-screenshots/ai-infrastructure-platform/benchmark-25users-locust.png`: 25-user Locust benchmark evidence.
- `content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-grafana.png`: 50-user Grafana benchmark evidence.
- `content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-locust.png`: 50-user Locust benchmark evidence.
- `content/images/project-screenshots/ai-infrastructure-platform/benchmark-50users-nvidia-smi.png`: 50-user GPU validation evidence from `nvidia-smi`.

## Matt's Notes

Going into the 25-user test, I expected to start seeing request queueing. That did not happen. The 50-user test surprised me even more. Throughput climbed to roughly 1,440 generation tokens/sec, latency stayed around five seconds, and vLLM still showed zero waiting requests.

The biggest surprise was the GPU behavior. The RTX 3090 hit 100% utilization and 249W power draw, but temperature stayed around 56 degrees C. That gave me confidence that the card is healthy and that the platform is doing real work without thermal issues.

At this point, the project shifted from "can I run a local model?" to "how do I monitor and scale a real AI runtime platform?"

## Business and Platform Value

This work demonstrates practical AI infrastructure capability across several operating layers:

- Runtime serving with vLLM
- GPU-backed local model inference
- OpenAI-compatible API integration
- Prometheus-based metric collection
- Grafana dashboarding
- Runtime metric validation
- Load testing with Locust
- Early performance baseline creation

The value is not only that a model runs. The value is that the runtime can be observed, tested, measured, and improved.

## Current Status

Completed so far:

- CT202 deployed for vLLM runtime serving
- CT203 deployed for observability and load testing
- Qwen 2.5 7B served through vLLM
- Open WebUI connected to the vLLM API endpoint
- Prometheus scraping vLLM metrics
- Grafana dashboard created
- Locust installed and validated
- Baseline, 5-user, 25-user, and 50-user benchmark results captured
- Token/sec, prompt tokens/sec, latency, running requests, waiting requests, and current RPS validated
- GPU utilization, power draw, temperature, VRAM usage, and performance state validated during the 50-user benchmark

## Next Steps

1. Run 10-user benchmark.
2. Add RTX 3090 GPU metrics to the Grafana dashboard:
   - GPU utilization
   - VRAM usage
   - GPU temperature
   - GPU power draw
3. Add KV cache usage panel if available:

```promql
vllm:kv_cache_usage_perc * 100
```

4. Research Qwen 14B AWQ/GPTQ options.
5. Publish the case study to GitHub Pages.

## Interview Story Potential

This project can be framed as a platform engineering story:

- Built a local AI runtime service using vLLM and an RTX 3090.
- Integrated the runtime with Open WebUI through an OpenAI-compatible API.
- Added observability with Prometheus and Grafana.
- Identified and corrected misleading dashboard queries.
- Validated the serving path with Locust load testing.
- Established an early performance baseline for future tuning.

The core lesson is that AI infrastructure needs the same operational discipline as traditional infrastructure: clear endpoints, reliable metrics, validated dashboards, load testing, and documentation.
