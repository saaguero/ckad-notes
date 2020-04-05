# Important

## Cert
- The Candidate must contact the Exam Proctoring Partner’s Support team within 15 minutes of the scheduled start time for their reservation to report an issue. Otherwise, they will be markedas a "No Show" for the Exam
- Candidate is not allowed to read the questions out loud, to themselves, during the exam
- Candidate is permitted to drink clear liquids from a label-free clear bottle or a clear glass
- Candidate is not allowed to wear any electronic device in their ears, on their face or on their body
- During the exam, Candidate may only run one application - the Chrome/Chromium browser in which the exam is shown
- Candidate is not allowed to write or enter input on anything (whether paper, electronic device, etc.) outside of the exam console screen.

## What can we do
- Root privileges can be obtained by running `sudo -i`.
- Rebooting of your server is permitted at any time.
- `Ctrl+C` & and `Ctrl+V` are not supported in your exam terminal.
  - For Linux: select text for copy and `middle button` for paste (or both `left` and `right` simultaneously if you have no middle button.)
  - For Mac: `⌘+C` to copy and `⌘+V` to paste.
  - For Windows: `Ctrl+Insert` to copy and `Shift+Insert` to paste.
- Issues with wrapped text within the terminal pane may be resolved by resizing your browser window temporarily.
- Candidates can confirm the time remaining with the proctor directly.

## Tips
- For getting the feeling about the browser terminal: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-interactive/
- Skip hard questions and come back to them later (flag it in the provided notepad)
- Pay attention to the point value of the question. Skip difficult, low-value questions!
- Use the documentation: https://kubernetes.io/docs
- Dont be afraid of using sudo -i for root operations to save time.
- After sshing to a given node (ssh command will be given) don't forget to exit to return to main machine.
- **WARN**: if the exercises says that you need to find the broken `service`, and only the affected k8s object, describe the `service` search for the selector and find the pod with that one, and that is the failing one even though there may be other kubernetes objects .
- **WARN**: if it is asked to delete a resource, delete it **without** `--force`, as it may be requested later to delete it forcefully.
- **WARN**: if a livenessProbe is requested through something like `Implement a liveness-probe which checks the container to be reachable on port 80` it should be implemented as a `tcpSocket` and not an `httpGet`.

# Cheatsheet

## Random
- https://en.wikipedia.org/wiki/Kibibyte
    - Name comes from the contraction `kilo binary byte`.
    - 1 kibibyte = 1024 B = 2^10 bytes.
    - 1 kilobyte = 1000 B = 10^3 bytes.
    - kibibyte > kilobyte (in bytes)

## tmux && screen
- Prefer tmux (even if you have to install it). If not possible, fallback to screen.

```markdown
# Tmux: https://www.linode.com/docs/networking/ssh/persistent-terminal-sessions-with-tmux/
- Prefix: ctrl+b
- New-window: Prefix + c
- Split-horizontal: Prefix + "
- Split-vertical: Prefix + %

- List sessions: tmux ls
- Attach session: tmux attach -t 0
- Destroy all sessions: tmux kill-server

## Inside tmux
- set mouse # for copy/paste with `Prefix + [` and `Prefix + ]`
- set synchronize-panes
```

```markdown
# Screen: https://gist.github.com/jctosta/af918e1618682638aa82
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
set tabstop=2
set expandtab
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
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c 'echo $(date); sleep 3600'


kubectl run curl --image=radial/busyboxplus -it --rm -- curl -m 5 my-service:8080 # useful to include the 5 second timeout
kubectl run wget --image=busybox -it --rm -- wget --timeout=5 -O- my-service:8080 
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

# Docs to study

Most of the links were taken from:
- https://github.com/dgkanatsios/CKAD-exercises
- https://github.com/twajr/ckad-prep-notes#tasks-from-kubernetes-doc

## P1

https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- Must read, come here many times until you grasp every tip.

https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Network policies do not conflict, they are `additive`. If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies’ ingress/egress rules. Thus, order of evaluation does not affect the policy result.
- `podSelector`: Each NetworkPolicy includes a podSelector which selects the grouping of pods to which the policy applies. The example policy selects pods with the label “role=db”. An empty podSelector selects all pods in the namespace.
- `policyTypes`: Each NetworkPolicy includes a policyTypes list which may include either Ingress, Egress, or both. The policyTypes field indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If no policyTypes are specified on a NetworkPolicy then by default Ingress will always be set and Egress will be set if the NetworkPolicy has any egress rules.
- `namespaceSelector` and `podSelector`, be careful to use correct YAML syntax:

```yaml
# contains a single from element allowing connections from Pods with the label role=client IN namespaces with the label user=alice.
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
---
# contains two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client OR from any Pod in any namespace with the label user=alice.
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
# This works even if policies are added that cause some pods to be treated as "isolated"
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

https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

