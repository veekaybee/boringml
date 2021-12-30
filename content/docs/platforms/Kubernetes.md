+++
title = "Kubernetes"
type = "docs"
bookToC = false
weight = 6
+++

# Kubernetes Cheat Sheet

What is it? 
Kubernetes is just a bunch of Docker containers that are abstracted away from you and that you operate on in a group.

Frequently used shortcuts: 

```bash
// see clusters available to you
kubectl config view

// pick cluster
kubectl config use-context dca-production

// which namespaces do we have
kubectl get namespace | wc -l   65

// how many pods do we have running in our namespace? 
kubectl -n namespace get pods | wc -l  190

// All current allocated pods
kubectl get pods --all-namespaces | wc -l  4038

// pods with container names per spec
kubectl get pods -n namespace -o jsonpath="{.items[*].spec.containers[*].image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```
