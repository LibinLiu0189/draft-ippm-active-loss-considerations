---
stand_alone: true
ipr: trust200902
cat: info
submissiontype: IETF
area: Operations and Management
wg: IPPM

docname: draft-liu-ippm-measurement-interpretation-00

title: Considerations for Interpreting IP Performance Measurement Results
abbrev: Measurement Interpretation
lang: en

author:
- ins:
  name: Libin Liu
  org: Zhongguancun Laboratory
  city: Beijing
  country: China
  email: liulb@zgclab.edu.cn

informative:
  RFC2678:
  RFC2681:
  RFC2827:
  RFC3393:
  RFC3704:
  RFC4656:
  RFC4737:
  RFC5136:
  RFC5357:
  RFC5560:
  RFC6703:
  RFC7679:
  RFC7680:
  RFC7799:
  RFC8704:
  RFC8762:
  RFC8911:
  RFC9097:
  RFC9232:
  RFC9341:

--- abstract
IP Performance Measurement (IPPM) metrics characterize properties such as connectivity, packet loss, delay, delay variation, packet reordering, packet duplication, and capacity. A result can be correct for the metric and measurement method used while still being insufficient to explain the network condition that produced it or to describe traffic outside the measured scope.

This document provides guidance for interpreting IPPM measurement results. It considers the metric definition and parameters, the measured packet or traffic population, the observation scope, and the measurement method. It also discusses metric-specific considerations and differences among active, passive, and hybrid measurements. Operational information can support an interpretation, but this document keeps it separate from the metric result itself.

This document does not define new metrics or measurement protocols, and it does not specify a mechanism for determining a unique operational cause.
--- middle

# Introduction {#intro}

The IP Performance Measurement (IPPM) framework defines metrics and measurement methods for characterizing packet delivery and network performance. Examples include connectivity {{RFC2678}}, one-way and round-trip delay {{RFC7679}} {{RFC2681}}, packet loss {{RFC7680}}, delay variation {{RFC3393}}, packet reordering {{RFC4737}}, packet duplication {{RFC5560}}, and capacity-related metrics {{RFC5136}} {{RFC9097}}. The Performance Metrics Registry in {{RFC8911}} records metric definitions, parameters, methods, and other information needed to perform and report measurements. This document focuses on packet-transfer and network-performance metrics in the IPPM context.

The metric, its parameters, and the measurement method define the scope of a metric result. That result does not identify the condition that produced it. For example, a delay value does not separate propagation, transmission, queueing, processing, and timestamping effects. A loss result does not identify where the network discarded packets or why it did so. Reordering identifies an ordering property of the measured stream, not the mechanism that caused it. Capacity results depend on the metric definition, offered load, packet characteristics, qualification criteria, and network state.

The distinction matters when operators generalize a measurement result or use it to support an operational explanation. The metric result states the measured fact. Statements about congestion, routing or forwarding changes, filtering, endpoint behavior, or other causes require additional evidence. A valid result can also be unrepresentative of traffic outside the measured population, path, direction, or time interval.

Metric parameters shape the interpretation. {{RFC8911}} requires the parameters needed to perform and interpret a measurement to be known, and notes that runtime parameters can affect the measured property. Results in the same broad metric family may not support comparison when their parameters or measurement conditions differ.

The measurement method is a separate consideration. {{RFC7799}} classifies measurement methods as Active, Passive, or Hybrid. Active methods use dedicated measurement traffic. Passive methods observe existing traffic. Hybrid methods combine active and passive characteristics. These methods differ in traffic control, observation scope, representativeness, and measurement-system effects, so the same metric may require different interpretation depending on the method that produced it.

This document uses three parts of the measurement context when interpreting a result:

* the metric definition and parameters, which define the measured property;
* the measurement method and observation scope, which define how the measurement forms the measured population; and
* supporting evidence, when an operator uses the result to justify a broader or causal conclusion.

The general and metric-specific considerations address the first part. The sections on active, passive, and hybrid measurement address the second. The discussion of interpretation context and supporting evidence addresses the third. These parts are complementary. The document does not assume that every IPPM metric applies to every measurement-method category.

This document limits its scope to interpretation of measurement results. The document does not redefine existing metrics, change their semantics, specify a general troubleshooting procedure, or define a mechanism for determining a unique operational cause. {{RFC6703}} discusses how the intended use of IP performance measurements affects measurement design and reporting; this document complements that work by focusing on what conclusions a reported result can support.

{{terms}} defines terminology. {{problem}} describes the general interpretation problem. {{conditions}} summarizes operational conditions that can affect measurement results. {{general-considerations}} gives common interpretation considerations, and {{metric-considerations}} discusses representative metric families. {{active}}, {{passive}}, and {{hybrid}} cover method-specific considerations. {{context}} describes supporting context, and {{ops}} discusses how to apply the framework when comparing, correlating, or generalizing results. {{sec}} and {{privacy}} discuss security and privacy considerations.

# Terminology {#terms}

This document uses the terms Active Performance Measurement, Passive Performance Measurement, and Hybrid Performance Measurement consistently with {{RFC7799}}. It also uses Performance Metric and the distinction between fixed and runtime parameters consistently with {{RFC8911}}. Where terms are already defined by existing IPPM documents or measurement protocols, this document uses those terms consistently with their existing definitions.

Performance Metric:
: A quantitative measure of performance targeted to an IETF-specified protocol or to an application transported over an IETF-specified protocol, consistent with {{RFC8911}}. In this document, the discussion focuses on IPPM metrics that characterize packet-transfer or network-performance properties.

Method of Measurement:
: The procedure or set of operations used to determine a measured value or measurement result. This document uses the Active, Passive, and Hybrid classifications described in {{RFC7799}}.

Metric Result:
: A value, classification, sample, statistic, or other output produced by applying a defined performance metric and method of measurement with specified parameters. Examples include a connectivity result, a packet loss ratio, a one-way delay sample, a delay variation statistic, a reordered-packet ratio, a duplication result, or a capacity value.

Metric Parameters:
: Fixed and runtime parameters that define or configure a metric and its measurement method. Parameter values can affect the measured packet or traffic population, the network property under assessment, and the interpretation of the resulting value {{RFC8911}}.

Interpretation Context:
: Information needed to understand what conclusions a metric result can support. Interpretation context can include the metric definition, metric parameters, packet or traffic population, measurement interval, observation scope, measurement method, path information, and relevant network or measurement-system state.