https://kubernetes.io/docs/concepts/services-networking/service/
- Port definitions in `Pods` have names, and you can reference these names in the `targetPort` attribute of a Service. You can change the port numbers that Pods expose in the next version of your backend software, without breaking clients.
- If `kube-proxy` is running in `iptables mode` and the first Pod that’s selected does not respond, the connection fails. This is different from `userspace mode`: in that scenario, kube-proxy would detect that the connection to the first Pod had failed and would automatically retry with a different backend Pod.
- You can use Pod readiness probes to verify that backend Pods are working OK, so that kube-proxy in iptables mode only sees backends that test out as healthy. Doing this means you avoid having traffic sent via kube-proxy to a Pod that’s known to have failed.
- If you want to make sure that connections from a particular client are passed to the same Pod each time, you can select the session affinity based on the client’s IP addresses by setting `service.spec.sessionAffinity` to `ClientIP` (default is None). You can also set the maximum session sticky time by setting `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` (the default value is 10800, which works out to be 3 hours).
- For some Services, you need to expose more than one port. Kubernetes lets you configure multiple port definitions on a Service object. **When using multiple ports for a Service, you must give all of your ports names so that these are unambiguous**.
- You can specify your own cluster IP address as part of a Service creation request. To do this, set the `.spec.clusterIP` field. For example, if you already have an existing DNS entry that you wish to reuse, or legacy systems that are configured for a specific IP address and difficult to re-configure.
- When you have a Pod that needs to access a Service, and you are using the `environment variable` method to publish the port and cluster IP to the client Pods, you must create the Service before the client Pods come into existence.
  - If the service's environment variables are not desired (because possible clashing with expected program ones, too many variables to process, only using DNS, etc) you can disable this mode by setting the `enableServiceLinks` flag to false on the pod spec.
- A cluster-aware DNS server, such as `CoreDNS`, watches the Kubernetes API for new Services and creates a set of DNS records for each one. If DNS has been enabled throughout your cluster then all Pods should automatically be able to resolve Services by their DNS name.
- Kubernetes also supports `DNS SRV` (Service) records for `named ports`. If the `my-service.my-ns` Service has a port named `http` with the protocol set to `TCP`, you can do a `DNS SRV` query for `_http._tcp.my-service.my-ns` to discover the port number for http, as well as the IP address.
- Sometimes you don’t need load-balancing and a single Service IP. In this case, you can create what are termed `headless services`, by explicitly specifying `None` for the `.specclusterIP`.
    - For `headless services` that define selectors, the endpoints controller creates `Endpoints` records in the API, and modifies the DNS configuration to return records (addresses) that point directly to the Pods backing the Service.
    - For `headless services` that do not define selectors, the endpoints controller does not create `Endpoints` records. However, the DNS system looks for and configures either: CNAME records for `ExternalName services` OR `A records` for any `Endpoints` that share a name with the Service.
- `ExternalName`: Maps the Service to the contents of the `externalName` field (e.g. foo.bar.example.com), by returning a `CNAME record` with its value. **No proxying of any kind is set up**.
- In the service spec, `externalIPs` can be specified along with any of the service types, then a `my-service` will be accessible by clients on `externalIP:port`.
- Service without selectors are usueful to point to external places or service in a different namespace/cluster. Remember the corresponding Endpoint object is not created automatically.
- The set of pods that a `service` targets is defined with a `labelSelector`, and **only equality-based** requirement selectors are supported (no set-based support.)
- You can run multiple `nginx pods` on the `same node` all using the `same containerPort` and access them `from any other pod or node` in your cluster using IP. Like Docker, ports can still be published to the host node's interfaces, but the need for this is radically diminished because of the networking model.

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

https://kubernetes.io/docs/concepts/storage/volumes/
- A `volume` has an explicit `lifetime` - the same as the Pod that encloses it. Consequently, a volume outlives any containers that run within the Pod, and data is preserved across container restarts. Of course, when a Pod ceases to exist, the volume will cease to exist, too.
- Volumes can not mount onto other volumes or have hard links to other volumes. Each Container in the Pod must independently specify where to mount each volume.
- `configMap`
    - A Container using a ConfigMap as a `subPath` volume mount will not receive ConfigMap updates.
- `secret`
    - A Container using a Secret as a subPath volume mount will not receive Secret updates.
    - Backed by `tmpfs` (a RAM-backed filesystem) so they are never written to non-volatile storage.
