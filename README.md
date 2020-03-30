# Important

## Cert
- The Candidate must contact the Exam Proctoring Partner’s Support team within 15 minutes of the scheduled start time for their reservation to report an issue. Otherwise, they will be markedas a "No Show" for the Exam
- Candidate is not allowed to read the questions out loud, to themselves, during the exam
- Candidate is permitted to drink clear liquids from a label-free clear bottle or a clear glass
- Candidate is not allowed to wear any electronic device in their ears, on their face or on their body
- During the exam, Candidate may only run one application - the Chrome/Chromium browser in which the exam is shown
- Candidate is not allowed to write or enter input on anything (whether paper, electronic device, etc.) outside of the exam console screen

## What can we do
- Root privileges can be obtained by running 'sudo -i'.
- Rebooting of your server IS permitted at any time.
- Ctrl+C & and Ctrl+V are not supported in your exam terminal. For Windows: Ctrl+Insert to copy and Shift+Insert to paste.
- Issues with wrapped text within the terminal pane may be resolved by resizing your browser window temporarily.
- Candidates can confirm the time remaining with the proctor directly.

## Tips
- Skip hard questions and come back to them later (flag it in the notepad)
- Pay attention to the point value of the question. Skip difficult, low-value questions!
- Use the documentation: https://kubernetes.io/docs
- Dont be afraid of using sudo -i for root operations to save time.
- After sshing to a given node (ssh command will be given) don't forget to exit to return to main machine.
- **WARN**: if the exercises says that you need to find the broken `service`, and only the affected k8s object, describe the `service` search for the selector and find the pod with that one, and that is the failing one even though there may be other kubernetes objects .
- **WARN**: if it is asked to delete a resource, delete it with `--force`, as it may be requested later to delete it forcefully.

# Cheatsheet

## Random
- https://en.wikipedia.org/wiki/Kibibyte
    - Name comes from the contraction `kilo binary byte`.
    - 1 kibibyte = 1024 B = 2^10 bytes.
    - 1 kilobyte = 1000 B = 10^3 bytes.
    - kibibyte > kilobyte (in bytes)

## tmux && screen
```markdown
# https://www.linode.com/docs/networking/ssh/persistent-terminal-sessions-with-tmux/
- Prefix: ctrl+b
- New-window: Prefix + c
- Split-horizontal: Prefix + "
- Split-vertical: Prefix + %

- List sessions: tmux ls
- Attach session: tmux attach -t 0
- Destroy all sessions: tmux kill-server
```

```markdown
# Screen cheatsheet: https://gist.github.com/jctosta/af918e1618682638aa82
- Prefix: ctrl-a
- New-window: Prefix + c
- Move between windows: Prefix + n || Prefix + p || Prefix + " || Prefix + <number>
- Kill window: Prefix + k
- Split-horizontal: Prefix + S
- Split-vertical: Prefix + |
- Move between splits: Prefix + tab (You need to create a New-window in a new split, with Prefix + c)
- Detach: Prefix + d

- List sessions: screen -ls
- Attach running session: screen -x
- Attach session with name: screen -r <session_name>
- New session with name: screen -S <session_name>
```

## various tools
```bash
# grep
grep -Hnri # grep: print file name, print line number, recursive, ignore-case

# vim
set shiftwidth=2 softtabstop=2 tabstop=2 
set list

# ubuntu
apt update
apt install net-tools dnsutils

# wget
wget -qO- <link> # quiet, output to - (stdout)
wget --spider <link> # just check the page is there 

# cron syntax: https://en.wikipedia.org/wiki/Cron
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command to execute

# Note: A question mark (?) in the schedule has the same meaning as an asterisk *

# Example: https://crontab.guru/
# */1 * * * * => every minute
# 1   * * * * => at minute 1 (18:01, 19:01...)
```

