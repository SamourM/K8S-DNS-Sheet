# K8S-DNS-Sheet-num1


1- By default most Kubernetes clusters automatically configure an internal DNS service to provide a lightweight mechanism for service discovery.

2- Before Kubernetes version 1.11, the Kubernetes DNS service was based on kube-dns.

3- Version 1.11 introduced CoreDNS to address some security and stability concerns with kube-dns.

4- The kube-dns service listens for service and endpoint events from the Kubernetes API and updates its DNS records as needed. These events are triggered when you create, update or delete Kubernetes services and their associated pods.

5- kubelet sets each new pod's /etc/resolv.conf nameserver option to the cluster IP of the kube-dns service, with appropriate search options to allow for shorter hostnames to be used:

resolv.conf
nameserver 10.32.0.10

6- Applications running in containers can then resolve hostnames such as example-service.namespace into the correct cluster IP addresses.

7- full DNS A record for a K8S service: service.namespace.svc.cluster.local.

8- pod A record: 10.32.0.125.namespace.pod.cluster.local.

9- SRV record: _port-name._protocol.service.namespace.svc.cluster.local.

10- If you're addressing a service in the same namespace, you can use just the service name to contact it,thanks to resolve.conf, if in another namespace: other-service.other-namespace, if targeting pod: pod-ip.other-namespace.pod.

11- Kube-dns is made up of three containers running in a kube-dns pod in the kube-system namespace: kube-dns (runs SkyDNS), dnsmasq (DNS resolver and cacher) and sidecar (healthchecks & reporting).

12- Security vulnerabilities in Dnsmasq, and scaling performance issues with SkyDNS led to the creation of a replacement system, CoreDNS.


13- CoreDNS is a single process, written in Go, that covers all of the functionality of the previous system. A single container resolves and caches DNS queries, responds to health checks, and provides metrics.

14- Additional Configuration Options: Kubernetes operators often want to customize how their pods and containers resolve certain custom domains, or need to adjust the upstream nameservers or search domain suffixes configured in resolv.conf. You can do this with the dnsConfig option of your pod's spec.


apiVersion: v1
kind: Pod
metadata:
  namespace: example
  name: custom-dns
spec:
  containers:
    - name: example
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 203.0.113.44
    searches:
      - custom.dns.local
      
**Updating this config will rewrite a pod's resolv.conf to enable the changes. The configuration maps directly to the standard resolv.conf options, so the above config would create a file with nameserver 203.0.113.44 and search custom.dns.local lines**








