# kustomize-vpc-cni
Example configuration to run multiple customized AWS VPC CNI DaemonSets in an EKS cluster using Kustomize.

## Overview
This repository provides a solution for running multiple VPC CNI DaemonSets with different configurations in the same cluster. This is particularly useful when:
- Different node groups require different IP allocation settings
- You need to optimize IP address management for specific workloads
- You want to apply different configurations based on node labels

The `aws-node-default` installation serves as a catch-all DaemonSet for nodes that don't match the selectors of other customized VPC CNI DaemonSets.

## Prerequisites
- Access to an EKS cluster
- kubectl installed and configured
- Basic understanding of Kubernetes and VPC CNI

## Installation
1. Clone this repository:
```
git clone https://github.com/david-a-aws/kustomize-vpc-cni.git
cd kustomize-vpc-cni
```
    
2. Customize the configuration:

- Modify overlays/stateless/node-affinity.yaml with your desired node selectors
- Update overlays/stateless/env-vars.yaml with appropriate IP targets:
  -  WARM_IP_TARGET: Number of IPs to pre-allocate
  -  MINIMUM_IP_TARGET: Minimum IPs to maintain per node
- Adjust any other settings in the overlay directories as needed
  
Important note: Make sure to add the node groups which will have a customised aws-node to default/node-affinity.yaml like so:
```
- key: nodegroup
  operator: NotIn
  values:
  - stateless
```

4. Apply the configurations:
```    
kubectl apply -k overlays/default/
kubectl apply -k overlays/stateless/
```
  
## Configuration Structure
```
.
├── base/
│   ├── aws-vpc-cni.yaml    # Base VPC CNI manifest
│   └── kustomization.yaml
└── overlays/
    ├── default/            # Default catch-all configuration
    │   ├── kustomization.yaml
    │   └── node-affinity.yaml
    └── stateless/          # Custom configuration for stateless workloads
        ├── env-vars.yaml
        ├── kustomization.yaml
        └── node-affinity.yaml
```

## Upgrading VPC CNI Version

1. Update the version(s) in base/aws-vpc-cni.yaml and ensure version compatibility with your EKS cluster version
2. Apply the updates:
```
kubectl apply -k overlays/default/
kubectl apply -k overlays/stateless/
```
    
## Verification

After applying changes, verify the installation:
```
# Check DaemonSet status
kubectl get ds -n kube-system aws-node-default aws-node-stateless

# Verify pod placement
kubectl get pods -n kube-system -l k8s-app=aws-node-default
kubectl get pods -n kube-system -l k8s-app=aws-node-stateless
```

## Uninstalling
```
kubectl delete -k overlays/default/
kubectl delete -k overlays/stateless/
```

## Troubleshooting
Common issues and solutions:
- If pods aren't scheduling, verify node selectors match your node labels
- Check pod logs for configuration errors: kubectl logs -n kube-system -l k8s-app=aws-node-default
- Ensure RBAC permissions are correct for both DaemonSets

## Limitations
- This configuration shares RBAC resources between DaemonSets
- Requires careful planning of node selectors to avoid conflicts
- No automatic handling of CNI version compatibility checks

## Disclaimer

This is a community-maintained example. Use at your own risk and test thoroughly before deploying to production environments.