## k8s  
```bash
alias k=kubectl
alias kn="kubectl config set-context --current --namespace"
source <(kubectl completion bash)
complete -F __start_kubectl k                       

# check if metric-server is installed
kubectl get apiservices | grep metrics

# run including a command!
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'

kubectl run curl --image=radial/busyboxplus:curl -it --rm
kubectl get pod mypod -o yaml --export > mypod.yaml # Export spec without status (WARN: does not include the namespace!)
kubectl api-resources
kubectl api-versions # https://akomljen.com/kubernetes-api-resources-which-group-and-version-to-use/
kubectl autoscale deployment <deploy> --min=2 --max=10
kubectl annotate pods <pod> icon-url=http://goo.gl/XXBTWq
kubectl logs my-pod --previous # dump pod logs (stdout) for a previous instantiation of a container
kubectl delete pods,services -l name=myLabel # Delete pods and services with label name=myLabel
kubectl -n my-ns delete pod,svc --all # Delete all pods and services in namespace my-ns,
kubectl attach my-pod -i # Attach to Running Container
kubectl port-forward my-pod 5000:6000 # Listen on port 5000 on the local machine and forward to port 6000 on my-pod
kubectl top pod POD_NAME --containers # Show metrics for a given pod and its containers
kubectl logs --previous (also works with deploy/job)

kubectl set image deployment/<deploy> <container_name>=<new_image> --record  # Rolling update & Rollbacks (remember to include the flag --record so be able to rollback)
kubectl rollout history [--revision <rev>]
kubectl rollout undo deployment/<deploy> [--to-revision=<rev>]

kubectl describe networkpolicies # very useful to see the ingress/egress rules
```

## yaml
```yaml
# https://stackoverflow.com/questions/3790454/how-do-i-break-a-string-over-multiple-lines

# Usually you want a line continuation, use >:
key: >
  Your long
  string here.

# If you want the linebreaks to be preserved as \n in the string (for instance, embedded markdown with paragraphs), use |.
key: |
  ### Heading

  * Bullet
  * Points

# If you need to split lines in the middle of words or literally type linebreaks as \n, use double quotes instead:
key: "Antidisestab\
 lishmentarianism.\n\nGet on it."

# HINT: Use >- or |- instead if you don't want a linebreak appended at the end.
```

# To improve

- What's the use of giving a Port name in a Pod definition?
  - Each named port in a pod must have a unique name. It can be referred to by services.

- Rolling update strategy in the deployment
    - maxSurge (max of extra replicas that can be created with the new version) && maxUnavailable (max number of replicas that can be considered unavailable)
    - percentage (based on the replicas value) or a fixed number
- Not sure if included but review a bit of:
    - Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
    - Statefulset
    - Daemonset
    - RBAC: https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/
    - Quotas
    - PodDisruptionBudget
    - PriorityClass
    - Taints/Tolerations/Affinity


# Docs to study

Most of the links were taken from:
- https://github.com/dgkanatsios/CKAD-exercises
- https://github.com/twajr/ckad-prep-notes#tasks-from-kubernetes-doc

## P1

https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Network policies do not conflict, they are additive. If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies’ ingress/egress rules. Thus, order of evaluation does not affect the policy result.
- podSelector: Each NetworkPolicy includes a podSelector which selects the grouping of pods to which the policy applies. The example policy selects pods with the label “role=db”. An empty podSelector selects all pods in the namespace.
- policyTypes: Each NetworkPolicy includes a policyTypes list which may include either Ingress, Egress, or both. The policyTypes field indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If no policyTypes are specified on a NetworkPolicy then by default Ingress will always be set and Egress will be set if the NetworkPolicy has any egress rules.
- namespaceSelector and podSelector, be careful to use correct YAML syntax:

```yaml
# contains a single from element allowing connections from Pods with the label role=client in namespaces with the label user=alice. But this policy:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
---
# contains two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client, or from any Pod in any namespace with the label user=alice.
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
```
- Examples:

```yaml
# Default deny all ingress traffic (in a namespace)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---

# Default allow all ingress traffic (in a namespace)
# This works even if policies are added that cause some pods to be treated as “isolated”
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress

# Same cases for egress traffic (or both!)
```

