# Ametnes Agent Helm Chart

## Getting started
1. `helm dependency update`
2. `helm template . -f values.yaml -n ametnes-system --name-template=ametnes-cloud-agent --namespace=ametnes-system`

## OpenEbs
1. `helm repo add openebs https://openebs.github.io/charts`
2. `helm repo update`
3. `helm install openebs --namespace openebs openebs/openebs --version 2.10.0 --create-namespace -f /Users/msekamanya/Projects/ametnes-infrastructure/src/utilities/rke/openebs.yml`
4. `kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'`

## Secrets
1. `kubectl create secret docker-registry gitlab-prod-pull-secrets -n ametnes-system-test --docker-server=registry.gitlab.com --docker-username=ametnes+cloud+token --docker-password=__`
2. `kubectl create secret generic client-tls -n ametnes-system-test --from-file=ca.crt=./ca.crt --from-file=client.crt=./client.crt --from-file=client.key=./client.key`

## Unittest
https://github.com/vbehar/helm3-unittest/blob/master/DOCUMENT.md