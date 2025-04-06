# MetalLB Configuration

MetalLB is a load-balancer implementation for bare-metal Kubernetes clusters, allowing you to create services of type `LoadBalancer` in environments that don't natively support this feature.

## Configuration Details

- **IPAddressPool**: Defines the range of IP addresses that MetalLB can allocate.
  - Address Range: `10.0.0.201-10.0.0.209`

- **L2Advertisement**: Advertises the allocated IPs using Layer 2 mode.

## Deployment

To apply the MetalLB configuration:

```bash
kubectl apply -f metallb-config.yaml
```

Ensure that the `metallb-system` namespace and MetalLB controller are already deployed in your cluster.