https://kubernetes.io/docs/concepts/services-networking/service/
- Port definitions in `Pods` have names, and you can reference these names in the `targetPort` attribute of a Service. You can change the port numbers that Pods expose in the next version of your backend software, without breaking clients.
- If kube-proxy is running in iptables mode and the first Pod that’s selected does not respond, the connection fails. This is different from userspace mode: in that scenario, kube-proxy would detect that the connection to the first Pod had failed and would automatically retry with a different backend Pod.
- You can use Pod readiness probes to verify that backend Pods are working OK, so that kube-proxy in iptables mode only sees backends that test out as healthy. Doing this means you avoid having traffic sent via kube-proxy to a Pod that’s known to have failed.
- If you want to make sure that connections from a particular client are passed to the same Pod each time, you can select the session affinity based on the client’s IP addresses by setting service.spec.sessionAffinity to “ClientIP” (the default is “None”). You can also set the maximum session sticky time by setting service.spec.sessionAffinityConfig.clientIP.timeoutSeconds appropriately. (the default value is 10800, which works out to be 3 hours).
- For some Services, you need to expose more than one port. Kubernetes lets you configure multiple port definitions on a Service object. When using multiple ports for a Service, you must give all of your ports names so that these are unambiguous.
- You can specify your own cluster IP address as part of a Service creation request. To do this, set the .spec.clusterIP field. For example, if you already have an existing DNS entry that you wish to reuse, or legacy systems that are configured for a specific IP address and difficult to re-configure.
- When you have a Pod that needs to access a Service, and you are using the environment variable method to publish the port and cluster IP to the client Pods, you must create the Service before the client Pods come into existence. Otherwise, those client Pods won’t have their environment variables populated.
- A cluster-aware DNS server, such as CoreDNS, watches the Kubernetes API for new Services and creates a set of DNS records for each one. If DNS has been enabled throughout your cluster then all Pods should automatically be able to resolve Services by their DNS name.
- Kubernetes also supports DNS SRV (Service) records for named ports. If the `my-service.my-ns` Service has a port named `http` with the protocol set to `TCP`, you can do a `DNS SRV` query for `_http._tcp.my-service.my-ns` to discover the port number for http, as well as the IP address.
- Sometimes you don’t need load-balancing and a single Service IP. In this case, you can create what are termed “headless” Services, by explicitly specifying "None" for the cluster IP (.spec.clusterIP).
    - For headless Services that define selectors, the endpoints controller creates Endpoints records in the API, and modifies the DNS configuration to return records (addresses) that point directly to the Pods backing the Service.
    - For headless Services that do not define selectors, the endpoints controller does not create Endpoints records. However, the DNS system looks for and configures either: CNAME records for ExternalName-type Services || A records for any Endpoints that share a name with the Service, for all other types.
- ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.
- In the Service spec, externalIPs can be specified along with any of the ServiceTypes. In the example below, “my-service” can be accessed by clients on “80.11.12.10:80” (externalIP:port)
- Service without selectors, usueful to point to external places or service in a different namespace/cluster. Remember the corresponding Endpoint object is not created automatically.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service # must have the same name than the service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
- You should be able to ssh into any node in your cluster and curl both IPs. Note that the containers are not using port 80 on the node, nor are there any special NAT rules to route traffic to the pod. This means you can run multiple nginx pods on the same node all using the same containerPort and access them from any other pod or node in your cluster using IP. Like Docker, ports can still be published to the host node’s interfaces, but the need for this is radically diminished because of the networking model.
- Note: If the service environment variables are not desired (because possible clashing with expected program ones, too many variables to process, only using DNS, etc) you can disable this mode by setting the `enableServiceLinks` flag to false on the pod spec.

