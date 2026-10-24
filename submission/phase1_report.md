# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->240.7<!--/answer--> | <!--answer:vm_warm_median_ms-->210.2<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->1730.3<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->635.2<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1350.2<!--/answer--> | <!--answer:function_warm_median_ms-->675.4<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Warm median latency (ms)
VM             210.2 | ##########
Cloud Run      635.2 | ##############################
Cloud Function 675.4 | ################################
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM does not automatically create new instances when traffic increases. Its capacity is limited by the selected e2-micro machine and the application configuration. Cloud Run automatically scales container instances according to demand, up to the maximum of three instances specified during deployment. Cloud Functions Gen 2 also scales automatically using Cloud Run infrastructure. Its capacity is constrained by its configured instance limits and concurrency settings. Unlike the VM, both serverless deployments can scale down to zero when idle.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
Deploying on the VM required creating a custom VPC, subnet and firewall rules, provisioning an e2-micro instance, installing Docker, configuring permissions, pulling the image and starting the container manually. For Cloud Run, I built and pushed the Docker image to Artifact Registry, then deployed it using gcloud, specifying the port and scaling limits. Cloud Functions Gen 2 required only the Python source files, dependencies, runtime and entry point. Google Cloud automatically built and deployed the function. The VM requires the most infrastructure management, while the serverless options simplify deployment and maintenance.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
marouane_jenane@cloudshell:~/EurecomGPT-lab (project-d6e19430-0553-4179-aec)$ gcloud run services list
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-86708531510.us-central1.run.app
LAST DEPLOYED BY: service-86708531510@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-24T00:04:12.788304Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-86708531510.us-central1.run.app
LAST DEPLOYED BY: marouane.jenane@eurecom.fr
LAST DEPLOYED AT: 2026-10-23T23:42:15.207368Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output of `gcloud run services list` shows two services: `echo-bot`, deployed manually using Cloud Run, and `echo`, created automatically when deploying the second-generation Cloud Function. This demonstrates that Cloud Functions Gen 2 uses Cloud Run infrastructure to execute functions. Both platforms manage containers and can scale down to zero when idle, explaining why their cold-start and warm latencies are relatively similar. In my measurements, Cloud Run recorded 1730.3 ms cold latency and 635.2 ms warm median, while Cloud Functions recorded 1350.2 ms and 675.4 ms respectively. The main difference is the deployment model: Cloud Run accepts a container image, whereas Cloud Functions accepts source code and automatically builds the runtime environment.
<!--/answer-->
