# Deploy the lab

The SSO solution is composed of [Keycloak](https://www.keycloak.org) and [OpenLDAP](https://www.openldap.org). 

<img src="../docs/images/keycloak-federation.png" width="350px" />

I rely on the [talkingquickly](https://www.talkingquickly.co.uk/kubernetes-sso-a-detailed-guide) articles for the installation.

## OpenLDAP

### Installation

Configure helm values in `openldap/values-openldap.yml`. 

```yml
# Default configuration for openldap as environment variables. These get injected directly in the container.
# Use the env variables from https://github.com/osixia/docker-openldap#beginner-guide
env:
  LDAP_ORGANISATION: "Thomas Perelle's Demo"
  LDAP_DOMAIN: "ssotest.perelle.com"
  LDAP_BACKEND: "hdb"
  LDAP_TLS: "true"
  LDAP_TLS_ENFORCE: "false"
  LDAP_REMOVE_CONFIG_AFTER_SETUP: "true"
  LDAP_READONLY_USER: "true"
  LDAP_READONLY_USER_USERNAME: readonly
  LDAP_READONLY_USER_MASSWORD: password

# Default Passwords to use, stored as a secret. If unset, passwords are auto-generated.
# You can override these at install time with
# helm install openldap --set openldap.adminPassword=<passwd>,openldap.configPassword=<passwd>
adminPassword: password
configPassword: password

# Custom openldap configuration files used to override default settings
customLdifFiles:
  0-initial-ous.ldif: |-
    dn: ou=People,dc=ssotest,dc=perelle,dc=com
    objectClass: organizationalUnit
    ou: People

    dn: ou=Group,dc=ssotest,dc=perelle,dc=com
    objectClass: organizationalUnit
    ou: Group
```

- With default values the base domain of the cluster is `dc=ssotest,dc=perelle,dc=com`.
- 2 `OrganizationalUnit`'s will be created in the LDAP, `People` and `Group`.

Install OpenLDAP
```bash
$ helm upgrade --install openldap ./charts/openldap --values ./openldap/values-openldap.yml --namespace identity --create-namespace

Release "openldap" does not exist. Installing it now.
NAME: openldap
LAST DEPLOYED: Wed May  4 09:35:59 2022
NAMESPACE: identity
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
OpenLDAP has been installed. You can access the server from within the k8s cluster using:

  openldap.identity.svc.cluster.local:389


You can access the LDAP adminPassword and configPassword using:

  kubectl get secret --namespace identity openldap -o jsonpath="{.data.LDAP_ADMIN_PASSWORD}" | base64 --decode; echo
  kubectl get secret --namespace identity openldap -o jsonpath="{.data.LDAP_CONFIG_PASSWORD}" | base64 --decode; echo


You can access the LDAP service, from within the cluster (or with kubectl port-forward) with a command like (replace password and domain):
  ldapsearch -x -H ldap://openldap.identity.svc.cluster.local:389 -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w $LDAP_ADMIN_PASSWORD


Test server health using Helm test:
  helm test openldap


You can also consider installing the helm chart for phpldapadmin to manage this instance of OpenLDAP, or install Apache Directory Studio, and connect using kubectl port-forward.
```

### Tests

Expose the LDAP locally:
```bash
kubectl port-forward --namespace identity \
    $(kubectl get pods -n identity --selector='release=openldap' -o jsonpath='{.items[0].metadata.name}') \
    3890:389
```

Test that the LDAP works using the `ldapsearch` CLI:
```bash
$ ldapsearch -x -H ldap://localhost:3890 -b dc=ssotest,dc=perelle,dc=com -D "cn=admin,dc=ssotest,dc=perelle,dc=com" -w password
```

This return the content of our LDAP:
```ldif
# extended LDIF
#
# LDAPv3
# base <dc=ssotest,dc=perelle,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# ssotest.perelle.com
dn: dc=ssotest,dc=perelle,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: Thomas Perelle's Demo
dc: ssotest

# admin, ssotest.perelle.com
dn: cn=admin,dc=ssotest,dc=perelle,dc=com
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9SklHbHoxbHJsRXJQS3J4cWl6ODJGeURXTzgrUnNlWnU=

# readonly, ssotest.perelle.com
dn: cn=readonly,dc=ssotest,dc=perelle,dc=com
cn: readonly
objectClass: simpleSecurityObject
objectClass: organizationalRole
userPassword:: e1NTSEF9djg3SUg5RDAySkYvOVZSQVNUdHJJZEtzSnZoQU9kZlc=
description: LDAP read only user

# People, ssotest.perelle.com
dn: ou=People,dc=ssotest,dc=perelle,dc=com
objectClass: organizationalUnit
ou: People

# Group, ssotest.perelle.com
dn: ou=Group,dc=ssotest,dc=perelle,dc=com
objectClass: organizationalUnit
ou: Group

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
```

## Keycloak

Adapt the configuration in `keycloak/values-keycloak.yml`, especially the domain to be concordant with your environement and the OpenLDAP deployment.

```bash
helm repo add codecentric https://codecentric.github.io/helm-charts
helm upgrade --install keycloak codecentric/keycloak --values keycloak/values-keycloak.yml
```

Navigate to the DNS your configured in keycloak configuration, here is `https://sso.ssotest.perelle.com`:

<img src="../docs/images/keycloak-home.png" width="400px" />

If it doesn't work, check if the `keycloak` ingress has been processed in the logs of the nginx controller. It may be some [issues arround the IngressClass](https://kubernetes.github.io/ingress-nginx/).

> Additionnal Sources:
> - [Keycloak helm chart configuration](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak)
> - [NGINX ingress controller troubleshooting](https://kubernetes.github.io/ingress-nginx/troubleshooting/)

