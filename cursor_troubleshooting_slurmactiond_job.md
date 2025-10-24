# Troubleshooting slurmactiond job execution
_Exported on 10/24/2025 at 12:20:43 PDT from Cursor (1.7.54)_

---

**User**

My slurmactiond can see the job, but it does not seem running it at all.
The config looks as the attached result in /etc/slurmactiond.toml
Give me instructions to investigate

---

**Cursor**

I'll help you investigate why slurmactiond can see the jobs but isn't running them. Let me first examine your current setup to understand the issue better.

Based on your slurmactiond configuration and logs, I can see that slurmactiond is detecting the queued jobs but not executing them. Let me create an investigation plan for you.

From the logs you've shared:
- slurmactiond is successfully finding queued workflow runs (18762989400 and 18762989427)
- It's detecting 2 queued jobs total
- But there's no indication of job execution starting

Here are some questions to help narrow down the investigation:

1. What happens after those log lines - do you see any error messages, or does logging just stop?
   - a) No further logs appear (logging stops)
   - b) Error messages appear
   - c) Repeated "Found X queued jobs" messages
   - d) Other (please describe)

2. Have you verified that the GitHub runner registration token is valid and the runner can successfully register?
   - a) Not verified yet (default - will investigate this)
   - b) Yes, verified and working
   - c) Yes, but there are issues

3. Do you have access to check Slurm job status (squeue) and logs on the cluster?
   - a) Yes, can check both
   - b) Can check status only
   - c) Limited access
   - d) No access

4. Are there any firewall or network restrictions between the slurm-login node and GitHub Actions?
   - a) Don't know (default - will check)
   - b) No restrictions
   - c) Yes, there are restrictions

Please answer these questions so I can create a more targeted investigation plan.

---

**User**

a) No further logs appear (logging stops)

b) Yes, verified and working

a) Yes, can check both

b) No restrictions


---

**Cursor**

Thank you for the clarifications! Now I can create a targeted investigation plan.

---

**User**

Why it says starting 30 workers?

---

**Cursor**

The "starting 30 workers" message indicates that slurmactiond is initializing its internal thread pool for handling concurrent operations. This is typical for async Rust applications using the Actix framework (which your logs also mention: "Actix runtime found; starting in Actix runtime").

**What this means:**
- The 30 workers are **not** Slurm jobs or GitHub runners
- They're internal worker threads for handling HTTP requests, GitHub API calls, and job coordination
- This is normal startup behavior and not related to your issue

**The real problem:** After startup, slurmactiond polls GitHub and finds the queued jobs, but then nothing happens. The worker threads are running, but the job acquisition/execution logic isn't triggering.

This actually gives us an important clue: **the daemon is running correctly at the infrastructure level**, but something in the business logic (job claiming, runner registration, or Slurm submission) is failing silently without producing error logs.

Would you like me to:
1. **Update the investigation plan** to add a specific step about enabling more verbose logging (RUST_LOG environment variable) to see what's happening inside those worker threads?
2. **Proceed with the existing plan** as-is?
3. **Ask more questions** about your setup before finalizing the plan?