Operational Context:
: Control-plane, management-plane, data-plane, Operations, Administration, and Maintenance (OAM), telemetry, endpoint, observation-system, controller, or operator-provided information that can help explain a metric result. Examples include routing state, forwarding state, interface state, queue counters, policy state, endpoint state, observation-point state, collector state, logs, and event records.

Observation Point:
: A location where a measurement function observes, counts, timestamps, samples, collects, or otherwise processes packets or measurement information. An endpoint, network device, interface, or another measurement function can host an observation point.

Measurement Flow:
: A set of packets that share header fields or other properties relevant to forwarding, load balancing, filtering, policing, observation, or measurement processing. Different measurement flows can experience different paths or treatment.

Measurement Representativeness:
: The extent to which the packets, traffic, paths, time periods, and observation points in a measurement represent the traffic or service behavior to which an analyst applies the result.

Collection Loss:
: Loss of measurement records, samples, exported data, or observation information within the measurement or collection system. Collection loss is distinct from packet loss in the measured network, although both can create gaps in available measurement data.

Active Performance Measurement:
: A measurement method that generates dedicated measurement traffic for measurement purposes {{RFC7799}}.

Passive Performance Measurement:
: A measurement method based on observation of an existing packet stream without modifying that stream for the purpose of the measurement {{RFC7799}}.

Hybrid Performance Measurement:
: A measurement method that combines characteristics of active and passive measurement {{RFC7799}}.

# General Problem Statement {#problem}

A performance metric can define the measured property without explaining why the result occurred. The interpretation problem is the gap between the result established by the metric and any broader or causal conclusion drawn from it.

## Metric Result and Operational Explanation {#result-vs-explanation}

A metric result establishes the quantity defined by the metric for the measured population and conditions. It does not, by itself, establish the cause. A loss result does not distinguish congestion from filtering or forwarding failure. A delay value does not identify the contribution of propagation, queueing, processing, or a path change. A reordering result does not identify the mechanism that changed packet order. Operational explanation remains separate from the metric result.

## Metric Definition and Parameters {#metric-definition-parameters}

Metric definitions and parameters determine the scope of a result. Fixed and runtime parameters can specify packet type, packet size, timing, thresholds, traffic class, observation interval, or other measurement conditions {{RFC8911}}.

Compare results in the same broad metric family only when the relevant definitions and parameters are compatible. A network change, a measurement-configuration change, or both can cause a difference.

## Similar Results from Different Operational Conditions {#similar-results}

Different conditions can produce similar results. Delay can increase because of queueing, path changes, endpoint processing, or timestamping effects. Loss can result from congestion, forwarding problems, filtering, policing, or endpoint behavior. Capacity can change because of competing traffic, path changes, shaping, physical-layer conditions, or the measurement setup.
A result consistent with one explanation does not exclude the others.

## Visibility and Observation Scope {#visibility}

Endpoints or observation points, traffic selection, the time interval, and the measurement method define measurement scope. A result for one path, traffic class, direction, or interval does not describe another.
Any generalization should remain within the visibility of the measurement unless additional evidence supports a broader claim.

## Flow-, Path-, Direction-, and Time-dependent Results {#flow-path-time}

Performance can depend on packet fields, traffic class, direction, path selection, and time. ECMP, link aggregation, tunneling, traffic engineering, and policy can cause flows between the same endpoints to experience different conditions. Routing convergence, queue bursts, policy changes, endpoint load, and measurement-system changes can produce short-lived differences.
Reports should identify the flow, path, direction, and interval represented by a result when those factors affect interpretation.

## Sample Population and Representativeness {#sample-population}

Many results derive from only part of the relevant packet or observation population. Delay statistics exclude packets that the measurement system does not observe and timestamp. Passive systems may sample traffic. Capacity methods can exclude intervals that fail qualification criteria. Reordering and duplication measurements depend on packet identification and observation completeness.

A result can be correct for the observed sample without representing packets or intervals that the network lost, the measurement system did not sample, the method excluded, or routing sent elsewhere.

## Limits of Cause Attribution {#cause-attribution-limits}

Additional context can narrow the set of plausible explanations, but it may not identify a unique cause. Counters can aggregate unrelated traffic, logs can be incomplete, clocks can differ, and several events can overlap.
Cause attribution should remain an inference unless the available evidence supports a definitive conclusion.

# Operational Conditions Affecting Measurement Results {#conditions}

Many operational conditions can affect one or more performance metrics. Their effects depend on the metric: congestion can increase delay and delay variation before causing loss; routing changes can alter delay, reordering, connectivity, or capacity; endpoint and measurement-system behavior can affect results even when the network does not change.

The list below is not a complete taxonomy of network faults. Its purpose is to illustrate why a metric result requires interpretation in context.

## Congestion and Queue Behavior {#congestion}

Congestion and queueing can affect delay, delay variation, packet loss, and capacity-related results. Queue occupancy can increase delay and delay variation before a queue begins discarding packets. Long aggregation intervals can hide short-lived changes caused by microbursts.

The effect can depend on traffic class, Differentiated Services Code Point (DSCP), packet size, flow identifiers, queue selection, and path selection. A result observed for one flow or traffic class may therefore not represent another.

Increased delay, queue occupancy, and packet loss can support a congestion hypothesis, but no single metric value proves congestion without supporting context.

## Link, Forwarding, and Routing Conditions {#routing-forwarding}

Link failures, physical errors, forwarding-plane faults, Forwarding Information Base (FIB) inconsistencies, routing loops, blackholes, tunnel failures, and routing convergence can affect connectivity, loss, delay, delay variation, reordering, and capacity-related results.

A path change can increase or decrease delay without loss. Convergence can create transient loss or reordering. A forwarding blackhole can affect connectivity and loss. Interpretation can benefit from routing events, interface state, forwarding state, and path information.

## Endpoint and Measurement-System Behavior {#endpoint-system}

Endpoints and the measurement system itself can affect measurement outcomes. Active receivers or reflectors can reach processing limits, apply rate limits, use incorrect configuration, or fail to timestamp or process packets as expected. Passive observation systems can miss packets or records. Hybrid measurement functions can apply marking or measurement processing inconsistently.

Such behavior can alter loss, delay, reordering, duplication, or capacity results without a corresponding change in the network path. Measurement-system state is therefore part of the interpretation context.

