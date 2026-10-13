**CNI stands for Container Network Interface.** It is a standard way for a container runtime to ask networking software: **“Please connect this Pod to the network.”**

Remember the setup you learned: create interfaces, connect a veth pair, assign an IP, and configure routes. CNI lets software arrange the necessary networking automatically. The exact setup depends on the networking solution. [cni.dev](https://www.cni.dev/docs/?utm_source=chatgpt.com)

Think of connecting a new house:

- The **Pod** is the house needing a connection.
- The **container runtime** requests the connection.
- **CNI** specifies the common format for that request.
- A **CNI plugin** is the software that performs the networking job.

**CNI defines how to ask; the plugin does the work.**

When Kubernetes starts an ordinary Pod, this is what happens:

1. **Kubelet asks the container runtime**, such as containerd, to start the Pod.
2. **The runtime creates the Pod’s network namespace**—its separate network view.
3. **The runtime calls the configured CNI plugins.**
4. **The plugins configure networking:** interfaces, addresses, routes, and any other configured features.

For example, a bridge-based setup can connect the Pod’s `eth0` through a veth pair to a host bridge.

When that network attachment is removed, the runtime also requests cleanup, including releasing allocated addresses. CNI defines operations such as `ADD` for connecting and `DEL` for cleanup. [Kubernetes](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/?utm_source=chatgpt.com)

**A plugin is a program with a particular networking job.** Several plugins can cooperate.

These are common building-block plugins:

| Plugin | Its job in plain English |
|---|---|
| **`bridge`** | Connects the container to a virtual switch using a veth pair |
| **`ptp`** | Creates a direct connection between the container and host using a veth pair |
| **`host-local`** | Picks an IP from a configured pool and tracks allocated addresses on that host |
| **`static`** | Uses explicitly configured IP addresses |
| **`dhcp`** | Requests an address from a DHCP server |
| **`loopback`** | Sets up the `lo` interface used for local communication |
| **`portmap`** | Maps a host port to a container port, such as for a Pod’s `hostPort` |
| **`bandwidth`** | Limits how much network traffic the connection can carry |

The address-allocation plugins belong to **IPAM: IP Address Management**. Their job is to decide which addresses to allocate and track them. [cni.dev](https://www.cni.dev/plugins/current/?utm_source=chatgpt.com)

For a concrete example, suppose a setup uses **`bridge` + `host-local`**:

1. The runtime creates the Pod’s network namespace.
2. The bridge plugin creates the virtual cable and attaches the Pod to the bridge.
3. The bridge plugin asks `host-local` for an address.
4. `host-local` allocates an address—for example, `10.10.0.2`.
5. The bridge plugin configures the Pod interface with the returned address information.

This gives the two plugins different responsibilities: **one handles the connection; the other allocates the address**. A basic local bridge setup also needs additional arrangements to connect Pods across different nodes. [cni.dev](https://www.cni.dev/plugins/current/main/bridge/?utm_source=chatgpt.com)

**Calico, Flannel, and Cilium are larger networking solutions.** People commonly call them “CNI plugins,” but their complete installations can include plugins, agents, and controllers working together.

| Networking solution | Layman explanation |
|---|---|
| **Calico** | Connects workloads and provides rules controlling which workloads may communicate |
| **Flannel** | Provides Pod connectivity across nodes; policy enforcement uses an additional policy controller |
| **Cilium** | Provides networking and security, with tools such as Hubble for seeing which workloads communicate and where connections fail |
| **Azure CNI** | Provides Pod networking for AKS, with options that use Azure network addresses or a separate overlay address range |

Their features and implementation differ even though they integrate through the CNI interface. [Calico Documentation](https://docs.tigera.io/calico/latest/about/?utm_source=chatgpt.com)

So, when someone says **“our cluster uses Calico as its CNI,”** they mean: **“Our cluster uses Calico’s networking software to provide Pod connectivity and the networking features we’ve configured.”**
