# keyclock-federation

The purpose of this project is to test Keycloak using federation with an external LDAP.

<img src="docs/images/keycloak-logo.png" width="250px" /> <img src="docs/images/openldap-logo.png" width="200px" />

---

## Install the lab

Install and configure the lab:
- See [Setup the kubernetes cluster](cluster/README.md)
- See [Setup the SSO solution](lab/README.md)

## Deploy a secured application

Now we have a SSO solution ready to use, we are going to deploy a first secured application.

We use [NGINX](https://nginx.org/en/) as demo application to check if the SSO solution, including OAuth2 Proxy, is working well. In this example we want that users login with Keycloak before they can access the welcome page of NGINX.

Adapt the configuration in `nginx-demo-app/values-nginx.yml`.

Dploy the demo application:

```bash 
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install nginx-demo-app bitnami/nginx --values nginx-demo-app/values-nginx.yml
```

Then use your favorite browser with private windows to try accessing the demo app https://nginx-demo-app.ssotest.perelle.com with or without being logged in before.

> Note: Log in with the user account you have created in the `Business` realm. It's in this one that we have created the `oauth2-proxy` client application. The `admin` account is in the `master` realm and cannot access it.

Result:
- You get the Keycloak login page if your are not already identified
- You can directly access the demo app if you are already identified 

## Identification process

Let's have a look at the detailled process when we access a secured application:

<img src="docs/images/sso-process-overview.png" width="500px" />

1. Customer requests the demo app
2. Ingress controler redirect to OAuth2 Proxy according annotations in the demo app ingress
3. OAuth2 Proxy checks with Keycloak if the user is authenticated
4. The user is not identified, Keycloak presents the login page
5. The user fills in his credentials for authentcation
6. Keycloak passes identification datas to OAuth2 Proxy
7. OAuth2 Proxy informs Ingress Controler that the user is identified and is authorized to access the application
8. Ingress controler routes the request to the demo app

## Few tests

Now that we have a working platform, it's time to do some tests to understand how it works.

### Deploy an unsecured app

I just want to demonstrate that the security with OAuth2 Proxy is setup in the ingress and that we can continue to deploy unsecured applications.

Adapt the configuration in `nginx-unsecured/values-nginx.yml` that don't contains annotation for OAuth2 Proxy and deploy it:

```bash 
helm upgrade --install nginx-unsecured bitnami/nginx --values nginx-unsecured/values-nginx.yml
```

Check with a new private window that you can access the application https://nginx-unsecured.ssotest.perelle.com without being identified.


### Federation failure

What happens if OpenLDAP goes down and the federation doesn't work anymore ?\
Let's see if I can continue authenticating to Keycloak and accessing the secured application.

Scale OpenLDAP down to 0 replicas:

```bash
kubectl scale --replicas=0 deployment.apps/openldap -n identity
```

Try to access our demo application at https://nginx-demo-app.ssotest.perelle.com with a new private window. 

We are redirected to the Keycloak login page for authentication, and then get an error due to identity provider unavailability:

<img src="docs/images/test-error-federation.png" width="350px" />

Get back to 1 replica for OpenLDAP:

```bash
kubectl scale --replicas=1 deployment.apps/openldap -n identity
```

It works again. So we can conclude that, in this configuration, Keycloak cannot work in an autonomous way if the federation goes down.

> Additional sources:
> - https://www.keycloak.org/docs/latest/server_admin/