https://kubernetes.io/docs/concepts/storage/volumes/
- A Kubernetes volume, on the other hand, has an explicit lifetime - the same as the Pod that encloses it. Consequently, a volume outlives any Containers that run within the Pod, and data is preserved across Container restarts. Of course, when a Pod ceases to exist, the volume will cease to exist, too.
- Volumes can not mount onto other volumes or have hard links to other volumes. Each Container in the Pod must independently specify where to mount each volume.
- `configMap`
    - A Container using a ConfigMap as a `subPath` volume mount will not receive ConfigMap updates`.
- `secret`
    - A Container using a Secret as a subPath volume mount will not receive Secret updates.
    - Backed by tmpfs (a RAM-backed filesystem) so they are never written to non-volatile storage.
- `emptyDir`
    - By default, emptyDir volumes are stored on whatever medium is backing the node - that might be disk or SSD or network storage, depending on your environment. However, you can set the `emptyDir.medium` field to `Memory` to tell Kubernetes to mount a `tmpfs` (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on node reboot and any files you write **will count against your Container’s memory limit**.
- `gitRepo (deprecated)`
    - As an alternative, mount an `emptyDir` into an `InitContainer` that clones the git repo, then mount the EmptyDir into the Pod’s container.
- `hostPath`
    - Mounts a file or directory from the host node's filesystem into your Pod.
    - Use: running a Container that needs access to Docker internals; use a hostPath of `/var/lib/docker`, allowing a Pod to specify whether a given hostPath should exist prior to the Pod running, whether it should be created, and what it should exist.
    - `hostPah.type`: DirectoryOrCreate (created with 0755, same ownership with Kubelet), Directory, FileOrCreate, File, Socket, CharDevice, BlockDevice, '' (default, no check performed)
    - **WARN**: the files or directories created on the underlying hosts are only writable by `root`. You either need to run your process as root in a privileged Container or modify the file permissions on the host to be able to write to a hostPath volume.
- `nfs`
    - An nfs volume allows an existing NFS (Network File System) share to be mounted into your Pod. Unlike emptyDir, which is erased when a Pod is removed, the contents of an nfs volume are preserved and the volume is merely unmounted. This means that an NFS volume can be pre-populated with data, and that data can be “handed off” between Pods. NFS can be mounted by multiple writers imultaneously.
- `persistentVolumeClaim`
    - A persistentVolumeClaim volume is used to mount a PersistentVolume into a Pod. PersistentVolumes are a way for users to "claim" durable storage (such as a GCE PersistentDisk or an iSCSI volume) without knowing the details of the particular cloud environment.
- `local`
    - A local volume represents a mounted local storage device such as a disk, partition or directory.
    - Local volumes can only be used as a statically created PersistentVolume. Dynamic provisioning is not supported yet.
    - PersistentVolume nodeAffinity is required when using local volumes. It enables the Kubernetes scheduler to correctly schedule Pods using local volumes to the correct node.
    - They are subject to the availability of the underlying node and are not suitable for all applications. If a node becomes unhealthy, then the local volume will also become inaccessible, and a Pod using it will not be able to run. Applications using local volumes must be able to tolerate this reduced availability.
- `projected`
    - Maps several existing volume sources into the same directory.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items: # optional
            - key: username
              path: my-group/my-username
      - secret:
          name: mysecret2
          items: # optional
            - key: password
              path: my-group/my-password
```

