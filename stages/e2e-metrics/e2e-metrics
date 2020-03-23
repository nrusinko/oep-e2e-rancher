#!/bin/bash

pod() {

## Cloning oep-e2e-rancher repo
echo "Cleaning up the cluster *************"
sshpass -p $pass ssh -o StrictHostKeyChecking=no $user@$ip -p $port 'cd oep-e2e-rancher && bash stages/e2e-metrics/e2e-metrics node '"'$CI_PIPELINE_ID'"' '"'$CI_JOB_ID'"''
}

node() {

# Set environment variables
COVERAGE_NAMESPACE="e2e-metrics"
E2E_METRICS_PIPELINE_ID=$(echo $1)
E2E_METRICS_RUN_ID=$(echo $2)

# Create namespace for e2e-metric components
kubectl create ns $COVERAGE_NAMESPACE

# Clone oep-e2e repo and copy master-plan to oep-e2e-rancher repo
git clone https://github.com/mayadata-io/oep-e2e.git
cp oep-e2e/.master-plan.yml .master-plan.yml

# Create configmap from master test plan file
kubectl create configmap metrics-config-test -n $COVERAGE_NAMESPACE --from-file=.master-plan.yml --from-file=.gitlab-ci.yml

# Cloning e2e-metrics repo 
git clone https://github.com/mayadata-io/e2e-metrics.git

# Creating kubernetes resources 
kubectl apply -f e2e-metrics/deploy/rbac.yaml
kubectl apply -f e2e-metrics/deploy/crd.yaml
kubectl create configmap metac-config-test -n $COVERAGE_NAMESPACE --from-file="e2e-metrics/deploy/metac-config.yaml"
kubectl apply -f e2e-metrics/deploy/operator.yaml
kubectl set env sts/e2e-metrics E2E_METRICS_PIPELINE_ID=$E2E_METRICS_PIPELINE_ID -n $COVERAGE_NAMESPACE
kubectl set env sts/e2e-metrics E2E_METRICS_RUN_ID=$E2E_METRICS_RUN_ID -n $COVERAGE_NAMESPACE
sleep 50

# Fetching coverage percentage from custom resource
e2e_coverage_cr=$(kubectl get pcover -n $COVERAGE_NAMESPACE --no-headers | awk '{print $1}')
kubectl get pcover $e2e_coverage_cr -n $COVERAGE_NAMESPACE -oyaml
kubectl get pcover -n $COVERAGE_NAMESPACE -o=jsonpath='{.items[0].result.coverage}{"\n"}'

}

if [ "$1" == "node" ];then
  node $2 $3
else
  pod
fi