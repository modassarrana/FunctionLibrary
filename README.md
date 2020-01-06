# FunctionLibrary
## This document covers step to be performed for knative installation on cluster

Ansible playbook has roles for istio as well as knatie install & uninstall

``` 
For install 
ansible-playbook deploy-knative.yml -i <inventory>
It downloads istio , helm binary & install them . Deploy knative based on profile ( User input will decide)

For uninstall
ansible-playbook knative-uninstall.yml -i inventory
```

User has to update initial_config Json file for specific release
```
[root@svpod7mgmt002 ansibleAutomation]# cat utilities/initial_config.json
{
        "knative_version": 0.11.0,
        "home_path": "/home/centos/FAAS_Cluster",
        "istio_dir": "{{home_path}}/istio",
        "helm_dir": "{{home_path}}/helm",
        "knative_dir": "{{home_path}}/knative",
        "istio_version": 1.3.5,
        "helm_version":  v2.16.1,
        "sidecar_disable": false, # by defaukt sidecar is enabled. incase if you dont want sidecar , make this true
        "profile": "kiali", // if kiali is not required then set profile value as default
        "docker_artifactory_repo_url": "clna-prototype-docker.apro.nbnco.net.au,ngops-docker.apro.nbnco.net.au",
        "domain": "faas.nrst.nbnco.lab"
}
```

## kiali setup
Kiali will be installed as part of knative installation . To setup dashboard urls , manual activity has to be done

```
Fetch both Jaegar & grafana url from kialiurls file ( its under output foler of ansible playbook)
[root@svpod7mgmt002 output]# ls -lrt
total 12
-rw-r--r--. 1 root root 167 Sep 16 06:02 kiali-secret.yml
-rw-r--r--. 1 root root 472 Dec 16 04:59 dns-config.yml
-rwxrwxrwx. 1 root root  73 Dec 23 02:50 kialiurls
```

Edit kiali configmap using below command
```
kubectl edit cm kiali -n istio-system -o yaml
Ex-> 
    external_services:
      tracing:
        url: http://10.0.41.120:15032
      grafana:
        url: http://localhost:15031

save the changes(using option :wq! in esc mode) like noremal file is saved in vi editor
```

Edit will not reflect the chnages on kiali pod running on kubernetes. Restart pod to have the effect
```
ret=$(kubectl get po -n istio-system | grep 'kiali' | awk '{ print $1 }')

kubectl delete po $ret -n istio-system

```

Verify nelwy created kiali pod is in running state
```
kubectl get po -n istio-system | grep 'kiali'
```