- `emptyDir`
    - By default, emptyDir volumes are stored on whatever medium is backing the node - that might be disk or SSD or network storage, depending on your environment. However, you can set the `emptyDir.medium` field to `memory` to tell Kubernetes to mount a `tmpfs` (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that unlike disks, it is cleared on node reboot and any files you write **will count against your container’s memory limit**.
- `gitRepo (deprecated)`
    - As an alternative, mount an `emptyDir` into an `InitContainer` that clones the git repo, then mount the emptyDir into the Pod's container.
- `hostPath`
    - Mounts a file or directory from the host node's filesystem into your Pod.
    - Use: running a Container that needs access to Docker internals; use a hostPath of `/var/lib/docker`, allowing a Pod to specify whether a given hostPath should exist prior to the Pod running, whether it should be created, and what it should exist.
    - `hostPah.type`: DirectoryOrCreate (created with 0755, same ownership with Kubelet), Directory, FileOrCreate, File, Socket, CharDevice, BlockDevice, '' (default, no check performed)
    - **WARN**: the files or directories created on the underlying hosts are only writable by `root`. You either need to run your process as root in a privileged Container or modify the file permissions on the host to be able to write to a hostPath volume.
- `nfs`
    - An nfs volume allows an existing NFS (Network File System) share to be mounted into your Pod. Unlike emptyDir, which is erased when a Pod is removed, the contents of an nfs volume are preserved and the volume is merely unmounted. This means that an NFS volume can be pre-populated with data, and that data can be handed off between Pods. NFS can be mounted by multiple writers imultaneously.
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
        fieldRef: # Cool trick to reference data from this k8s object
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
- What's the use of giving a port name in a Pod definition?
  - Each named port in a pod must have a unique name. It can be referred to by services.
  - `pod.spec.containers.ports`: List of ports to expose from the container. Exposing a port here gives the system additional information about the network connections a container uses, but is `primarily informational`. Not specifying a port here `DOES NOT prevent that port from being exposed`. Any port which is listening on the default "0.0.0.0" address inside a container will be ccessible from the network.

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
- Pod phases:
  - `Pending`: includes time before being scheduled as well as time spent downloading images over the network. 
  - `Running`: Pod bound to a node, and containers were created. 
  - `Succeeded`: All containers have have terminated in success, and will not be restarted.
  - `Failed`: All containers have terminated, and at least one of them in failure.
  - `Unknown`: state of the Pod could not be obtained, typically due to an error communicating with the host of the Pod.
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

https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
- The command and arguments that you define in the configuration file override the default command and arguments provided by the container image. If you define args, but do not define a command, the default command is used with your new arguments. If you supply a command but no args for a Container, only the supplied command is used.
- The environment variable appears in parentheses, `$(VAR)`. This is required for the variable to be expanded in the command or args field.

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
- When you are creating a ConfigMap based on a file, the key in the <data-source> defaults to the filename, and the value defaults to the file content.
- The total delay from the moment when the ConfigMap is updated to the moment when new keys are projected to the pod can be as long as kubelet sync period (1 minute by default) + ttl of ConfigMaps cache (1 minute by default) in kubelet. `You can trigger an immediate refresh by updating one of the pod’s annotations`.
- If you use `envFrom` to define environment variables from ConfigMaps, keys that are considered invalid will be skipped. The pod will be allowed to start, but the invalid names will be recorded in the event log `InvalidVariableNames`.
- Use the option `--from-env-file` to create a ConfigMap from an env-file:

```yaml
# env-file.properties
enemies=aliens
lives=3
allowed="true"

# Create the configmap and then check it
# kubectl create configmap config-env-file --from-env-file=env-file.properties
# kubectl get configmap game-config-env-file -o yaml
...
data:
  allowed: '"true"' # quotation marks are preserved
  enemies: aliens
  lives: "3"
...
```

- How to use configmaps in pods:

```yaml
containers:
- name: test-container
  image: busybox
  command: [ "/bin/sh", "-c", "env" ]
  # command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) ] # You can use ConfigMap-defined env in the command section
  env:
  - name: SPECIAL_LEVEL_KEY # Define the environment variable
    valueFrom:
      configMapKeyRef:
        name: special-config # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
        key: special.how # Specify the key associated with the value
---
  envFrom: # Configure all key-value pairs in a ConfigMap as container environment variables
  - configMapRef:
    name: special-config
---
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config # WARN: If there are some files in this directory, they will be deleted.
    # Files will be mounted as /etc/config/<config_key_1>.../etc/config/<config_key_N>
volumes:
- name: config-volume
  configMap:
    name: special-config # Provide the name of the ConfigMap containing the files you want
---
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
    # MY_KEY_1 will be mounted as /etc/config/key_1
volumes:
- name: config-volume
  configMap:
    name: special-config
    items:
    - key: MY_KEY_1
      path: key_1
```

https://kubernetes.io/docs/concepts/configuration/secret/
```yaml
containers:
- name: test
  image: busybox
  volumeMounts:
  - name: secret-volume
    readOnly: true # useful attribute
    mountPath: "/etc/secret-volume"
    # username will be mounted as /etc/secret-volume/my-group/my-username 
volumes:
- name: foo
  secret:
    secretName: mysecret
    items:
    - key: username
      path: my-group/my-username
      mode: 0777 # 511 in decimal in case you use json
```
- Linux users should use the base64 command as `base64 -w 0`.
- If `.spec.volumes[].secret.items` is used, only keys specified in items are projected. To consume all keys from the secret, all of them must be listed. All listed keys must exist in the corresponding secret. Otherwise, the volume is not created.
- Inside the container that mounts a secret volume, the secret keys appear as files and the secret values are `base64 decoded`.
- When a secret currently consumed in a volume is updated, projected keys are eventually updated as well. The kubelet checks whether the mounted secret is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the Secret. The type of the cache is configurable using the `ConfigMapAndSecretChangeDetectionStrategy` field in the `KubeletConfiguration` struct. A Secret can be either propagated by watch (default), ttl-based, or simply redirecting all requests directly to the API server. As a result, the total delay from the moment when the Secret is updated to the moment when new keys are projected to the Pod can be as long as the `kubelet sync period + cache propagation delay`, where the cache propagation delay depends on the chosen cache type (it equals to watch propagation delay, ttl of cache, or zero correspondingly).
- Secret resources reside in a namespace. `Secrets can only be referenced by Pods in that same namespace`.

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- When you create a pod, if you do not specify a service account, it is `automatically` assigned the `default` service account in the same namespace.
- `automountServiceAccountToken: false` can be set both in the `ServiceAccount` as well as in the `Pod.spec` (this takes precedence).
- How to use a `docker-registry` secret type?
  - `kubectl create secret docker-registry docker-secret --docker-server --docker-username=user --docker-password`
  - Then attach it with `imagePullSecrets` in a `Pod.spec` or or better yet in a `ServiceAccount` so that it is included as default in your pods.
- `Service Account Token Volume Projection` example:
```yaml
kind: Pod
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: vault-token
  serviceAccountName: my-sa
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          path: vault-token
          expirationSeconds: 7200
          audience: vault
```

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
- Best way to learn about fields is to run: `kubectl explain` with `pod.spec.securityContext` and `pod.spec.containers.securityContext`.
- Remember that `pod.spec.containers.securityContext` has precedence over `pod.spec.securityContext`.
- Important ones in `pod.spec.securityContext`:
  - `fsGroup`: A special supplemental group that applies to all containers in a pod. Some `volumes` allow the Kubelet to change the ownership of that volume to be owned by the pod: 1. The owning GID will be the FSGroup 2. The setgid bit is set (new files created in the volume will be owned by FSGroup) 3. The permission bits are OR'd with rw-rw---- If unset, the Kubelet will not modify the ownership and permissions of any volume.
  - `runAsGroup`: The GID to run the entrypoint of the container process. Uses runtime default if unset, usualy root (0).
  - `runAsNonRoot`: Indicates that the container must run as a non-root user. If `true`, the Kubelet will validate the image at runtime to ensure that it does not run as UID 0 (root) and fail to start the container if it does.
  - `seLinuxOptions`: The SELinux context to be applied to all containers.
  - `supplementalGroups`: A list of groups applied to the first process run in each container, in addition to the container's primary GID.
- Important ones in `pod.spec.containres.securityContext`:
  - `allowPrivilegeEscalation`: Controls whether a process can gain more privileges than its parent process. `True` always when the container is: 1) run as Privileged OR 2) has `CAP_SYS_ADMIN`.
  - `capabilities`: The capabilities to add/drop when running containers, eg: `capabilities.add: ["NET_ADMIN", "SYS_TIME"]`.
  - privileged: Processes in privileged containers are equivalent to root on the host. Defaults to false.
  - `readOnlyRootFilesystem`: Whether this container has a read-only root filesystem. Default is false.
  - `runAsGroup`, `runAsNonRoot`, `runAsUser`, `seLinuxOptions`.

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

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  initContainers: # These containers are run to completion during pod initialization before others are run.
  - name: install
    image: busybox
    command: [wget, "-O", "/work-dir/index.html", "http://kubernetes.io"]
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