- Use  `volumeMounts.subPath` to share one volume for multiple uses in a single pod. It specifies a subpath inside the referenced volume instead of its root.
- You can use `subPath` with expanded environment variables:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef: # Cool trick!
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```

https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- Once bound, PersistentVolumeClaim binds are exclusive, regardless of how they were bound. A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.
- Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.
- If a user deletes a PVC in active use by a Pod, the PVC is not removed immediately. PVC removal is postponed until the PVC is no longer actively used by any Pods. Also, if an admin deletes a PV that is bound to a PVC, the PV is not removed immediately. PV removal is postponed until the PV is no longer bound to a PVC.
- When a user is done with their volume, they can delete the PVC objects from the API that allows reclamation of the resource. The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim. Currently, volumes can either be:
    - `Retained`: allows for manual reclamation of the resource. You need to delete the PV, cleanup the data, create a new PV (or delete the associated storage asset like AWS EBS)
    - `Deleted`: removes both the PV from Kubernetes, as well as the associated storage asset in the external infrastructure, like AWS EBS.
    - `Recycled`: Deprecated, use dynamic provisioning instead. (Basic scrub: rm -rf /thevolume/*). only `NFS` and `HostPath` support recycling.
- PV `HostPath` single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster.
- Currently, `storage size` is the only resource that can be set or requested. Future attributes may include IOPS, throughput, etc.
- A volume with `volumeMode: Filesystem` is mounted into Pods into a directory. If the volume is backed by a block device and the device is empty, Kuberneretes creates a filesystem on the device before mounting it for the first time. You can set the value of `volumeMode: Block` to use a volume as a raw block device. Such volume is presented into a Pod as a block device, without any filesystem on it. This mode is useful to provide a Pod the fastest possible way to access a volume, without any filesystem layer between the Pod and the volume.
- `Access modes`: ReadWriteOnce (RWO), ReadOnlyMany (ROX), ReadWriteMany (RWX)
- A volume can only be mounted using one access mode at a time, even if it supports many. For example, a GCEPersistentDisk can be mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes, but not at the same time.
- A PV can have a `storageClassName` attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class
- A PV can specify `node affinity` to define constraints that limit what nodes this volume can be accessed from. Pods that use a PV will only be scheduled to nodes that are selected by the node affinity.
- `Phases of a Volumes`: Available, Bound, Released, Failed
- PVC can have a `selector` to filter the set of volumes. YOu can use `matchLabels` as well as `matchExpressions`.
- PVC without attribute `storageClassName` will default to the configured value in the admission plugin, or "" if there is no one.
- Claims (in a Pod) must exist in the same namespace as the Pod using the PVC.
- PV binds are exclusive, and since PVC are namespaced objects, mounting claims with "Many" modes (ROX, RWX) is only possible within one namespace.
- Volume Snapshot / Restore and Cloning available to some CSI volume plugins.
- Writing Portable Configuration:
    - Do not include PersistentVolume objects in the config, since the user instantiating the config may not have permission to create PersistentVolumes.
    - In your tooling, watch for PVCs that are not getting bound after some time and surface this to the user, as this may indicate that the cluster has no dynamic storage support (in which case the user should create a matching PV) or the cluster has no storage system (in which case the user cannot deploy config requiring PVCs).
- When a Pod consumes a PV that has a `pv.beta.kubernetes.io/gid: "1234"` annotation, the annotated GID is applied to all containers in the Pod in the same way that GIDs specified in the Pod’s security context are. Every GID, whether it originates from a PV annotation or the Pod’s specification, is applied to the first process run in each container.
- `Dynamic volume provisioning`: https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
    - The implementation of dynamic volume provisioning is based on the API object StorageClass from the API group storage.k8s.io. A cluster administrator can define as many StorageClass objects as needed, each specifying a volume plugin (aka provisioner) that provisions a volume and the set of parameters to pass to that provisioner when provisioning. A cluster administrator can define and expose multiple flavors of storage (from the same or different storage systems) within a cluster, each with a custom set of parameters. 
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 30Gi
```

https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
- Avoid deleting job's pods with: `kubectl delete jobs/old --cascade=false`.
- Job is only appropriate for pods with `RestartPolicy` equal to  `OnFailure` or `Never`. Note: If RestartPolicy is not set, the default value is Always.
- An entire Pod can also fail, for a number of reasons, such as when the pod is kicked off the node (node is upgraded, rebooted, deleted, etc.), or if a container of the Pod fails and the `.spec.template.spec.restartPolicy = "Never"`. When a Pod fails, then the Job controller starts a new Pod. This means that your application needs to handle the case when it is restarted in a new pod. In particular, it needs to handle temporary files, locks, incomplete output and the like caused by previous runs.
  - Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and `.spec.template.spec.restartPolicy = "Never"`, the same program may sometimes be started twice.