## Policy, Filtering, Policing, and Traffic Treatment {#policy}

Access control lists, firewalls, service filters, traffic conditioning, administrative policy, or policing can discard, delay, rate limit, or otherwise treat packets differently. Such treatment can affect connectivity, loss, delay, delay variation, and capacity-related results.

Effects can depend on source and destination addresses, protocol, transport ports, DSCP, packet size, direction, interface, customer, or traffic class. Before generalizing a metric result or attributing it to a specific condition, use policy configuration, per-class counters, or other supporting evidence when needed.

## Validation-related Discard {#validation}

Validation functions can discard packets that are not accepted in the applicable validation context. Source Address Validation is one example {{RFC2827}} {{RFC3704}} {{RFC8704}}.

Validation-related discard can affect connectivity and packet loss results and, through selective removal of packets or paths, can affect the sample population underlying other metrics. However, the presence of a validation function does not imply that validation caused a measurement result. Such an attribution requires supporting evidence such as validation counters, rule state, routing state, and event timing.

## Multipath and Load-balancing Effects {#multipath}

ECMP, link aggregation, and other multipath mechanisms can cause different packets or flows to traverse different component paths. Those paths can differ in propagation delay, queueing, loss, available capacity, policy, or observation coverage.

Multipath can affect delay distributions, delay variation, loss, reordering, and capacity results. Aggregate statistics can hide concentration of a result on one component path. Relevant context includes the fields and forwarding behavior that influence path selection.

## Transient Conditions and State Changes {#transient}

Routing convergence, link failure and restoration, FIB updates, tunnel reoptimization, policy changes, endpoint restarts, queue bursts, and measurement configuration changes can produce transient changes across several metrics.

A short event can appear as a burst of loss, a delay spike, reordering, a temporary connectivity failure, or a temporary reduction in measured capacity. Fine-grained timing and event records can help distinguish transient behavior from persistent conditions, although temporal correlation alone does not establish causality.

# General Considerations for Result Interpretation {#general-considerations}

The following considerations apply across metric families and measurement methods.

## Start with the Metric Definition and Parameters {#consider-definition}

Use the exact metric definition and the fixed and runtime parameters of the measurement. They define the measured property, the packet or traffic population, and the conditions under which the measurement produced the result {{RFC8911}}.

Do not compare or generalize results because they share a label such as loss, delay, or capacity. Where a Performance Metrics Registry entry applies, the registry entry and parameter values make the measurement semantics explicit.

## Identify the Method of Measurement {#consider-method}

Record how the measurement produced the result, including whether the method is Active, Passive, or Hybrid when the {{RFC7799}} classification applies. The method determines how the system forms the measured population, where it observes packets, which variables it controls, and which measurement-system effects can appear in the result.

Results for the same metric can have different scopes when the methods, observation points, selection rules, sampling procedures, or collection systems differ. {{active}}, {{passive}}, and {{hybrid}} discuss these cases in more detail.

## Separate Metric Results from Inferred Causes {#separate}

Keep the measured result separate from the operational explanation. A delay value is not a measurement of queueing unless the metric specifically defines it as such. A loss ratio is not a measurement of congestion. A reordering result is not a measurement of ECMP behavior.

If analysis infers a cause, report the evidence that supports the inference and any material uncertainty.

## Consider Multiple Metrics Together {#joint-metrics}

Related metrics can constrain an interpretation. Consider delay, delay variation, and loss together when examining queueing or path behavior. Reordering together with delay changes can provide evidence of a path change. Some capacity methods use loss and delay during qualification and reporting {{RFC9097}}. Cross-metric correlation is supporting evidence, not proof that one metric caused another.

## Consider Direction, Observation Scope, and Measurement Role {#scope-role}

Record the direction, endpoints, observation points, and measurement roles represented by the result. A one-way result does not describe the reverse path. A result at one observation point does not describe behavior outside that point's visibility. A reflected measurement includes the forward path, remote processing, and the return path. Operational conclusions should remain within that visibility.

## Consider Flow and Path Selection {#flow-selection}

Packet fields and encapsulation can affect forwarding, load balancing, filtering, policing, and queueing. Relevant fields can include source and destination addresses, transport ports, flow labels, DSCP, packet size, and tunnel information.

A change in result after these fields change can indicate flow- or path-dependent behavior rather than random measurement variation.

## Consider Timing and State Changes {#timing}

Interpret results in the interval in which the measurement produced them. Routing updates, link events, queue bursts, policy changes, endpoint events, and measurement-system changes can produce transient effects.

Correlation with such events requires compatible timestamps and sufficient time resolution. Long aggregation intervals can hide short events.

## Consider the Observed Sample Population {#observed-population}

Identify which packets, observations, or intervals contributed to the result. Packets that the metric classifies as lost do not contribute one-way delay samples. Sampled passive traffic does not directly describe unsampled traffic. Reordering results depend on the packets that arrive, while loss or duplication can change that observed population. Capacity methods can exclude intervals that fail qualification criteria.

Interpret the result for the population defined by the metric and method before extending it to a broader population.

## Consider Measurement Representativeness {#representativeness}

The measured population may differ from the traffic or service to which the result is later applied. Active probes can differ from production traffic. Passive measurements depend on traffic availability, observation-point placement, and sampling. Hybrid methods may mark or select only part of production traffic. When representativeness matters to the conclusion, report the relevant differences rather than assuming equivalence.

## Treat Correlation as Supporting Evidence, Not Proof {#correlation}

Network and measurement-system events often correlate with metric results. Correlation can make an explanation more plausible, but common timing does not establish causality and several events can overlap. Reports should distinguish the metric result, correlated evidence, and any inferred explanation.

## Report Uncertainty {#uncertainty}

If the available evidence does not support a unique cause or broader generalization, retain that uncertainty. At minimum, the report can state the result, relevant parameters, measured population, observation scope, interval, and the evidence used for interpretation.

# Metric-Specific Interpretation Considerations {#metric-considerations}

The common principles above apply across metrics, but different metric families have distinct interpretation concerns. This section discusses representative IPPM metric families. It is not intended to enumerate every metric defined or maintained by IPPM. In each case, interpretation begins with the specific metric definition rather than with a generic label such as "loss", "delay", or "capacity".

## Connectivity {#metric-connectivity}