https://kubernetes.io/docs/concepts/workloads/controllers/deployment
- To update a deployment: `kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record` or just edit it `kubectl edit deployment <deploy>`.
- Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% `deploy.spec.strategy.rollingUpdate.maxUnavailable`).
- Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% `deploy.spec.strategy.rollingUpdate.maxSurge`).
- `Multiple updates in-flight (Rollover)`: suppose you create a Deployment to create 5 replicas of `nginx:1.14.2`, but then update the Deployment to create 5 replicas of `nginx:1.16.1`, when only 3 replicas of `nginx:1.14.2` had been created. In that case, the Deployment immediately starts killing the 3 `nginx:1.14.2` Pods that it had created, and starts creating `nginx:1.16.1` Pods. **It does not wait for the 5 replicas of `nginx:1.14.2` to be created before changing course.**
- A `Deployment's revision` is created when a Deployment's rollout is triggered. This means that the new revision is created if and only if the Deployment's Pod template `.spec.template` is changed, for example if you update the labels or container images of the template. Other updates, such as scaling the Deployment, do not create a Deployment revision, so that you can facilitate simultaneous manual- or auto-scaling. This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.
- Rolling back a deployment: `kubectl rollout history deployment.v1.apps/nginx-deployment [--revision=<#rev>]` and rollback with `kubectl rollout undo deployment.v1.apps/nginx-deployment [--to-revision=<#rev>]`
- Scaling a deployment: `kubectl scale deployment.v1.apps/nginx-deployment --replicas=10`
- Scaling with HPA (Horizontal-Pod-Autoscaling): `kubectl autoscale deployment <deploy> --min=10 --max=15 --cpu-percent=80`
- You can pause a Deployment before triggering one or more updates and then resume it. This allows you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.
  - `kubectl rollout pause|resume deployment <deploy>`
  - You can set `.spec.revisionHistoryLimit` field in a Deployment to specify how many old ReplicaSets for this Deployment you want to retain. The rest will be garbage-collected in the background. By default, it is 10.
  - If you want to roll out releases to a subset of users or servers using the Deployment, you can create multiple Deployments, one for each release, following the `canary pattern`.
  - Rolling update strategy: `.spec.strategy.type` can be `RollingUpdate` (default) or `Recreate`.
  - Only a `.spec.template.spec.restartPolicy = Always` is allowed, which is the default if not specified.
  - `.spec.progressDeadlineSeconds` seconds the Deployment controller waits before indicating (in the Deployment status) that the Deployment progress has stalled. In the future, once automatic rollback will be implemented, the Deployment controller will roll back a Deployment as soon as it observes such a condition.

https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/
- Directly accessing the REST API:
  - Use `kubectl proxy --port=8080`, then do `curl http://localhost:8080/api`
  - This is recommended instead of doing it manually becaues it verifies identity of apiserver using self-signed cert, so no MITM attacks are possible. It will authenticate to `apiserver` and in the future it may do intelligent client-side local-balancing and failover.
- Accesing the API from a Pod:
  - `kubernetes.default.svc` resolves to a `Service IP` which in turn will be routed to an apiserver.
  - Run `kubectl proxy` in a sidecar containers in the pod or as a background process within the container.
  - Or use a k8s client library like Go, they will handle locating and authenticating to the apiserver.
  - Either case will use the credentials of the pod `/var/run/secrets/kubernetes.io/serviceaccount/*`.
