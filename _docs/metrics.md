---
title: Metrics
tags:
  - metrics
  - statistics
  - configuration
description: Rönd Prometheus metrics
order: 4
---

# Metrics

{%
  include alert.html
  type="info"
  content="This feature is available from version v1.6.2"
%}

By default, Rönd exposes the Prometheus metrics in the OpenMetrics format. The path at which this metrics are exposed is `/-/rond/metrics`.

Exposed metrics are:

| Name | Type | Description |
|------|------|-------------|
| `rond_policy_evaluation_duration_milliseconds` | Histogram | The policy evaluation durations in milliseconds |

## Grafana dashboards

We have created a Grafana dashboard which uses both Prometheus and Loki as data sources. <a download target="_blank" href="/assets/static/rond-dashboard.json">Click here</a> to download the json of the dashboard.

#### Disable metrics

To disable the collection of metrics, set the environment variables `EXPOSE_METRICS` to `false`. This will also prevent the exposition of the metrics collection route.