Connectivity is not a single generic Boolean property. {{RFC2678}} defines several Type-P connectivity metrics, including instantaneous and interval, unidirectional and bidirectional, and temporal connectivity. Interpretation therefore needs to identify the specific connectivity metric, Type-P, endpoints, direction or directions, and time semantics involved.

A negative connectivity result establishes lack of connectivity under the conditions of the selected metric, but it does not identify the cause or location of the failure by itself. Possible explanations include routing or forwarding state, filtering, endpoint behavior, security policy, or measurement-system behavior.

Do not generalize a positive result beyond the semantics of the selected metric. In particular, an interval connectivity result can establish that connectivity existed at one or more relevant times within an interval without establishing continuous connectivity throughout that interval. Likewise, connectivity for one Type-P or direction does not establish connectivity for other packet types, traffic classes, directions, paths, or times.

## Packet Loss {#metric-loss}

{{RFC7680}} defines the Type-P-One-way-Packet-Loss singleton result as either successful reception or loss for a packet sent from a specified source to a specified destination at a specified time. The loss-threshold waiting time, Tmax, defines loss: the metric classifies a packet as lost when the destination does not receive it within Tmax. The singleton loss result is therefore closely related to the corresponding one-way delay result: reception produces a finite delay, while non-reception within Tmax produces an undefined delay.

Preserve these semantics. A packet that the metric classifies as lost does not identify a discard location or cause; the metric establishes only that the destination did not receive the packet within the defined conditions. The configured waiting threshold separates a very large delay from a loss result, so Tmax forms part of the measurement context. Type-P, path information when available, and measurement error or calibration information can also matter.

Stream and statistical loss results add another interpretation layer. A loss ratio summarizes singleton outcomes over a defined sample, and the packet-generation or sampling process determines which times and packets the sample represents. Do not treat two loss ratios as directly comparable merely because their numerical values match when their Type-P, Tmax, sampling discipline, intervals, or packet populations differ.

Packet loss can also change the population available to other metrics. A packet that is lost does not contribute a finite one-way delay sample, can make an IP packet delay variation value undefined for a selected pair, and can interact with reordering and capacity qualification.

## Delay {#metric-delay}

{{RFC7679}} defines Type-P-One-way-Delay for a packet sent from a specified source to a specified destination at a specified time. A finite result represents the elapsed wire time from transmission of the first bit at the source to reception of the last bit at the destination. If the destination does not receive the packet within the loss-threshold waiting time, Tmax, the metric reports an undefined one-way delay. {{RFC2681}} defines corresponding round-trip delay metrics with different directional semantics.

A measured delay value establishes the delay defined by the metric; it does not decompose that value into propagation, serialization or transmission, queueing, processing, timestamping, or other components. An increase or decrease in delay therefore does not by itself identify which component changed. Path changes, queueing, endpoint or reflector processing, tunneling, and measurement-system behavior can all affect observed delay.

The measurement method and its error model also affect the result. For one-way delay, clock synchronization, accuracy, resolution, and the relationship between host time and wire time contribute to measurement uncertainty. Type-P, packet size, the selected path, and Tmax are also part of the measurement context. A delay statistic covers only packets that produce defined delay values; it does not characterize packets that the metric classifies as lost or packets outside the measured population.

## Delay Variation {#metric-delay-variation}

{{RFC3393}} defines IP Packet Delay Variation (ipdv) for a selected pair of packets as the difference between their one-way delays. The interpretation of an ipdv value therefore depends on the underlying one-way delay metric and on the selection function that identifies the two packets. Parameters such as the packet stream, packet size, Type-P, measurement interval, and selection function are part of the meaning of the result.

If either selected packet does not produce a defined one-way delay, the corresponding singleton ipdv value is undefined. Loss can change not only the observed distribution of delay variation but also which packet pairs contribute defined values. A singleton ipdv value alone does not support statistical inference; sample construction and the statistic that summarizes the sample also matter.

Because ipdv is a differential measurement, a constant clock offset can cancel, but relative clock skew or drift over the interval between selected packets can still contribute error. Delay variation is therefore not independent of clock behavior. The term "jitter" has several operational meanings; when referring to the {{RFC3393}} metric, the defined ipdv terminology is less ambiguous.

Increased or changed delay variation can be consistent with variable queueing, path changes, multipath behavior, scheduling, endpoint processing, or measurement-system effects. The metric result alone does not identify which explanation applies.

## Packet Reordering {#metric-reordering}

{{RFC4737}} defines a reordered singleton and several sample metrics that quantify different dimensions of packet reordering. Identify the reordering metric that the report uses, the original packet-order information, the stream characteristics, and the relevant thresholds and parameters. Reordering metrics operate even when a stream also contains lost or duplicate packets.

A sequence discontinuity does not by itself establish reordering. When a received packet indicates that earlier packets are missing, later observations can classify those packets as reordered or lost. A later arrival confirms reordering only when the packet appears in a position that violates the original sequence. The selected metric therefore needs to distinguish a temporary sequence gap from a confirmed reordered packet.

Loss, duplication, and reordering interact in the receiver's observation of the stream. Waiting thresholds, sequence-number interpretation, duplicate handling, packet-size patterns, and the measurement window can affect sample results and derived ratios. A reordering result establishes ordering behavior for the measured stream, but it does not by itself identify the mechanism that produced it. Multipath forwarding, path changes, parallel processing, and link-layer behavior are possible explanations that require additional evidence.

## Packet Duplication {#metric-duplication}

{{RFC5560}} defines one-way packet arrival count and one-way packet duplication for an individual packet. The duplication value counts additional uncorrupted and identical copies that the destination receives within the interval between the send time and maximum waiting time, T0. If exactly one copy arrives, the duplication value is zero; if no copy arrives within the interval, the duplication metric is undefined rather than a positive duplication value.

Packet identity drives the interpretation. {{RFC5560}} requires the implementation to specify which information fields it compares when determining whether packets are identical, and identity does not require every bit in the received packets to match. Fragmentation and packet-field changes along the path can affect how the measurement recognizes copies.

The measurement method also matters. In an active system, the sender can avoid transmitting multiple packets with identical identifying information within the relevant interval. In a passive system, the original source can transmit distinct packets that appear identical under the available identification fields, creating false positives if the observation system cannot distinguish them. Apparent duplication can also result from capture or correlation behavior. A duplication result therefore establishes additional observed copies according to the metric's identity rules, not the network mechanism that produced those copies.

## Capacity-related Metrics {#metric-capacity}

