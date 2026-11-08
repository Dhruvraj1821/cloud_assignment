## Page 1/3 - Overview and learning

**Cloud Computing Assignment - Reflection**  
**Name:** Dhruvraj 
**Roll Number:** 230103003

In this assignment, I built an end-to-end command parsing system on local
Kubernetes. Phi-3.5 labelled 3,000 supplied commands using two parallel jobs.
I cleaned and stored the data in SeaweedFS, trained two Pythia-70M student
models on CPU, selected version 2, served it through FastAPI, and deployed it
on k3d with health probes and autoscaling.

My main learning was that an ML service requires much more than model training.
Data quality, checksums, repeatability, API validation, deployment, monitoring
and failure recovery are equally important. Deterministic sharding allowed two
workers to process the dataset without missing or repeating IDs. Checksums
proved that the same dataset and model moved between storage and release.

The networking task improved my understanding of VLSM and stateless network
ACLs. I allocated the 10.20.0.0/16 block from the largest requirement to the
smallest. I learned why reply traffic needs explicit ephemeral-port rules and
why a specific deny rule must appear before general allow rules.

I also learned how Kubernetes Deployments, Services, ingress, Secrets, probes,
resource limits, HPA and PodDisruptionBudgets work together during real load
and failures.



The first challenge was the slow teacher-labelling process. I divided 3,000
records into two deterministic shards of 1,500. Both jobs ran in parallel, and
an automated check confirmed 3,000 unique IDs with none missing. Cleaning
removed 153 invalid-slot records and 49 duplicate commands, leaving 2,798
clean examples.

The second challenge was training on an 8 GB M1 machine. I selected the small
Pythia-70M model, limited training to 1,500 examples, used batch size four, and
calculated loss only on target JSON tokens. I trained v1 for three epochs and
v2 for four epochs. Version 2 achieved 99% valid JSON, 79% intent accuracy and
46% full-frame accuracy, so it passed the 95% validity gate.

The biggest deployment challenge was CPU inference speed. The first Locust run
with a one-CPU limit had a 58.82% failure rate during HPA scale-up. After a
controlled resource test, I raised the limit to two CPUs and repeated the full
test. Success improved to 95.12%, median latency became 8.1 seconds, and p95
latency was 26 seconds.

I also tested failures instead of assuming reliability. Kubernetes replaced a
deleted pod in eight seconds while all 25 health probes returned HTTP 200.
Malformed model output produced a controlled HTTP 500 error and normal service
was restored afterward.


The main finding was that evidence makes a cloud project trustworthy. Logs,
checksums, evaluations, Locust results and CI outputs made each important claim
traceable. The untrained model produced 0% valid JSON, while trained v2 produced
99%. The student matched the teacher's 79% intent accuracy and improved
full-frame accuracy from 40% to 46%, although this is not sufficient for
critical automation.

The load test showed that autoscaling does not instantly solve latency. HPA
scaled correctly, but two requests timed out during the initial transition
from two to four pods. Using the supplied rate card, I estimated INR 25.3194
per 1,000 successful requests. This is an assumption-based estimate, not a real
cloud bill.

The evaluation had only 100 examples, the load test lasted two minutes, and
the cluster used one local node. Future improvements would include a larger
human-reviewed validation set, all clean training data, constrained JSON
decoding, and a smaller or quantized model. I would also pre-warm replicas for
traffic bursts, tune HPA behaviour, add request backpressure, run longer load
tests, and use a multi-node cloud cluster with monitoring and alerts.

Overall, I learned to connect data engineering, networking, machine learning,
containers, Kubernetes, testing, security and cost analysis as one system. I
also learned to clearly separate measured results from assumptions.

**GitHub repository:**  
https://github.com/Dhruvraj1821/cloud_assignment