- Accessing services running on the cluster:
  - Access services, nodes, or pods using the `Proxy Verb`: apiserver does authentication and authorization prior to accessing the remote  service. Use this if the services are not secure enough to expose to the internet, or to gain access to ports on the node IP, or for debugging. Note that proxies may cause problems for some web applications, and only works for HTTP/HTTPS.
  - `kubectl cluster-info` to get the list of services using the `Proxy Verb`.
  - You can also have a `kubectl proxy --port=8080` and access a service passing in a the corresponding url like `http://localhost:8080/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/`
  - Syntax: `http://kubernetes_master_address/api/v1/namespaces/namespace_name/services/[https:]service_name[:port_name]/proxy`

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/
- When you bind a pod to a `hostPort` there are a limited number of places that the pod can be scheduled. In most cases, hostPort is unnecessary; try using a `service` object to expose your pod. If you do require hostPort then you can only schedule as many pods as there are nodes in your container cluster.
- To list all events you can use `kubectl get events` but you have to remember that events are `namespaced`. This means that if you’re interested in events for some namespaced object (e.g. what happened with Pods in namespace my-namespace) you need to explicitly provide a namespace to the command. To see events from all namespaces, you can use the `--all-namespaces` argument.
- Rememeber to inspect `nodes` as the may become `NotReady`, and also notice that the pods are no longer running (they are evicted after five minutes of `NotReady` status).

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/
- **Highly recommended to read everything from this link.**
- Run `kubectl apply --validate -f mypod.yaml`. It will error if for instance you misspelled `command as commnd`.
- If you are missing `endpoints` for your `service`, try listing pods using the labels that Service uses:
```yaml
spec:
  - selector:
     name: nginx
     type: frontend
---
# kubectl get pods --selector=name=nginx,type=frontend
```
- If the list of pods matches expectations, but your endpoints are still empty, it’s possible that you don’t have the right ports exposed. Verify that the `Pod’s containerPort` matches up with the `Service’s targetPort`.

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/
- Same tips than for debugging Pods.

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
- **Highly recommended to read everything from this link.**
- Here are key points (most of them have to be run in a temp pod like a `busybox`)

1. Test connection directly from Pod IP
1. Check the service work by DNS, `nslookup <service>` or `nslookup <service>.<namespace>` in case you are in a different namespace than the service or with FQDN `nslookup <service>.<namespace>.svc.cluster.local` (WARN: cluster.local could be different in your cluster).
    1. If you can do a FQDN lookup but not a relative one, check the file `/etc/resolv.conf` which should have the `"nameserver: 10.0.0.10"`, `"search: default.svc.cluster.local, svc.cluster.local, cluster.local"` and `"options ndots:5"`.
    1. `<nameserver>` is the cluster's DNS Service IP, find it with `kubectl get svc -A | grep kube-dns`.
    1. You can test from a Node as well with `nslookup <FQDN> <nameserver_IP>`. Also, you can `curl <service>`.
1. The `nslookup kubernetes.default` (k8s master service) should always work from within a Pod, if not there might be something with `kube-proxy`.
1. If DNS worked, you should know check if you can connect using the `Service IP`.
1. Whatch out for Service definition:
    1. If you meant to use a numeric port, is it a number 9376 or a string “9376” (named port)?
    1. Is the port's protocol correct for your Pods?
    1. Use service selectors to find the associated pods and check that the correct `endpoints` are created with same pods IP.
1. If you get here, your Service is running, has Endpoints, and your Pods are actually serving. At this point, the whole `Service proxy mechanism is suspect`:
    1. In your nodes: `ps auxw | grep kube-proxy` and check logs with `/var/log/kube-proxy.log` or using `journalctl`.
    1. Ensure `conntrack` is installed.
    1. If `kube-proxy` is in `iptables` mode, check them with `iptables-save | grep hostnames`, for each port of each Service, there should be 1 rule in `KUBE-SERVICES`.


## P2

https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
- If the pod does not have a `ServiceAccount` set, it sets it to `default`.
- It adds a `volumeSource` to each container of the pod mounted at `/var/run/secrets/kubernetes.io/serviceaccount`.

https://kubernetes.io/docs/concepts/cluster-administration/logging/
- Using a node-level logging agent is the most common and encouraged approach for a Kubernetes cluster, because it creates only one agent per node, and it doesn’t require any changes to the applications running on the node. However, node-level logging only works for applications' standard output and standard error.
- By having your `sidecar containers` stream to their own stdout and stderr streams, you can take advantage of the kubelet and the logging agent that already run on each node. The sidecar containers read logs from a file, a socket, or the journald. Each individual sidecar container prints log to its own stdout or stderr stream.
- Note: Using a logging agent in a sidecar container can lead to significant resource consumption. Moreover, you won't be able to access those logs using kubectl logs command, because they are not controlled by the kubelet.