Capacity-related results depend strongly on terminology and metric definition. {{RFC5136}} distinguishes IP-layer capacity, usage, utilization, and available capacity and emphasizes that capacity is meaningful only with respect to a specified protocol layer, Type-P, path or link, time, and interval. Do not conflate these quantities with one another or with generic "bandwidth" or application throughput.

As a concrete IPPM measurement example, {{RFC9097}} defines Type-P-One-way-IP-Capacity and Maximum IP-Layer Capacity using active methods. {{RFC9097}} determines Maximum IP-Layer Capacity over specified sub-intervals and qualifies the result with other performance metrics and thresholds. The Type-P, number of flows, offered load, sub-interval duration, test interval, qualification criteria, sender and receiver behavior, and competing traffic are therefore part of the interpretation context. {{RFC9097}} also requires related loss and round-trip-delay results to accompany capacity reporting.

A change in a capacity-related result does not identify whether path capacity, competing traffic, shaping or policing, packet treatment, endpoint limits, physical-layer conditions, or the measurement procedure changed. Some active capacity methods can perturb the measured environment because they offer substantial load by design. Tie the reported value to the selected capacity metric rather than a generic throughput result, and report measurement-induced effects when they affect interpretation.

## Cross-Metric Interpretation {#metric-cross}

Operational analysis often combines several metrics because no single metric captures all aspects of performance. Such combinations can help when measurements cover compatible traffic populations, paths, directions, and time intervals. {{RFC6703}} shows that the intended use of the measurements can shape the choice and reporting of loss, delay, delay variation, reordering, and capacity-related results.

Before drawing a joint conclusion, verify that the metrics refer to comparable scopes and measurement conditions. For example, do not combine a delay statistic from one Type-P or traffic class with a loss ratio from a different path or interval. Likewise, interpret a capacity result qualified with particular loss or delay thresholds together with those associated measurements, not unrelated observations from another test.

Multiple metrics can strengthen or weaken an operational hypothesis, but their combination does not by itself establish causality. Multi-metric evidence becomes strongest when measurements align packet or traffic populations, paths, directions, parameters, and timing, and when the measurement method or analysis defines the relationship among the metrics.

# Considerations for Active Performance Measurements {#active}

Active Methods of Measurement generate a packet stream for measurement and provide substantial a priori knowledge of that stream, including its endpoints and relevant packet or stream characteristics {{RFC7799}}. This control supports repeatable experiments and tests of specific hypotheses. At the same time, the generated stream can differ from production traffic and can influence the measured network conditions. Interpretation of an active result therefore depends on both the metric definition and the characteristics and effects of the active measurement stream.

## Active Measurement Stream and Representativeness {#active-representativeness}

The packet and stream characteristics used by an active measurement are part of the interpretation context. Relevant characteristics can include source and destination addresses, Type-P or equivalent packet-treatment parameters, transport ports, packet sizes, DSCP values, flow labels, encapsulation, packet rate, packet timing, test duration, and endpoint placement.

An active measurement can characterize the specified measurement stream accurately without representing another application, traffic class, flow, or packet population. Differences in packet or stream characteristics can affect path selection, queue selection, filtering, policing, security treatment, and endpoint processing. Reports that generalize an active result beyond the measurement stream should explain why the measurement traffic represents the traffic or service of interest.

## Measurement-induced Effects {#active-induced-effects}

Because active methods add a generated stream to the network, the measurement activity can influence the measured quantity {{RFC7799}}. The significance of this effect depends on the metric and the measurement stream. A sparse probe stream can have little effect on network conditions, while a sustained or high-rate stream used for a capacity-related measurement can increase queueing, contention, congestion, or network adaptation.

The offered load, packet size, timing pattern, duration, and concurrency of the active stream are part of the measurement context, together with any additional loading traffic used by the method. Do not present a result as describing an unperturbed network when the measurement changes the conditions experienced by the measured stream or other traffic.

## One-way Active Measurements {#active-oneway}

One-way active measurements associate results with a specific source-to-destination direction. This is useful for isolating directional behavior, but different metrics introduce different interpretation requirements.

For a loss metric, the applicable definition and parameters classify an expected packet as lost when the receiver does not receive it, while leaving the discard location and operational cause unknown. For one-way delay and delay variation, timestamp semantics, clock synchronization, clock accuracy, and the received packet population form part of the interpretation context. For reordering or duplication metrics, the controlled source stream and packet-identification information can help establish expected packet order and correspondence, but the metric-specific definitions still determine how events are classified.

Interpret a direction-specific active result as a result for that direction and measurement stream, not as evidence that a specific device, link, queue, or operational mechanism caused the observed behavior.

## Two-way and Reflected Active Measurements {#active-reflected}

Two-way or reflected active measurements introduce an additional interpretation boundary because the observed exchange can include a forward path, processing at a reflector or remote endpoint, and a reverse path. A round-trip result characterizes the complete measurement exchange defined by the method; it does not by itself identify the contribution of each direction or of reflector processing.

For loss, a missing response can result from failure to deliver the forward packet, failure to process or reflect the packet, or failure to deliver the response. For round-trip delay, an increased value can reflect conditions on either direction and, depending on the protocol and timestamp semantics, processing at the remote endpoint. Do not infer one-way components from a round-trip result unless the measurement method and available timestamps support that inference.

Reflector-side information, directional measurements, or protocol timestamps can reduce ambiguity, but keep any additional conclusion consistent with the metric semantics and clock requirements.

## OWAMP, TWAMP, and STAMP {#active-protocols}

OWAMP, TWAMP, and STAMP illustrate different active measurement arrangements. OWAMP supports measurement of one-way characteristics such as one-way delay and one-way loss {{RFC4656}}. TWAMP builds on the OWAMP architecture and adds two-way or round-trip measurement capabilities; the protocol can use reflector timestamps to account for processing behavior {{RFC5357}}. STAMP supports both one-way and round-trip performance measurements, including delay, delay variation, and packet loss {{RFC8762}}.

The protocol used for a measurement does not remove the interpretation considerations associated with the underlying metric. Interpret a protocol result together with the metric definition, session configuration, packet and stream parameters, endpoint roles, timestamp semantics, and the traffic represented by the test.

For reflected protocols, sender-side observations may not localize an operational condition. For protocols that support one-way measurements, the applicable timestamp and clock requirements remain important. Protocol-specific fields can also influence forwarding or other packet treatment when they contribute to path selection or policy.

