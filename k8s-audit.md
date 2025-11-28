There are 9 questions I ask to audit any K8s cluster in under 1 hour:

1. How many namespaces are in active use?
If there are 50 namespaces but only 12 active teams, you’re hoarding infra debt.

2. Are your default resource limits set in Helm charts?
No? Congrats. You’ve got unlimited CPU parties burning your budget.

3. How are you handling horizontal pod autoscaling?
Set and forget is not optimization. Most HPA configs are outdated or misfiring.

4. Is your ingress controller running on a public IP by default?
That’s not just waste. That’s an attack vector.

5. What’s the oldest image running in your cluster?
If you don’t know, your dev-prod parity is probably broken.

6. Are CrashLoopBackOff pods older than 24 hours still hanging around?
You’re not debugging. You’re paying rent for dead pods.

7. Are node pools segregated by workload type (stateless, cron, batch)?
If not, your workloads are stepping on each other. And cost tracking is a mess.

8. Are your developers pulling :latest images in production?
That’s not DevOps. That’s roulette.

9. Is your cluster set to autoscale down during off-peak?
99% of prod clusters I audit never scale down. That’s 30% of cost right there.
