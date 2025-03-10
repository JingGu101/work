# Horizontal Pod Autoscaler
- is a featurein k8s that changes the number of running pods based on observed metrics.
- During each interval, the HPA checks if the current scaling metric value has deviated more than 10% from the target.
  - desiredReplicas = ceil [ currentRelicas * (currentMetricValue / desiredMetricValuecoper
  - pro)]