https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
- Labels can be attached to objects at creation time or later on. They can be modified at any time.
- The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (_), dots (.), and alphanumerics between. The prefix is optional. If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots (.), not longer than 253 characters in total, followed by a slash (/).
- The `kubernetes.io/` and `k8s.io/` prefixes are reserved for Kubernetes core components.
```yaml
selector:
  matchLabels: # equality-based selectors
    component: redis
  matchExpressions: # set-based selectors
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
- `kubectl describe pod <pod>  | grep -i qos`
- Types:
  - `Guaranteed`: Containers in a Pod has `memory_request = memory_limit && cpu_request == cpu__limit`
  - `Burstable`: At least one container in a Pod has a memory or cpu request without meeting Guaranteed QoS.
  - `BestEffort`: Containers in a Pod must not have any memory or cpu limits or requests.

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/
- `Sidecar`: Extend and enhance the main container. Example: A Pod with a WebServer container and a Sync container which syncs files from a git repository. File system is shared between the two containers.
- `Ambassador`: Proxy local connections to the world. Example: A Pod with your App container and a Redis Proxy container, which is responsible for splitting reads/writes to appropriate servers. Main container discovers the Redis Proxy container in localhost since they are in the same Pod.
- `Adapter`: Standardize and normalize output. Example: Pods have their App container and a Adapter container to unified logs to be consumed by a Centralized monitoring system.

https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/
- Better to see the help, `kubectl port-forward --help`

https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).
- It is not necessary to use multiple namespaces just to separate slightly different resources, such as different versions of the same software: use labels to distinguish them.
- When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`

https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

https://kubernetes.io/docs/tasks/debug-application-cluster/local-debugging/

https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/

https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

# Docs that may be out-of-scope for CKAD

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- Like a `Deployment`, a `StatefulSet` manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a `sticky identity` for each of their Pods.
- StatefulSet represents a set of pods with consistent identities, defined as: `Network`, a single stable DNS and hostname, `Storage`, as many `VolumeClaims` as requested. The StatefulSet guarantees that a given network identity will always map to the same storage identity.
- Use cases / limitations:
  - The storage for a given Pod must either be provisioned by a `PersistentVolume` based on the requested storage class, or pre-provisioned by an admin.
  - Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet.
  - StatefulSets currently require a `headless service` to be responsible for the network identity of the Pods.
  - To achieve `ordered and graceful` termination of the pods in the StatefulSet, scale the StatefulSet down to 0 prior to deletion.
- Naming example of a ReplicaSet named `web` and headless service named `nginx`:
  - Domain: nginx.default.svc.cluster.local
  - Pod DNS: web-{0..N-1}.nginx.default.svc.cluster.local
  - Pod Hostname: web-{0..N-1}
- Deployment / Scaling:
  - For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
  - When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
  - Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready. It will proceed in the same order as Pod termination (from the largest ordinal to the smallest).
  - Before a Pod is terminated, all of its successors must be completely shutdown.
  - `spec.podManagementPolicy` can be `OrderedReady` (default) or `Parallel` (does not wait for Pods to become Running and Ready)
  - `spec.updateStrategy.type` can be `OnDelete` or `RollingUpdates` (default)
- If you update the Pod template to a configuration that never becomes Running and Ready (for example, due to a bad binary or application-level configuration error), StatefulSet will stop the rollout and wait. After reverting the template, you must also delete any Pods that StatefulSet had already attempted to run with the bad configuration. StatefulSet will then begin to recreate the Pods using the reverted template.

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
- A `DaemonSet` ensures that all (or some) `Nodes` run a copy of a `Pod`. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created. Useful for running storage, log and monitoring daemons.
- A Pod Template in a DaemonSet must have a `RestartPolicy: Always` (default)
- If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that node selector.
- If you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that node affinity. - If you do not specify either, then the DaemonSet controller will create Pods on all nodes.
- Communicating with Daemon Pods:
  - Push: Pods in the DaemonSet are configured to send updates to another service, such as a stats database.
  - NodeIP and Known Port: Pods in the DaemonSet can use a hostPort, so that the pods are reachable via the node IPs.
  - DNS: Create a `headless service` with the same pod selector, and then discover DaemonSets using the endpoints resource.
  - Service: Create a service with the same Pod selector, and use the service to reach a daemon on a random node (No way to reach specific node.)
- Update strategies: `OnDelete` (new DaemonSet pods will only be created when you manually delete old DaemonSet pods.) and `RollingUpdate` (default)
- If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete Pods from newly not-matching nodes.
- It is possible to create Pods by writing a file to a certain directory watched by Kubelet. These are called `static pods`. Unlike DaemonSet, static Pods cannot be managed with kubectl or other Kubernetes API clients. Static Pods do not depend on the apiserver, making them useful in cluster `bootstrapping` cases. Also, static Pods may be deprecated in the future.
- Use a `Deployment` for stateless services, like frontends, where scaling up and down the number of replicas and rolling out updates are more important than controlling exactly which host the Pod runs on. Use a `DaemonSet` when it is important that a copy of a Pod always run on all or certain hosts, and when it needs to start before other Pods.

https://kubernetes.io/docs/concepts/services-networking/ingress/
- Ingress exposes HTTP and HTTPS routes from outside the cluster to `services` within the cluster. Traffic routing is controlled by rules defined on the Ingress resource. `Internet => Ingress => Services`
- An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name based virtual hosting.
- **An Ingress does not expose arbitrary ports or protocols**. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type `Service.Type=NodePort` or `Service.Type=LoadBalancer`.
- You must have an `ingress controller` (like ingress-nginx) to satisfy an Ingress. Only creating an Ingress resource has no effect.
- You can secure an `Ingress` by specifying a `Secret` that contains a TLS private key and certificate. Currently the Ingress only supports a single TLS port, `443`, and assumes TLS termination. If the TLS configuration section in an Ingress specifies different hosts, they are multiplexed on the same port according to the hostname specified through the SNI TLS extension (provided the Ingress controller supports SNI).
- Types of Ingress:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  # Single Service Ingress (default backend with no rules)
  backend:
    serviceName: testsvc
    servicePort: 80
