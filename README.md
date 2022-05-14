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

We use [NGINX](https://nginx.org/en/) as demo application to check if the SSO solution, including OAuth2 Proxy, is working well. In this example we want that users login with Keycloak before they can access the welcome page of NGINX.

Adapt the configuration in `nginx-demo-app/values-nginx.yml`.

Then deploy the demo application:

```bash 
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install nginx-demo-app bitnami/nginx --values nginx-demo-app/values-nginx.yml
```

Then use your favorite browser with private windows to try accessing the demo app `https://nginx-demo-app.ssotest.perelle.com` with or without being logged in before.

- OAuth2 Proxy redirects you to Keycloak login page if your are not already identified
- You can directly access the demo app if you are already identified 