## Active Measurement over Multipath {#active-multipath}

A single active measurement flow can exercise only one ECMP or link-aggregation component path, depending on the network's path-selection behavior. Changing source or destination addresses, transport ports, IPv6 flow labels, encapsulation, or other fields can move the measurement stream to a different component path and therefore change the measured result.

Controlled variation of flow identifiers can help reveal differential treatment across component paths. However, interpret a changed result after changing a flow identifier as evidence about a different measurement stream that may also use a different path, not as a repeat of the same condition.

## Controlled Active Measurements {#controlled-active}

Use the controllability of active measurement to test operational hypotheses. Examples include measuring both directions, varying flow identifiers, comparing one-way and round-trip measurements, changing packet sizes or traffic classes when appropriate, adjusting the probe schedule or offered load, or measuring from multiple vantage points.

A controlled change is useful when the changed parameter and the resulting change in measurement scope are both understood. Altering packet characteristics can change forwarding, queueing, policy, or security treatment, while changing probe rate can change the load imposed by the measurement itself. Treat a follow-up active test as a related experiment whose differences form part of the interpretation; do not assume that it reproduces the original condition.

# Considerations for Passive Performance Measurements {#passive}

{{RFC7799}} defines Passive Methods of Measurement as methods that observe an undisturbed and unmodified packet stream of interest at one or more Observation Points. Passive methods can provide direct evidence about production traffic without generating the stream of interest, but the resulting interpretation depends on the selected stream, observation coverage, packet correspondence, capture and export behavior, and the traffic that happened to be present during the measurement interval.

## Observation Scope and Stream Selection {#passive-visibility}

A passive result describes traffic that the observation function selects at specified Observation Points. Some passive methods observe all packets at an Observation Point, while others first apply filtering or other selection criteria. The selected stream of interest therefore forms part of the measurement definition and interpretation context.

The absence of a packet from an observation does not, by itself, establish network packet loss. The packet may bypass the Observation Point, fail the selection criteria, or escape capture or export. When an analyst infers loss or another transfer property by comparing observations at multiple points, the conclusion covers only the network segment and packet population that those observations cover.

## Packet Correspondence Across Observation Points {#passive-correlation}

Many passive packet-transfer metrics require observations at more than one point and therefore require a method that establishes correspondence between packets at those points {{RFC7799}}. Encapsulation, header changes, fragmentation, replication, reordering, packet-identification limits, timestamp resolution, or observation-system behavior can disrupt packet correspondence.

For loss-related analysis, compare observations from the same packet population. For delay and delay variation, packet matching and timestamp location and synchronization can affect the result. For reordering and duplication, the method used to establish packet identity and order is part of the interpretation context.

If multiple paths exist, an upstream Observation Point and a downstream Observation Point can cover different packet populations. Incomplete observation coverage or path divergence can create apparent loss or duplication unless the method establishes correspondence and coverage.

## Traffic Availability and A Priori Stream Knowledge {#passive-traffic}

Passive measurement observes only traffic that exists during the measurement interval. If a service, traffic class, endpoint pair, or path carries little or no traffic, the measurement provides little or no evidence about how that traffic would perform under other conditions.

Passive methods also often have less a priori knowledge of the stream than active methods {{RFC7799}}. Applications and users can determine packet rate, flow duration, protocol mix, packet size, endpoints, and path use rather than the measurement system. This is useful for observing real service behavior, but it can make comparisons across intervals or populations difficult when the traffic mix itself changes.

Match the reported scope to the observed traffic population. Generalization to other traffic, paths, loads, or time periods requires supporting evidence.

## Sampling, Aggregation, Export, and Collection Effects {#passive-sampling}

Passive measurement systems can use packet sampling, flow sampling, filtering, aggregation, export, or other data-reduction mechanisms. Capture loss, exporter loss, collector overload, buffering limits, and processing limits can remove observations independently of packet transfer in the measured network.

Do not interpret a missing record or incomplete observation as network packet loss without supporting evidence. Where the distinction matters, the measurement system should account for or report loss and omission that capture, sampling, export, collection, and processing introduce.

Aggregation can hide transient behavior and differences among packet populations, paths, or traffic classes. In addition, RFC 7799 notes that communication or export of passive measurement results can create traffic load; consider this management and collection traffic when it can affect the network or the measured values.

## Selection Effects and Observed Population {#passive-selection}

Passive measurement results reflect packets that traverse the selected Observation Points and that the observation system captures. This can create selection effects. For example, a delay distribution built from packets matched at two Observation Points excludes packets that the network loses, routing diverts outside the observation coverage, or the observation system omits.

Such a result can be valid for the observed population while remaining unrepresentative of the full traffic population. Reports should state the inclusion conditions for the measured sample and note any systematic difference between the observed and unobserved populations.

## Passive Measurement-System State {#passive-system}

Observation interfaces, filters, sampling configuration, timestamping functions, exporter behavior, collector capacity, and analysis pipelines are part of the passive measurement system. Changes or faults in these components can alter the measured data while the packet-transfer behavior of the network remains unchanged.

Check unexpected changes in passive results against measurement-system state as well as network state. The required checks depend on the metric: loss-related interpretation can require capture and export loss information, delay-related interpretation can require timestamp and clock status, and packet correspondence metrics can require validation of identification and matching behavior.

# Considerations for Hybrid Performance Measurements {#hybrid}

Hybrid Methods of Measurement combine attributes of active and passive methods {{RFC7799}}. The particular combination matters for interpretation. RFC 7799 distinguishes, among other cases, Hybrid Type I methods that augment or modify a single stream of interest or otherwise modify its treatment, and Hybrid Type II methods that coordinate two or more streams of interest measured using different method characteristics. Hybrid interpretation should begin by identifying which active and passive attributes are present and how they can affect the result.

The Alternate-Marking Method provides a concrete example on live traffic. {{RFC9341}} notes that the application determines whether the method qualifies as Passive or Hybrid, and {{RFC7799}} classifies implementations that change marking-field values at domain edges as Hybrid Type I.

## Identify the Hybrid Construction {#hybrid-construction}

A hybrid result should identify how the method departs from purely passive observation or purely active generation. Relevant questions include whether the measurement function modifies or augments an existing packet stream, generates additional load or measurement streams, coordinates multiple streams, or controls specific packet or stream characteristics.