- If you do specify `.spec.parallelism` and `.spec.completions` both greater than 1, then there may be multiple pods running at once. Therefore, your pods must also be tolerant of concurrency.
- There are situations where you want to fail a Job after some amount of retries due to a logical error in configuration etc. To do so, set `.spec.backoffLimit` to specify the number of retries before considering a Job as failed (`and any running Pods will be terminated`). The back-off limit is set by default to 6. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s …) capped at six minutes.
- If your job has `restartPolicy = "OnFailure"`, keep in mind that your container running the Job will be terminated once the job backoff limit has been reached. This can make debugging the Job’s executable more difficult. We suggest setting `restartPolicy = "Never"` when debugging the Job.
- Another way to terminate a Job is by setting the `.spec.activeDeadlineSeconds` field to a number of seconds.
- Keep in mind that the `restartPolicy` applies to the Pod, and not to the Job itself: there is no automatic Job restart once the Job status is `type: Failed`. That is, the Job termination mechanisms activated with `.spec.activeDeadlineSeconds` and `.spec.backoffLimit` result in a permanent Job failure that requires manual intervention to resolve.
- The Job object can be used to support reliable parallel execution of Pods. The Job object is not designed to support closely-communicating parallel processes, as commonly found in scientific computing. It does support parallel processing of a set of independent but related work items. These might be emails to be sent, frames to be rendered, files to be transcoded, ranges of keys in a NoSQL database to scan.
- Job Parallel Patterns:
  - `Job Tamplate Expansion`: use `completion: 1` and `parallelims: 1` (default values, can be omitted)
    - Simply have some placeholders in your `Job` and replace them (eg: with sed) and then create them in k8s. This is one `Job object for each work item`.
  - `Queue with Pod Per Work Item`: use `completion: #work-items` and `parallelism: any (usually >=2)`.
    - Each POD should be passed in an item read from a `message queue`, so your app may not need modifications!
    - If the number of completions is set to less than the number of items in the queue, then not all items will be processed.
    - If the number of completions is set to more than the number of items in the queue, then the Job will not appear to be completed, even though all items in the queue have been processed. It will start additional pods which will block waiting for a message.
  - `Queue with Variable Pod Count`: use `completion: 1` (default) and `parallelism: any (usually >=2)`.
    - The app needs to be modified (eg: use a redis client to read message from the queue)
    - Each pod works on several items from the `message queue` and then exits when there are no more items. Since the workers themselves detect when the workqueue is empty, and the Job controller does not know about the workqueue, it relies on the workers to signal when they are done working. The workers signal that the queue is empty by exiting with success. So, as soon as any worker exits with success, the controller knows the work is done, and the Pods will exit soon. That's why we set the `completion: 1`. The job controller will wait for the other pods to complete too.
  - `Single Job with Static Work Assignment`: use `completion: #work-items` and `parallelism: any (usually >=2)`.
  - If you have a continuous stream of background processing work to run, then consider running your background workers with a `ReplicaSet` instead, and consider running a background processing library such as https://github.com/resque/resque.

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

- A `CronJob` creates `Jobs` on a `time-based schedule` with cron format: https://en.wikipedia.org/wiki/Cron.
- Cron jobs have limitations and idiosyncrasies. For example, in certain circumstances, a single cron job can create `multiple jobs`. Therefore, jobs should be idempotent.
- The `CronJob` is only responsible for creating `Jobs` that match its schedule, and the `Job` in turn is responsible for the management of the Pods it represents.
- The `startingDeadlineSeconds` field is optional. It stands for the deadline in seconds for starting the job if it misses its `scheduled time` for any reason. After the deadline, the cron job does not start the job. Jobs that do not meet their deadline in this way count as failed jobs. If this field is not specified, the jobs have no deadline.
- The `CronJob controller` counts how many `missed schedules` happen for a cron job. If there are more than `100 missed schedules`, the cron job is no longer scheduled. When `startingDeadlineSeconds` is not set, the CronJob controller counts missed schedules from `status.lastScheduleTime` until now.
  - For example, one cron job is supposed to run every minute, the `status.lastScheduleTime` of the cronjob is `5:00am`, but now it’s `7:00am`. That means `120` schedules were missed, so the cron job is no longer scheduled.
- If the `startingDeadlineSeconds` field is set (not null), the CronJob controller counts how many missed jobs occurred from the value of `startingDeadlineSeconds` until now.
  - For example, if it is set to 200, it counts how many missed schedules occurred in the last 200 seconds. In that case, if there were more than 100 missed schedules in the last 200 seconds, the cron job is no longer scheduled.
