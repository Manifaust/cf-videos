## Verification

```sh
$ git clone https://github.com/pivotal-cf-experimental/kubo-ci.git
$ kubectl apply -f ~/kubo-ci/specs/nginx-lb.yml
$ kubectl get deployments
$ kubectl rollout status deployment/nginx
$ kubectl get svc nginx
$ curl <external IP>
```

### Show Scaling Up

Modify `kubo.yml` to deploy 4 instances instead of 3. Execute `deploy_k8s` again.

### Show Self Healing

Go into GCP console and show the labels for all the VM instances. Find a worker VM and kill it. Show BOSH bringing up another VM and that VM becomes a node. The node then accept pods from the scheduler.

```sh
$ gcloud compute instances list --filter="labels.job=worker"
$ kubectl --kubeconfig=kubeconfig get pods -o=custom-columns=name:{.metadata.name},node:{.spec.nodeName},status:{.status.phase},host-ip:{.status.hostIP},start-time:{.status.startTime}
$ kubectl --kubeconfig=kubeconfig get nodes
```