These distinctions limit the scope of generalization from the result. A method that marks production packets, one that observes production traffic while adding a separate load stream, and one that analyzes an active stream together with a production stream can all be hybrid, but they do not have the same interpretation properties.

## Measurement Marking, Augmentation, and Treatment Equivalence {#hybrid-marking}

Hybrid Type I methods can add or modify fields used to identify packet groups, measurement intervals, or measurement state. Such augmentation is useful for packet correspondence and measurement control, but it can also change packet treatment if the modified field affects forwarding, filtering, queueing, policy, or other behavior.

Do not assume that a marked or augmented stream represents the corresponding unmodified production stream. RFC 7799 notes that Hybrid results match Passive results only when modified and unmodified packets receive equivalent treatment. Where interpretation depends on such equivalence, establish or validate it rather than assume it.

## Partial Controllability and Traffic Availability {#hybrid-control}

Hybrid methods can combine controlled measurement state with traffic characteristics determined by production applications. For example, a method can control marking periods or metadata while packet timing, packet sizes, endpoints, and offered load remain determined by the production stream.

This combination provides more measurement structure than passive observation while preserving many production-traffic characteristics, but the method cannot control some experimental variables independently. A lack of suitable production traffic limits what the method can measure, and changes in the production traffic mix can affect comparisons across marking periods or measurement intervals.

## Consistency of Measurement State Across Observation Points {#hybrid-consistency}

Hybrid measurement can depend on consistent interpretation of markings, packet groups, timestamps, counters, or other measurement state across Observation Points. Inconsistent marking state, interval boundaries, packet classification, counter resets, or timestamp behavior can produce differences that originate in the measurement function rather than in packet transfer.

Observation points need compatible measurement state, and transitions between measurement intervals need consistent handling. For methods based on counters or timestamps, alignment of measurement periods and clocks is part of the interpretation context.

## Coordinated Streams and Hybrid Type II Interpretation {#hybrid-type2}

Hybrid Type II methods can use two or more coordinated streams of interest with different measurement-method characteristics {{RFC7799}}. Joint analysis of these streams can provide information unavailable from either stream alone, but coordination does not imply that the streams experience identical network treatment.

Before comparing or combining results, check whether the streams use the same paths, traffic classes, packet sizes, timing intervals, endpoints, and network conditions. Differences between an active stream and a production stream can be informative, but do not attribute them to an operational cause when the stream characteristics themselves differ.

## Production Traffic Representativeness {#hybrid-representativeness}

When a hybrid method operates on production traffic, the measured population can closely reflect real service behavior. The result nevertheless applies to the traffic selected, marked, observed, or correlated by the method. Unselected traffic can use different paths, classes, or packet characteristics, and the measurement mechanism itself can introduce selection effects.

Reports should identify the production traffic that the method includes and state whether marking, selection, or augmentation could affect packet treatment or sample composition.

## Sampling, Aggregation, and Collection Effects {#hybrid-collection}

Hybrid methods can rely on counters, packet samples, exported records, or aggregated measurement intervals and can inherit the observation and collection limits of passive measurement. In addition, active components can introduce state, load, or packet modifications that require separate interpretation.

Interpret a hybrid measurement result together with the state of both the measured traffic and the measurement function. Collection loss, sampling, inconsistent marking state, generated load, or differences among coordinated streams can all affect interpretation without implying that the underlying metric definition is invalid.

# Interpretation Context and Supporting Evidence {#context}

Metric definition, parameters, measurement method, and observed population establish the primary meaning of a result. Other information can support or limit a broader interpretation. Its value depends on how closely it matches the measured traffic, path or observation scope, direction, and time interval.

## Measurement Metadata and Intended Use {#reporting}

Useful metadata includes the metric definition or registry entry, fixed and runtime parameters, measurement method, timestamps, direction, measurement or observation points, packet or traffic selection, packet characteristics, sampling or marking configuration, aggregation interval, statistical function, and relevant clock information. {{RFC8911}} requires operators to supply Runtime Parameter values when they execute a measurement and to report those values with the results.

The intended use also matters. {{RFC6703}} shows that intended use shapes measurement design, parameter selection, and reporting. A result sufficient for one purpose can be insufficient for a broader conclusion about other traffic, paths, intervals, or application behavior.

## Path, Routing, and Forwarding Context {#routing-state}

Routing and forwarding information can show whether the measured traffic could have experienced a route change, next-hop change, forwarding inconsistency, tunnel change, or different component path. This information is most useful when the result varies by flow, direction, or time.

Control-plane state does not always identify the data path that packets traverse. When interpretation depends on the actual path, use forwarding-plane or path-specific evidence. In multipath environments, match path information, as far as practical, to the fields and traffic population represented by the metric.

## Resource and Traffic-Treatment Context {#counters}

Interface state, utilization, queue state, discard counters, policer or shaper state, filtering state, and similar information can show whether a proposed explanation is consistent with conditions during the measurement. Queue and discard information can support an interpretation of loss or delay; utilization, competing traffic, traffic class, and qualification conditions can be relevant to capacity results. Validation state and counters provide one form of traffic-treatment context where networks deploy validation functions.

These data are not direct explanations unless their scope matches the measurement. A device counter can cover several traffic populations or time intervals, and an overlapping counter increment does not prove that the measured traffic caused or experienced that increment.

## Measurement-System Context {#system-state}

The measurement system can affect the observations. For active measurements, relevant state includes session status, endpoint or reflector availability, processing limits, timestamping behavior, and local rate limits. For passive and hybrid measurements, relevant state includes capture status, sampling or selection configuration, marking state, exporter behavior, collector capacity, record loss, and processing-pipeline state.

Use this context when either the measured network or the measurement system could cause a gap or change. Separate collection loss, timestamping errors, overload, and configuration changes from the packet-transfer behavior represented by the metric.

## OAM, Telemetry, and Event Context {#oam-telemetry}

OAM mechanisms, telemetry, device logs, and event records can provide state and timing information. {{RFC9232}} describes telemetry as including data generation, collection, correlation, and consumption for network visibility and operation. Such data can show whether a result is consistent with a routing event, interface event, resource condition, policy change, or measurement-system change.

Data granularity limits the correlation. A telemetry series with samples every several minutes cannot localize a short per-packet event within that interval. Path-discovery or OAM packets may also follow a different path or receive different treatment from the traffic that the metric represents.

