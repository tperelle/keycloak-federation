# Deploy the lab

<img src="../docs/images/keycloak-federation.png" width="350px" />

## Kubernetes cluster

Before anything else, we need a kubernetes cluster. I use [Rancher desktop](https://rancherdesktop.io/) to have a local cluster on my laptop but you can use any other kubernetes cluster.

```bash
$ kubectl get nodes          
NAME                   STATUS   ROLES                  AGE   VERSION
lima-rancher-desktop   Ready    control-plane,master   40d   v1.22.7+k3s1
```

## LDAP

We use [OpenLDAP](https://www.openldap.org) as LDAP solution. 

Prepare the deployment:
```bash
# Create a dedicated namespace
kubectl create ns ldap

# Create a secret
kubectl create secret generic openldap --from-literal=adminpassword=adminpassword --from-literal=users=user01,user02 --from-literal=passwords=password01,password02 -n ldap

# Install OpenLDAP
kubectl apply -f openldap/
```

Check that everything is running:
```bash
$ k get all -n ldap                                                    
NAME       READY   STATUS    RESTARTS   AGE
pod/ldap   1/1     Running   0          86s

NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/ldap   ClusterIP   10.43.196.150   <none>        1389/TCP   6s
```



## Keycloak



## Demo app
