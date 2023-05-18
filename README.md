# kubernetes-security-policies

Before deploying manifests into Kubernetes, certain tools can scan for security vulnerabilities. These tools can be integrated into your CD pipeline to reduce the risk of deploying a vulnerable pod. 

## Prerequisite 

before starting you need to install these tools:
- Helm
- Kubectl-slice
- Git
- Conftest

you can install the above tools with the following commands:

```bash
#!/bin/bash

sudo apt install git wget -y

########## installing helm ##########
wget -c https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz -O helm.tar.gz
tar -xvzf helm.tar.gz
sudo mv linux-amd64/helm /usr/bin/

########## installing kubectl-slice ##########
wget -c https://github.com/patrickdappollonio/kubectl-slice/releases/download/v1.2.6/kubectl-slice_linux_x86_64.tar.gz -O kubectl-slice.tar.gz
tar -xvzf kubectl-slice.tar.gz
sudo mv kubectl-slice /usr/bin/

########## installing conftest ##########
wget -c https://github.com/open-policy-agent/conftest/releases/download/v0.42.1/conftest_0.42.1_Linux_x86_64.tar.gz -O conftest.tar.gz
tar -xvzf conftest.tar.gz
sudo mv conftest /usr/bin/
```

## Checking the manifests
Okay, let's say we want to check the security best practices for a sample helm chart like bitname/redis. You could use the kubectl-slice tool to slice the chart into manifest files. Next, clone a set of standard policies and use them to scan the manifests for violence. This process will help you determine if the manifests are violent or not. like this:

```bash
#!/bin/bash

helm pull oci://registry-1.docker.io/bitnamicharts/redis --version 17.11.1 --untar --untardir helm-chart

helm template test ./helm-chart/redis > all_manifests.yaml

kubectl-slice --input-file=all_manifests.yaml --output-dir=manifests

git clone https://github.com/mlkmhd/kubernetes-security-policies policy

for f in manifests/*
do 
  echo "checking $f file"
  conftest test $f
done
```

the output of the above commands will be this:

```bash
checking manifests/configmap-test-redis-configuration.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/configmap-test-redis-health.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/configmap-test-redis-scripts.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/secret-test-redis.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/serviceaccount-test-redis.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/service-test-redis-headless.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/service-test-redis-master.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/service-test-redis-replicas.yaml file

20 tests, 20 passed, 0 warnings, 0 failures, 0 exceptions
checking manifests/statefulset-test-redis-master.yaml file
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master does not have a CPU limit set
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master does not have a memory limit set
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master doesn't drop all capabilities
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master has a UID of less than 10000
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master is not using a read only root filesystem
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master is running as root
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master should not be configured to live in the default namespace
FAIL - manifests/statefulset-test-redis-master.yaml - main - redis in the StatefulSet test-redis-master should use imagePullPolicy=Always

20 tests, 12 passed, 0 warnings, 8 failures, 0 exceptions
checking manifests/statefulset-test-redis-replicas.yaml file
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas does not have a CPU limit set
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas does not have a memory limit set
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas doesn't drop all capabilities
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas has a UID of less than 10000
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas is not using a read only root filesystem
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas is running as root
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas should not be configured to live in the default namespace
FAIL - manifests/statefulset-test-redis-replicas.yaml - main - redis in the StatefulSet test-redis-replicas should use imagePullPolicy=Always

20 tests, 12 passed, 0 warnings, 8 failures, 0 exceptions
```

It's worth noting that even the official `bitnami/redis` may not apply some best practices for certain manifests. So, it's important to exercise caution.