---
  # Simple fanout (routes traffic from a single IP address to more than one Service)
  # foo.bar.com -> 178.91.123.132 -> / foo    service1:4200
  #                                 / bar    service2:8080
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: service1
          servicePort: 4200
      - path: /bar
        backend:
          serviceName: service2
          servicePort: 8080
---
  # Name based virtual hosting (routing traffic to multiple host names at the same IP address.)
  # foo.bar.com --|                 |-> foo.bar.com service1:80
  #               | 178.91.123.132  |
  # bar.foo.com --|                 |-> bar.foo.com service2:80
  spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
```

https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/
- Pods can have priority. Priority indicates the importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the scheduler tries to preempt (evict) lower priority Pods to make scheduling of the pending Pod possible.
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000 # 32-bit integer, the higher the number, the higher the priority
globalDefault: false # default value. If true, will be used as defaults for Pods (only one is allowed to be true)
preemptionPolicy: PreemptLowerPriority # default value. If 'Never', will be scheduled ahead of other lower-priority pods, but cannot preempt other pods; subject to scheduler back-off.
description: "This priority class should be used for XYZ service pods only."
---
# Usage
kind: Pod
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: high-priority
```
- A Node is considered for preemption only when the answer to this question is yes: "If all the Pods with lower priority than the pending Pod are removed from the Node, can the pending Pod be scheduled on the Node?"
- Please note that `Pod P` is not necessarily scheduled to the `nominated Node`. After victim Pods are preempted, they get their `graceful termination period`. If another node becomes available while scheduler is waiting for the victim Pods to terminate, scheduler will use the other node to schedule `Pod P`. As a result `nominatedNodeName` and `nodeName` of Pod spec are not always the same. Also, if scheduler preempts Pods on `Node N`, but then a higher priority Pod than `Pod P` arrives, scheduler may give `Node N` to the new higher priority Pod. In such a case, scheduler clears nominatedNodeName of `Pod P`. By doing this, scheduler makes Pod P eligible to preempt Pods on another Node.
- When there are multiple nodes available for preemption, the scheduler tries to choose the node with a set of Pods with lowest priority. However, if such Pods have `PodDisruptionBudget` that would be violated if they are preempted then the scheduler may choose another node with higher priority Pods (`PodDisruptionBudget` is supported, **but not guaranteed**)

https://kubernetes.io/docs/concepts/workloads/pods/disruptions/
- Kubernetes offers features to help run `highly available` applications at the same time as frequent `voluntary` disruptions. We call this set of features `Disruption Budgets`.
- Not all voluntary disruptions are constrained by Pod Disruption Budgets. For example, deleting deployments or pods bypasses `Pod Disruption Budgets`.
- An Application Owner can create a PodDisruptionBudget object (PDB) for each application. A PDB limits the number of pods of a replicated application that are down simultaneously from voluntary disruptions. For example, a quorum-based application would like to ensure that the number of replicas running is never brought below the number needed for a quorum. A web front end might want to ensure that the number of replicas serving load never falls below a certain percentage of the total.
- Cluster managers and hosting providers should use tools which respect Pod Disruption Budgets by calling the Eviction API instead of directly deleting pods or deployments. Examples are the `kubectl drain`.
```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  minAvailable: 2
  selector: # similar selector than the one used in a Deployment
    matchLabels:
      app: zookeeper
```

https://kubernetes.io/docs/concepts/policy/resource-quotas/
- A `ResourceQuota`, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.
- `Resource quotas` divides up aggregate cluster resources, but it creates no restrictions around nodes: pods from several namespaces may run on the same node.
- ResourceQuotas are independent of the cluster capacity. They are expressed in absolute units. So, if you add nodes to your cluster, this does not automatically give each namespace the ability to consume more resources.
- A pod is in a `terminal state` if `.status.phase` in (Failed, Succeeded) is true.
- Pods can be created at a specific priority. You can control a pod’s consumption of system resources based on a pod’s priority, by using the `scopeSelector` field in the quota spec.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  # Every Container must have a memory request and cpu request.
  hard:
    # compute quota
    requests.cpu: "1" # Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.
    requests.memory: 1Gi # Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value.
    # storage quota
    requests.storage: 500Gi # Across all persistent volume claims, the sum of storage requests cannot exceed this value.
    gold.storageclass.storage.k8s.io/requests.storage: 500Gi # Across all persistent volume claims associated with the storage-class-name, the sum of storage requests cannot exceed this value.
    # Object count quota
    configmaps: 50 # The total number of config maps that can exist in the namespace.
    pods: 500
    services: 100
```

https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

- Pod `.spec.nodeName` is the simplest form of node selection constraint, but **due to its limitations it is typically not used**. If it is provided, it takes precedence over the other methods for node selection.
- `Node affinity` is conceptually similar to `nodeSelector` – it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector: # Simplest form of node selection constraint
    disktype: ssd # can be scheduled on nodes having this label
  affinity:
    # Node Affinity
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # required is a "hard" requirement
        nodeSelectorTerms: # multiple nodeSelectorTerms are ANDed (all must be satisfied)
        - matchExpressions: # multiple matchExpressions are ORed (one must be satisfied)
          - key: kubernetes.io/e2e-az-name
            operator: In # valid operators: In, NotIn, Exists, DoesNotExist, Gt, Lt. Use NotIn and DoesNotExist for node anti-affinity
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution: # preferred is a "soft" requirement
      - weight: 1 # valid values [1-100], the computed nodes with higher weights are preferred
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
```

