# 📘 A/B Testing Deployment Strategy -- Kubernetes

A/B Testing is a deployment strategy used to run **two versions of an
application simultaneously** (Version A and Version B) and distribute
user traffic between them to compare performance, UX behavior, or
validate new features.

This strategy is widely used in **production environments** for gradual
experimentation and risk-free validation.

------------------------------------------------------------------------

## 🌐 What is A/B Testing Deployment?

A/B Testing allows you to:

-   Run the **current version (A)** and **new version (B)** in parallel.

-   Split incoming traffic (e.g., 90/10, 80/20, 50/50).

-   Observe:

    -   User interaction
    -   Conversion rate
    -   Feature performance
    -   UI/UX impact
    -   Latency & error rate

-   Promote the best-performing version to full production.

------------------------------------------------------------------------

## 🛠 How It Works

### **Version A (Control):**

The existing, stable production version. Majority of traffic initially
flows here.

### **Version B (Experiment):**

The new version with UI changes, new features, or optimizations.

### **Traffic Split:**

A router, ingress, or service mesh divides traffic between A and B based
on defined percentages.

### **Analyze & Decide:**

-   If B performs well → Roll out fully.
-   If B fails → Revert 100% traffic back to A.

------------------------------------------------------------------------

## ✔️ Pros & ❌ Cons

  -----------------------------------------------------------------------
  Pro's                                 Con's
  ------------------------------------- ---------------------------------
  Real-world validation with actual     Requires monitoring/analytics
  user behavior                         tools

  Reduces risk by exposing only a small Users may experience inconsistent
  % of users                            interfaces

  Compare performance, UX, and feature  More complex routing
  impact                                configurations

  Gradual rollout approach              Requires additional compute
                                        resources
  -----------------------------------------------------------------------

> **Note:** A/B Testing is suitable for **production environments**,
> especially product-based applications.

------------------------------------------------------------------------

## 📦 Prerequisites

Before implementing the A/B testing deployment, ensure you have:

1.  EC2 Instance (Ubuntu OS)
2.  Docker installed & configured
3.  Kind installed
4.  Kubectl installed
5.  A running Kind cluster\
    Create using:

``` bash
kind create cluster --config kind-config.yml --name dep-strg
```

------------------------------------------------------------------------

## 🚀 Steps to Implement A/B Testing Deployment

### 1️⃣ Create Namespace

``` bash
kubectl apply -f ab-testing-ns.yml
```

### 2️⃣ Deploy Version A (v1) and Version B (v2)

``` bash
kubectl apply -f online-shop-v1.yaml
kubectl apply -f online-shop-v2.yaml
```

### 3️⃣ Monitor Pod Status

``` bash
watch kubectl get pods -n ab-testing-ns
```

### 4️⃣ View All Resources

``` bash
kubectl get all -n ab-testing-ns
```

------------------------------------------------------------------------

## 🌐 Accessing Version A and Version B

### 🔵 Access Version A (v1)

``` bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-v1-service 30001:3001 -n ab-testing-ns &
```

Open:

    http://<Public_IP>:30001

### 🟢 Access Version B (v2)

``` bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-v2-service 30000:3000 -n ab-testing-ns &
```

Open:

    http://<Public_IP>:30000

> **Note:** Both versions must have visual or behavioral differences for
> proper testing.

------------------------------------------------------------------------

## 🔄 Splitting Traffic Between A and B

Traffic can be split using:

-   NGINX Ingress
-   AWS ALB Ingress
-   Istio / Linkerd (Service Mesh)
-   API Gateway routing rules

### Example Traffic Distribution:

-   **80% → Version A**
-   **20% → Version B**

After evaluating metrics, adjust to:

-   **50/50**
-   **0/100 (full rollout)**

------------------------------------------------------------------------

## 🎯 Post-Analysis Decision

  If Version B Succeeds                 If Version B Fails
  ------------------------------------- ----------------------------
  Promote B as new production version   Keep Version A
  Increase traffic gradually            Redirect 100% traffic to A

This minimizes deployment risk and improves decision-making using real
data.

------------------------------------------------------------------------

## 🧹 Cleanup

Delete the Kind cluster:

``` bash
kind delete cluster --name dep-strg
```

------------------------------------------------------------------------

## ⚠️ Troubleshooting

If you see:

``` bash
error: lost connection to pod
```

It occurs because `kubectl port-forward` breaks when pods restart.

✔ Simply re-run the port-forward command.

This issue does **not** appear on EKS/GKE/AKS since services are exposed
via:

-   NodePort
-   LoadBalancer
-   Ingress
