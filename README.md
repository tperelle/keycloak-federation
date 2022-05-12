# keyclock-federation

The purpose of this project is to test Keycloak using federation with an external LDAP.

<img src="docs/images/keycloak-logo.png" width="250px" />

---

## The lab

<img src="docs/images/keycloak-federation.png" width="350px" />

Install and configure the lab:
- See [How to setup the kubernetes cluster](cluster/README.md)
- See [How to setup  the SSO solution](lab/README.md)

## Deploying a secure applications

### Install OAuth2 Proxy

Some products or applications integrate natively the capability to configure OAuth2 provider. For the others, we can use [OAuth2 Proxy](https://oauth2-proxy.github.io/oauth2-proxy/) to secure the application at the edge with ingress annotations.

**Process overview**

<img src="docs/images/sso-process-overview.png" width="500px" />

1. The customer send a request for the demo app
2. The ingress controler asks OAuth2 Proxy to check both authentication and authorization with Keycloak
3. If the customer is not already anthenticated, OAuth2 Proxy asks him to login 
4. The customer gives its credentials
5.
6.
7. 
8. 

**Installation**

Create a new client application `oauth2-proxy` in Keycloak:

<img src="docs/images/keycloak-oauth2-setup.png" width="400px" />

Save and go to the `Credentials` tab to note the associated secret.

In the `Mappers` tab, create a new `Groups`:

<img src="docs/images/keycloak-oauth2-mappers.png" width="400px" />

Now we can prepare the deployment of OAuth2 Proxy configuring the value file `oauth2-proxy/values-oauth2-proxy.yml`, mainly `clientID`, `clientSecret`, `cookieSecret` and the DNS.

Then deploy OAuth2 Proxy with the embedded chart:

```bash 
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests
helm upgrade --install oauth2-proxy oauth2-proxy/oauth2-proxy --values oauth2-proxy/values-oauth2-proxy.yml
```

We get the `Sign in with Keycloak` option when we access to OAuth2 Proxy ingress domain at https://oauth.ssotest.perelle.com

<img src="docs/images/keycloak-oauth2-login.png" width="400px" />

## Deploy a demo app

We use [NGINX](https://nginx.org/en/) as demo application to check if the SSO solution, including OAuth2 Proxy, is working well. In this example we want that users login with Keycloak before they can access the welcome page of NGINX.

Adapt the configuration in `nginx-demo-app/values-nginx.yml`.

Then deploy the demo application:

```bash 
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install nginx-demo-app bitnami/nginx --values nginx-demo-app/values-nginx.yml
```



