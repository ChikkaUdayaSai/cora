## Rasa EKS setup

Rasa with AWS EKS and Helm. The main Rasa docs on this are [here](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#kubernetes-openshift). Rasa Helm chart git repo is [here](https://github.com/rasahq/rasa-x-helm).

- Create AWS EKS Cluster
  - Need an EKS role first. [This page](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html) has instructions on how to check if the role exists. [This page](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role) describes how to create the role.
  - Make sure you are in the right AWS region before creating the EKS Cluster
- [Amazon page](https://docs.aws.amazon.com/eks/latest/userguide/helm.html) on using Helm with EKS
  - [Install aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) locally first
  - [Create a kubeconfig for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)
    - `aws eks --region us-west-2 update-kubeconfig --name wa-covid-bot`
  - `export KUBECONFIG=~/.kube/wa-covid`

## ToDo

- Action code storage [options](https://aws.amazon.com/premiumsupport/knowledge-center/eks-persistent-storage/) are EBS (block storage) or EFS.
- Deploy issue:
  - `Error from server (BadRequest): container "rasa-x" in pod "rasa-x-1584834511-rasa-x-68f7669879-vrhcn" is waiting to start: trying and failing to pull image`
  - `Error from server (BadRequest): container "rasa-x" in pod "prod-rasa-x-5bbdddd665-9nb68" is waiting to start: image can't be pulled`
- Setup channel `credentials.yml` which go under `values.yml` for Helm. See [docs](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#configure-rasa-open-source-channels)
- Update app docker image in `values.yml` (I presume we will need this for additional libraries - dynamoDB)
- Figure out how to assign AWS instance types to Rasa node types to reduce resource usage
  - `values.yml` [nodeSelector](https://github.com/RasaHQ/rasa-x-helm/blob/23ec145c99d68395b9dcdfa760c753943ccd20b4/charts/rasa-x/values.yaml#L201)
  - Recommended [container specs](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#deploy-to-a-cluster-logging)
  - AWS [instance specs](https://aws.amazon.com/ec2/instance-types/)

## Rasa k8s Notes

- [Rasa Enterprise Install](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#rasa-enterprise-installation)
- [Creating Rasa X users](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#create-update-rasa-x-users)
- [HTTPS Setup](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#using-https)
- [Access Logs](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#accessing-logs)

## Rasa EKS setup

Rasa with AWS EKS and Helm. The main Rasa docs on this are [here](https://rasa.com/docs/rasa-x/installation-and-setup/openshift-kubernetes/#kubernetes-openshift). Rasa Helm chart git repo is [here](https://github.com/rasahq/rasa-x-helm).

- Create AWS EKS Cluster
  - Need an EKS role first. [This page](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html) has instructions on how to check if the role exists. [This page](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role) describes how to create the role.
  - Make sure you are in the right AWS region before creating the EKS Cluster
- [Setup Node Groups](https://docs.aws.amazon.com/eks/latest/userguide/launch-workers.html)
  - Initial t3.small, 8 nodes initially, 12 max?
  - [Create Node IAM Role](https://docs.aws.amazon.com/eks/latest/userguide/worker_node_IAM_role.html)
  - [Create Workder Node Role](https://docs.aws.amazon.com/eks/latest/userguide/worker_node_IAM_role.html#create-worker-node-role)
- [Amazon page](https://docs.aws.amazon.com/eks/latest/userguide/helm.html) on using Helm with EKS
  - [Install aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) locally first
  - [Create a kubeconfig for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)
    - `aws eks --region us-west-2 update-kubeconfig --name wa-covid-bot`
  - `export KUBECONFIG=~/.kube/wa-covid`
  - `kubectl get svc`

## EKS Deployment Notes

From the directory containing the `values.yml` do the following:

```
export KUBECONFIG=~/.kube/wa-covid
kubectl --namespace <your namespace> \
kubectl get svc
kubectl create secret docker-registry gcr-pull-secret \
    --docker-server=gcr.io \
    --docker-username=_json_key \
    --docker-password="$(cat gcr-auth.json)"
kubectl create ns wa-covid-bot
helm repo add rasa-x https://rasahq.github.io/rasa-x-helm
helm install \
    --generate-name \
    --namespace wa-covid-bot \
    --values values.yml \
    rasa-x/rasa-x
kubectl --namespace wa-covid-bot get pods
kubectl --namespace wa-covid-bot logs rasa
kubectl --namespace wa-covid-bot describe pod rasa-x
kubectl --namespace wa-covid-bot get service \
    -l app.kubernetes.io/component=nginx \
    -o jsonpath="{.items..status..loadBalancer..ingress[0].ip}"
```

### Helm Cheatsheet

Preface all of these commands with `helm`.

| CMD                                                                     | Info            |
| ----------------------------------------------------------------------- | --------------- |
| install prod --namespace wa-covid-bot --values values.yml rasa-x/rasa-x |                 |
| list --namespace wa-covid-bot                                           | list releases   |
| uninstall --namespace wa-covid-bot prod                                 | uninstall chart |
| repo list --namespace wa-covid-bot                                      | List repos      |
| status --namespace wa-covid-bot prod                                    | Status          |
| upgrade --namespace wa-covid-bot rasa-x-1584837298 rasa-x/rasa-x        |                 |
| history rasa-x-1584837298 --namespace wa-covid-bot                      | Release history |

### Kubectl Cheatsheet

Preface all of these commands with `kubectl` and you need to have `KUBECONFIG` set to your config file.

`export KUBECONFIG=~/.kube/wa-covid`

| CMD                                 | Info              |
| ----------------------------------- | ----------------- |
| --namespace wa-covid-bot get pods   | Show running pods |
| --namespace wa-covid-bot logs <pod> | Show pod logs     |
