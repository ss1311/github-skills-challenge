# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)



Task 1: Set Up the Project

o operational data- The file used [service_data.json] It contains telemetry for payment-service, including: Response time,CPU utilization,Timestamp and service name,Memory utilization

o metrics and logs - The file used [service_data.json] contains response-time, CPU, memory, timestamp, log-level, and message fields for the monitored service.

o anomaly detection-The file used [anomaly_detector.py]  applies thresholds of 500 ms response time, 80% CPU, and 80% memory. It returns an anomaly event with reasons and the original record.

o event production-The file used[event_producer.py] publishes valid anomaly events to a topic.

o event topics-The file used[event_topic.py]provides an in-memory topic with publish, retrieve, and clear operations.

o event consumption-The file used[event_consumer.py] reads messages from a topic.

o final AIOps processing-The file used [aiops_pipeline.py] Loads the JSON data, evaluates each record, publishes detected anomalies, consumes events, and returns processing statistics and event lists.



Service monitored: A payment service that emits response-time, CPU, memory, and log-level telemetry.
Operational problem: Detect service degradation, including slow responses, resource saturation, and error logs, before it affects payment processing.
Purpose of AIOps: Automatically analyze, identify anomalous events, and publish them for downstream operational response.

Task 2: Analyse Logs and Metrics

1. Which fields represent metrics.  `response_time_ms`, `cpu_percent`, and `memory_percent`

2. Which fields represent log information `log_level` and `message` represent log information.  

3. How timestamps are used in the operational data.  `timestamp` records when each observation occurred. The one-minute sequence from 10:00 through 10:09.

4. Which observations appear to represent normal behaviour The observations from 10:00-10:04 and 10:07-10:09 appear normal. 

5. Which observations appear to represent unusual behaviour.  The observations at 10:05 and 10:06 appear unusual. They contain `ERROR` messages about payment and database timeouts, response times of 610 ms and 640 ms, and the 10:06 observation also has CPU utilization of 94% and memory utilization of 91%.

Task 3: Validate Event Processing

The event-processing workflow uses the provided components as follows:

- **Event/message:** `AnomalyDetector` creates an event when a record exceeds a metric threshold or contains an `ERROR` log. Each event includes the service, timestamp, anomaly reasons, and original source record.
- **Producer:** `EventProducer` receives the detected event and publishes it to the configured topic.
- **Topic:** `EventTopic` stores the published event in the in-memory `anomaly-events` topic.
- **Consumer:** `EventConsumer` reads the event from the same topic.
- **Downstream AIOps component:** `run_pipeline` consumes the event and returns it in `events_consumed`, where the main program prints the readable anomaly report.

Execution result from `python3 src/aiops_pipeline.py`:

- 10 operational records processed.
- 2 anomaly events detected and published.
- 2 events consumed from `anomaly-events` and passed to the downstream report.
- The event at `2026-09-20T10:05:00` reported high response time and an error log.
- The event at `2026-09-20T10:06:00` reported high response time, high CPU, high memory, and an error log.
- The unchanged test suite passed: 8 tests passed.

Task 5: Investigate and Correct the Workflow

The workflow contained three issues that were corrected within the existing architecture:

1. **Incorrect log condition**
	- **Affected component:** `AnomalyDetector` in `src/anomaly_detector.py`.
	- **Cause:** The detector checked for `WARNING`, but the operational data uses `ERROR` for concerning log events.
	- **Correction:** The condition now detects `record["log_level"] == "ERROR"` and adds `Error log detected` to the event reasons.
	- **Verification:** The 10:05 and 10:06 records are flagged for their error logs; normal `INFO` records are not flagged by log level.

2. **Producer and consumer used different topics**
	- **Affected component:** `run_pipeline` in `src/aiops_pipeline.py`.
	- **Cause:** Events were published to `service-events`, while the consumer read from a separate `anomaly-events` topic.
	- **Correction:** The producer and consumer now share the same `anomaly-events` topic instance.
	- **Verification:** Both detected events are consumed and included in the downstream result.

3. **Direct script execution import failure**
	- **Affected components:** `src/aiops_pipeline.py`, `src/event_producer.py`, and `src/event_consumer.py`.
	- **Cause:** Package-relative imports work with `python3 -m src.aiops_pipeline`, but fail when running `python3 src/aiops_pipeline.py` directly.
	- **Correction:** The components use package-relative imports with a fallback to direct imports when executed as scripts.
	- **Verification:** The pipeline can be run from the repository root with `python3 src/aiops_pipeline.py`; the recorded run processed 10 records, detected 2 anomalies, and consumed 2 events. The unchanged test suite passed all 8 tests.