## Operator- and Controller-Provided Context {#operator-context}

Operators and controllers can provide information unavailable to the measurement system, including maintenance events, intended policy, controller-computed paths, rule-distribution status, customer attachment information, and known failures. Such information can narrow the set of plausible interpretations.

It remains supporting context rather than part of the metric result. Across administrative boundaries, treat incomplete disclosure as uncertainty rather than filling gaps with assumptions.

# Applying the Interpretation Framework {#ops}

This section describes how to use the preceding considerations when analysts compare, correlate, or generalize results. It does not define a troubleshooting procedure.

## Define the Intended Use and Required Scope {#intended-use}

State the conclusion that the measurement aims to support. {{RFC6703}} shows that intended use affects measurement design and reporting. Apply the same principle to interpretation: compare the scope of the conclusion with the scope established by the metric, parameters, measured population, and method.

Make any generalization beyond that scope explicit and support it with evidence or stated assumptions.

## Align Scope and Time Before Correlation {#event-correlation}

External evidence is most useful when it refers to the same packet or traffic population, path or observation scope, direction, and interval as the result. Routing changes, forwarding events, queue counters, policy changes, endpoint events, and measurement-system events can all be relevant, but temporal overlap alone does not establish causality.

Time resolution also matters. Clock synchronization, timestamp location, aggregation period, telemetry sampling interval, and retention resolution limit the precision of a correlation. Do not infer finer causal localization than the available data support.

## Compare Like Measurement Results Before Combining Them {#comparable-results}

Results that share a broad metric name may not support comparison. Check the metric definition, fixed and runtime parameters, output statistic, traffic population, path, direction, interval, and measurement method {{RFC8911}}.

The same applies across Active, Passive, and Hybrid measurements. Different results can reflect different network behavior, but they can also follow from different populations, observation points, sampling rules, packet treatment, or method-induced effects.

## Use Complementary Measurements to Reduce Ambiguity {#complementary}

Different metrics and methods can provide complementary evidence when the analysis makes their scopes explicit. Loss together with delay or queue information can constrain some explanations. Active measurements can test selected packet characteristics or vantage points; passive measurements can show behavior in observed production traffic; hybrid methods can add measurement structure while retaining some production-traffic characteristics.

Do not merge measurements that cover different populations or conditions as if they measured the same event. Their differences can still be useful evidence.

## Treat Follow-up Measurements as Tests of a Hypothesis {#hypothesis-testing}

A follow-up measurement can test whether a proposed explanation is consistent with new observations. Active measurements can vary direction, packet fields, flow identifiers, or vantage points. Passive analysis can compare observation points, traffic classes, paths, or intervals. Hybrid methods can compare marking periods, measurement groups, or selected populations.

Changing the configuration also changes the experiment. A follow-up result can support or weaken a hypothesis, but it does not alter the original result or guarantee reproduction of the original conditions.

## Account for Measurement-Method Effects {#measurement-effects}

Measurement activity can affect the measured environment or the completeness of the observations. Active probes consume network and endpoint resources. Hybrid methods can modify packet fields, treatment, or offered load. Passive methods leave the stream of interest unchanged, but limited capture, export, and collection resources can omit observations; export traffic can also add load.

These effects matter when they are large enough to change the measured property or the completeness of the data.

## Interpretation Across Administrative Boundaries {#admin-boundaries}

A measurement that spans several administrative domains can remain valid even when the observer cannot identify which domain contributed to the result. Internal routing, forwarding, filtering, policing, validation, or measurement-system information may be unavailable.

Coordination can reduce uncertainty, but lack of visibility is not evidence of attribution. Statements about another domain should separate measured facts from explanations that shared operational evidence supports.

## Report Result, Evidence, Inference, and Uncertainty Separately {#reporting-uncertainty}

A report that includes interpretation should distinguish:

* the metric result and its measurement scope;
* the supporting evidence;
* any generalization or inferred operational explanation; and
* remaining uncertainty, assumptions, and plausible alternatives.

Where relevant, include the metric definition or registry entry, fixed and runtime parameters, measurement method, traffic population, observation scope, and interval. This keeps the metric result separate from the reasoning built on top of it.

# Security Considerations {#sec}

This document does not change the security properties of existing measurement, routing, forwarding, or security mechanisms. The measurements and supporting context used during interpretation create the main security considerations.

Do not create broad exemptions for measurement traffic from filtering, policing, or validation. If operations require an exception, limit its scope and keep it consistent with local policy.

Active measurements can expose path selection, filtering, policing, or other traffic-treatment behavior when operators vary packet fields, source addresses, traffic classes, or flow identifiers. Operators should authorize such measurements and keep them consistent with operational policy.

Passive and hybrid measurements can expose information about production traffic, observation points, path behavior, and marking state. Access controls should protect detailed measurement data and configuration.

Supporting context can also be sensitive. Routing and forwarding state, policy configuration, customer attachment information, counters, logs, observation-system state, and controller information can reveal topology, policy, or customer relationships. Reports shared outside an administrative domain should disclose only the information their purpose requires.

The integrity and freshness of supporting context shape the resulting interpretation. Stale, incomplete, inconsistent, or modified telemetry, counters, logs, measurement records, or controller state can lead to incorrect attribution.

# Privacy Considerations {#privacy}

Performance measurement data can contain source and destination addresses, endpoint identifiers, timestamps, flow identifiers, packet fields, traffic classes, marking state, observation-point information, and delivery outcomes. Correlation with operational context can reveal network structure, endpoint activity, customer relationships, or policy behavior.

Passive and hybrid measurements can observe production traffic. Even without payload collection, metadata such as addresses, timing, packet treatment, path information, marking state, and traffic volume can reveal information about users, services, or traffic patterns.

Collect only the information needed for the measurement and its intended interpretation. Reports should omit customer mappings, detailed policy, internal topology, or other sensitive information that the intended purpose does not require. Local policy should govern access and retention.

Across administrative boundaries, a domain may disclose the measured result or a supported interpretation without disclosing the underlying internal evidence. Reporting and operational procedures should preserve that distinction.

# IANA Considerations {#iana}

This document has no IANA actions.

--- back

# Acknowledgements {#ack}
{: numbered="false"}

The author thanks Giuseppe Fioccola for reviewing earlier versions of this document, and Ruediger Geib, Tim Chown, and Ike Kunze for their helpful comments and discussion.