- Deleting the `cronjob` removes all the `jobs` and `pods` it created and stops it from creating additional jobs.
- The `concurrencyPolicy` field is optional. It specifies how to treat concurrent executions of a job that is created by this cron job:
  - `Allow` (default)
  - `Forbid`: if it is time for a new job run and the previous job run hasn’t finished yet, the cron job skips the new job run.
  - `Replace`: if it is time for a new job run and the previous job run hasn’t finished yet, the cron job replaces the currently running job run with a new job run
- The `suspend` field is optional. If it is set to `true`, all subsequent executions are suspended. This setting does not apply to already started executions. Defaults to `false`.
- The `successfulJobsHistoryLimit` (default 3) and `failedJobsHistoryLimit` (default 1)fields are optional. Setting to 0 keeps nothing.

https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
- Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. Containers inside a Pod can communicate with one another using localhost. When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports).

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
- The type field is a string with the following possible values:
  - `PodScheduled`: the Pod has been scheduled to a node;
  - `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services;
  - `Initialized`: all init containers have started successfully;
  - `ContainersReady`: all containers in the Pod are ready.
- A `PodSpec` has a `restartPolicy` field with possible values `Always`, `OnFailure`, and `Never`. The default value is Always. restartPolicy applies to all Containers in the Pod. restartPolicy only refers to restarts of the Containers by the kubelet on the same node. Exited Containers that are restarted by the kubelet are restarted with an `exponential back-off` delay (10s, 20s, 40s …) capped at five minutes, and is reset after ten minutes of successful execution. 
- A pod, once bound to a node, will never be rebound to another node.
- `Pod readiness gate`: Additional conditions to be evaluated for Pod readiness. If Kubernetes cannot find such a condition in the `status.conditions`, the status of the condition is default to `False`:
```yaml
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: "www.example.com/feature-1"   # an extra PodCondition 
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
```

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- `Probe with command`
- `Probe with HTTP`: Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.
- `Probe with TCP`: The kubelet will attempt to open a `socket` to your container on the specified port. If it can establish a connection, the container is considered healthy.
- `Startup Probe`: The kubelet uses `startup probes` to know when a container application has started. If such a probe is configured, it disables `1liveness` and `readiness` checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running. Set up a `startup probe` with the same command, HTTP or TCP check, with a `failureThreshold * periodSeconds` long enough to cover the worse case startup time.
- You can use a named `containerPort` for HTTP or TCP checks `port` field.
- For a `TCP probe`, the kubelet makes the probe connection at the node, not in the pod, which means that you can not use a service name in the `host` parameter since the kubelet is unable to resolve it.
- Probes fields: `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds` (after which the probe times out), `successThreshold`, `failureThreshold`.
- Additional fields for HTTP probes: `host` (use 127.0.0.1 if container listens here and Pod has `hostNetwork: true`), `scheme` (http/https, no certificate check), `path`, `httpHeaders`, `port`.

https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/
- Fractional values are allowed. A Container that requests 0.5 CPU is guaranteed half as much CPU as a Container that requests 1 CPU. You can use the suffix m to mean milli. For example 100m CPU, 100 milliCPU, and 0.1 CPU are all the same. **Precision finer than 1m is not allowed**.
- If you do not specify a CPU limit for a Container, the Container has no upper bound on the CPU resources it can use; or if its running in a `namespace` that has a default CPU limit, the Container is automatically assigned the default limit. Cluster administrators can use a `LimitRange` to specify a default value for the CPU limit.

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/
- The memory resource is measured in bytes. You can express memory as a plain integer or a fixed-point integer with one of these suffixes: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.
- If you do not specify a memory limit for a Container, the Container has no upper bound on the amount of memory it uses. The Container could use all of the memory available on the Node where it is running which in turn could invoke the `OOM Killer`. Further, in case of an OOM Kill, a container with no resource limits will have a greater chance of being killed; or the Container is running in a namespace that has a default memory limit, and the Container is automatically assigned the default limit. Cluster administrators can use a `LimitRange` to specify a default value for the memory limit.
- Besides describing or seeing the status of a Pod, you can also run `kubectl describe nodes` and search for `OOMKilling`.

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/

https://kubernetes.io/docs/concepts/workloads/controllers/deployment

https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/

https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/
- The container process no longer has `PID 1`. Some container images refuse to start without PID 1 (eg: systemd). `kill -HUP 1` will signal the pod sandbox, `/pause` in the example below.
- Processes/Filesystems are visible to other containers in the pod. This includes all information visible in `/proc`, `/proc/$pid/root`, such as passwords that were passed as arguments or environment variables. These are protected only by regular Unix permissions.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true # This is the required field
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    securityContext:
      capabilities:
        add:
        - SYS_PTRACE # needed if you need to send signal (SIGHUP) to a process in other containers
    stdin: true
    tty: true
```

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/