- `Inter-pod affinity` and `anti-affinity` allow you to constrain which `nodes` your `pod` is eligible to be scheduled based on labels on pods that are already running on the node rather than based on labels on nodes. 
- `Rules` are of the form "this pod should (or, in the case of anti-affinity, should not) run in an `X` if that `X` is already running one or more pods that meet rule `Y`". 
- `Y` is expressed as a LabelSelector with an optional associated list of `namespaces`.
- `X` is a topology domain like `node`, `rack`, `cloud provider region`. You express it using a `topologyKey` which is the key for the node label that the system uses to denote such a topology domain. Examples: `kubernetes.io/hostname`, `topology.kubernetes.io/zone`, `topology.kubernetes.io/region`, `node.kubernetes.io/instance-type`, `kubernetes.io/os`, `kubernetes.io/arch`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          # do not co-locate web-store pods in the same node
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In # Valid operators: In, NotIn, Exists, DoesNotExist
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          # co-locate web-store pods with cache pods
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - cache
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine

# Will result in the following scheduling:
# node1: web1 && cache1
# node2: web2 && cache2
# node3: web3 && cache3
```

https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
- `Node affinity`, is a property of pods that `attracts` them to a set of nodes; `Taints` are the opposite - they allow a node to `repel` a set of pods.
- `Taints` are applied to a `node`; this marks that the node should not accept any pods that do not tolerate the taints. `Tolerations` are applied to `pods`, and allow (but do not require) the pods to schedule onto nodes with matching taints.

```bash
# Add a taint
kubectl taint nodes node1 key=value:NoSchedule
# Remove a taint
kubectl taint nodes node1 key:NoSchedule-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
    # value: only necessary if using the operator "Equal"
---
  # Tolerates everything (matches all keys, values and effects)
  tolerations:
  - operator: "Exists"
---
  # Matches all effects with key `mykey`.
  tolerations:
  - key: "mykey"
    operator: "Exists"
```

- You can put multiple taints on the same node and multiple tolerations on the same pod. The way Kubernetes processes multiple taints and tolerations is like a `filter`: start with all of a node's taints, then ignore the ones for which the pod has a matching toleration; the remaining `un-ignored` taints have the indicated effects on the pod. In particular:
  - if there is at least `one un-ignored` taint with effect `NoSchedule` then Kubernetes `will not schedule` the pod onto that node.
  - if there is `no un-ignored` taint with effect `NoSchedule` but there is at least `one un-ignored` taint with effect `PreferNoSchedule` then Kubernetes will `try to not schedule` the pod onto the node.
  - if there is at least `one un-ignored` taint with effect `NoExecute` then the pod `will be evicted` from the node (if it is already running on the node), and `will not be scheduled` onto the node (if it is not yet running on the node).
- Normally, if a taint with effect NoExecute is added to a node, then any pods that do not tolerate the taint will be evicted immediately, and pods that do tolerate the taint `will never be evicted`. However, a toleration with `NoExecute` effect can specify an optional `tolerationSeconds` field that dictates how long the pod will stay bound to the node after the taint is added.
- Kubernetes `automatically` adds a toleration for `node.kubernetes.io/not-ready` with `tolerationSeconds=300` and `node.kubernetes.io/unreachable` with `tolerationSeconds=300`.
-  `DaemonSet` are created with these tolerations plus other tolerations like `node.kubernetes.io/memory-pressure`, `node.kubernetes.io/disk-pressure`, `node.kubernetes.io/network-unavailable (host network only)` **without `tolerationSeconds` so they are never evicted**.

https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- **Highly recommended to read/watch the following link: https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/**
- K8s provides no API objects for users (something like we have for Pods, Deployments)
- User management must be configured by the cluster administrator:
  - Certificate-based auth, Token-based auth, Basic auth, OAuth2
- K8s is configured with a `CA`, so every cert signed with this CA will be accepted by the k8s API. You can use `OpenSSL` or `CloudFlare's PKI toolkit` to create these certs.
  - `/etc/kubernetes/pki/[ca.crt,ca.key]`
  - Important fields in the SSL cert: `Common Name (CN)`: k8s uses it as the `user` && `Organization (O)`: k8s uses it as the `group`.
- RBAC in Kubernetes
  - Three groups: Subjects, API Resources, Operations (verbs)
  - Roles: `Operations => API Resources`.
  - RoleBindings: `Role => Subects`.
  - ClusterRoles && ClusterRoleBindings are very similar but without namespaces (apply to the whole cluster)
    - K8s comes with predefined ClusterRoleBindings, you can use themp (eg: add the group in the Orgnatization field of your certificate)
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: myns
  name: admin
rules:
  - apiGroups: [] # empty if using resources from core, can be "*"
    resources: ["pods"] # can be "*"
    verbs: ["get", "list"] # can be "*"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
  namespace: myns
  name: test
subjects:
  - kind: Group # can be "user", "ServiceAccount"
    name: devs
    apiGroup: rbac.authorization.k8s.io
roleRef: # Only one role per binding!
  kind: Role
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

# Nice readings

- https://codeburst.io/the-ckad-browser-terminal-10fab2e8122e
- https://killer.sh/course/preview/052229bd-1062-44a4-8aae-f50d0770165a
