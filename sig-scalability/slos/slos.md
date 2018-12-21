# Kubernetes scalability and performance SLIs/SLOs

## What Kubernetes guarantees?

One of the important aspects of Kubernetes is its scalability and performance
characteristic. As Kubernetes user or operator/administrator of a cluster
you would expect to have some guarantees in those areas.

The goal of this doc is to organize the guarantees that Kubernetes provides
in these areas.

## What do we require from SLIs/SLOs?

We are in the process of extending the number of SLIs ([Service Level Indicators])
and SLOs ([Service Level Objectives]) built on top of these SLIs to cover more areas
of the system and user expectations.

Our SLIs/SLOs need to have the following properties:
- <b> They need to be testable </b> <br/>
  Ideally, they (SLIs and SLOs) should be measurable in all running clusters,
	but if   that isn't possible a benchmark may be enough in some situations.
  That means that not every SLO may be translatable to SLA ([Service
  Level Agreement]).
- <b> They need to be understandable for users </b> <br/>
  In particular, they need to be understandable for people not familiar
  with the system internals, i.e. their formulation can't depend on some
  arcane knowledge.

We may also introduce internal(for developers only) SLIs, that may be useful
for understanding performance characteristic of the system, but for which
we don't provide any guarantees for users (and thus don't require them to be
that easily understandable).

[Service Level Indicators]: https://en.wikipedia.org/wiki/Service_level_indicator
[Service Level Objectives]: https://en.wikipedia.org/wiki/Service_level_objective
[Service Level Agreement]: https://en.wikipedia.org/wiki/Service-level_agreement

## Types of SLOs

While SLIs are very generic and don't really depend on anything (they just
define what and how we measure), it's not the case for SLOs.
SLOs provide guarantees, and satisfying them may depend on meeting some
specific requirements.

As a result, we build our SLOs in "you promise, we promise" format.
That means, that we provide you a guarantee only if you satisfy the requirement
that we put on you.

As a consequence we introduce the two types of SLOs.

### Steady state SLOs

With steady state SLOs, we provide guarantees about system's behavior during
normal operations. We are able to provide much more guarantees in that situation.

```Definition
We define system to be in steady state when the cluster churn per second is <= 20, where

churn = #(Pod spec creations/updates/deletions) + #(user originated requests) in a given second
```

### Burst SLO

With burst SLOs, we provide guarantees on how system behaves under the heavy load
(when user wants the system to do something as quickly as possible not caring too
much about response time).

## Environment

In order to meet the SLOs, system must run in the environment satisfying
the following criteria:
- Runs a single or more appropriate sized master machines
- Main etcd running on master machine(s)
- Events are stored in a separate etcd running on the master machine(s)
- Kubernetes version is at least X.Y.Z
- ...

__TODO: Document other necessary configuration.__

## Thresholds

To make the cluster eligible for SLO, users also can't have too many objects in
their clusters. More concretely, the number of different objects in the cluster
MUST satisfy thresholds defined in [thresholds file][].

[thresholds file]: https://github.com/kubernetes/community/blob/master/sig-scalability/configs-and-limits/thresholds.md


## Kubernetes SLIs/SLOs

The currently existing SLIs/SLOs are enough to guarantee that cluster isn't
completely dead. However, they are not enough to satisfy user's needs in most
of the cases.

We are looking into extending the set of SLIs/SLOs to cover more parts of
Kubernetes.

```
Prerequisite: Kubernetes cluster is available and serving.
```

### Steady state SLIs/SLOs

| Status | SLI | SLO | User stories, test scenarios, ... |
| --- | --- | --- | --- |
| __Official__ | Latency of mutating API calls for single objects for every (resource, verb) pair, measured as 99th percentile over last 5 minutes | In default Kubernetes installation, for every (resource, verb) pair, excluding virtual and aggregated resources and Custom Resource Definitions, 99th percentile per cluster-day<sup>[1](#footnote1)</sup> <= 1s | [Details](./api_call_latency.md) |
| __Official__ | Latency of non-streaming read-only API calls for every (resource, scope pair, measured as 99th percentile over last 5 minutes | In default Kubernetes installation, for every (resource, scope) pair, excluding virtual and aggregated resources and Custom Resource Definitions, 99th percentile per cluster-day<sup>[1](#footnote1)</sup> (a) <= 1s if `scope=resource` (b) <= 5s if `scope=namespace` (c) <= 30s if `scope=cluster` | [Details](./api_call_latency.md) |
| __Official__ | Startup latency of stateless and schedulable pods, excluding time to pull images and run init containers, measured from pod creation timestamp to when all its containers are reported as started and observed via watch, measured as 99th percentile over last 5 minutes | In default Kubernetes installation, 99th percentile per cluster-day<sup>[1](#footnote1)</sup> <= 5s | [Details](./pod_startup_latency.md) |
| __WIP__ | Latency of programming a single (e.g. iptables on a given node) in-cluster load balancing mechanism, measured from when service spec or list of its `Ready` pods change to when it is reflected in load balancing mechanism, measured as 99th percentile over last 5 minutes | In default Kubernetes installation, 99th percentile of (99th percentiles across all programmers (e.g. iptables)) per cluster-day<sup>[1](#footnote1)</sup> <= X | [Details](./network_programming_latency.md) |
| __WIP__ | Latency of programming a single in-cluster dns instance, measured from when service spec or list of its `Ready` pods change to when it is reflected in that dns instance, measured as 99th percentile over last 5 minutes | In default Kubernetes installation, 99th percentile of (99th percentiles across all dns instances) per cluster-day <= X | [Details](./dns_programming_latency.md) |
| __WIP__ | In-cluster network latency from a single prober pod, measured as latency of per second ping from that pod to "null service", measured as 99th percentile over last 5 minutes. | In default Kubernetes installataion with RTT between nodes <= Y, 99th percentile of (99th percentile over all prober pods) per cluster-day <= X | [Details](./network_latency.md) |

<a name="footnote1">\[1\]</a> For the purpose of visualization it will be a
sliding window. However, for the purpose of reporting the SLO, it means one
point per day (whether SLO was satisfied on a given day or not).

### Burst SLIs/SLOs

| Status | SLI | SLO | User stories, test scenarios, ... |
| --- | --- | --- | --- |
| WIP | Time to start 30\*#nodes pods, measured from test scenario start until observing last Pod as ready | Benchmark: when all images present on all Nodes, 99th percentile <= X minutes | [Details](./system_throughput.md) |

### Other SLIs

| Status | SLI | User stories, ... |
| --- | --- | --- |
| WIP | Watch latency for every resource, (from the moment when object is stored in database to when it's ready to be sent to all watchers), measured as 99th percentile over last 5 minutes | [Details](./watch_latency.md) |
| WIP | Admission latency for each admission plugin type, measured as 99th percentile over last 5 minutes | [Details](./api_extensions_latency.md) |
| WIP | Webhook call latency for each webhook type, measured as 99th percentile over last 5 minutes | [Details](./api_extensions_latency.md) |
| WIP | Initializer latency for each initializer, measured as 99th percentile over last 5 minutes | [Details](./api_extensions_latency.md) |