https://kubernetes.io/docs/tasks/debug-application-cluster/local-debugging/

https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/

https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

## P2

https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
- If the pod does not have a ServiceAccount set, it sets the ServiceAccount to default.
- It adds a volumeSource to each container of the pod mounted at /var/run/secrets/kubernetes.io/serviceaccount.

https://kubernetes.io/docs/concepts/cluster-administration/logging/
- Using a node-level logging agent is the most common and encouraged approach for a Kubernetes cluster, because it creates only one agent per node, and it doesn’t require any changes to the applications running on the node. However, node-level logging only works for applications’ standard output and standard error.
- By having your sidecar containers stream to their own stdout and stderr streams, you can take advantage of the kubelet and the logging agent that already run on each node. The sidecar containers read logs from a file, a socket, or the journald. Each individual sidecar container prints log to its own stdout or stderr stream.
- Note: Using a logging agent in a sidecar container can lead to significant resource consumption. Moreover, you won’t be able to access those logs using kubectl logs command, because they are not controlled by the kubelet.

https://kubernetes.io/docs/concepts/configuration/secret/
```yaml
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 0777 # 511 in decimal in case you use json
```
- Linux users should add the option -w 0 to base64 commands.
- username secret is stored under /etc/foo/my-group/my-username file instead of /etc/foo/username. 
- If .spec.volumes[].secret.items is used, only keys specified in items are projected. To consume all keys from the secret, all of them must be listed in the items field. All listed keys must exist in the corresponding secret. Otherwise, the volume is not created.
- Inside the container that mounts a secret volume, the secret keys appear as files and the secret values are base64 decoded.
- When a secret currently consumed in a volume is updated, projected keys are eventually updated as well. The kubelet checks whether the mounted secret is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the Secret. The type of the cache is configurable using the ConfigMapAndSecretChangeDetectionStrategy field in the KubeletConfiguration struct. A Secret can be either propagated by watch (default), ttl-based, or simply redirecting all requests directly to the API server. As a result, the total delay from the moment when the Secret is updated to the moment when new keys are projected to the Pod can be as long as the kubelet sync period + cache propagation delay, where the cache propagation delay depends on the chosen cache type (it equals to watch propagation delay, ttl of cache, or zero correspondingly).
- Secret resources reside in a namespace. Secrets can only be referenced by Pods in that same namespace.
- How to use a `docker` secret type?
    - You can attach it in a POD, using the field `imagePullSecrets` or better yet, attaching it the default service-account in the desired namespace:
    - https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account

https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
- The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (_), dots (.), and alphanumerics between. The prefix is optional. If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots (.), not longer than 253 characters in total, followed by a slash (/).
- The kubernetes.io/ and k8s.io/ prefixes are reserved for Kubernetes core components.
- Valid label values must be 63 characters or less and must be empty or begin and end with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (_), dots (.), and alphanumerics between.
- The set of pods that a `service` targets is defined with a label selector, and only equality-based requirement selectors are supported (no set-based support)
```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

## P3

https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).
- It is not necessary to use multiple namespaces just to separate slightly different resources, such as different versions of the same software: use labels to distinguish them.
- When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`

https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
