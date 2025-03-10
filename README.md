# Demo Keycloak and Passkeys on OpenShift Local

### Prerequisites
Install and setup OpenShift Local by following [these steps](https://console.redhat.com/openshift/create/local), snag the OpenShift Client (oc cli) from the [product page](https://access.redhat.com/products/...)

### Demo Steps

1. Get OpenShift Local started and login as a developer user using `crc start && oc login -u developer -p developer https://api.crc.testing:6443`
2. Get into a new project using `oc new-project keycloak`
3. Use the the helm CLI to deploy the Helm Chart located at `quay.io/kugupta/helm-rh-keycloak`
    - can be done using `helm install demo oci://quay.io/kugupta/helm-rh-keycloak`
4. It should look like the image below if I didn't bork the chart
![helm deployed](assets/helm-deployed.png)
5. Click the open URL button (highlighted in red above) to be redirected to the admin console (example: my admin console is located at https://demo-helm-rh-keycloak-keycloak.apps-crc.testing/admin/)
6. Use the default credentials below to login (yes I know, I love the irony)
    - Username: admin
    - Password: admin
7. Create a new realm called `openshift`. This will be the realm we hook up into OpenShift for authentication!
![create realm](assets/create-realm.png)
8. Create a new client within the `openshift` realm with a Client ID of `ocp` and click next, turn on client authentication and hit next.
![create client](assets/create-client.png)
10. That should bring you to the Login settings page, where you can emulate the following closely depending on your OpenShift local configuration:
    - Root URL: `https://oauth-openshift.apps-crc.testing`
    - Home URL: `https://console-openshift-console.apps-crc.testing/`
    - Valid redirect URI: `https://oauth-openshift.apps-crc.testing/oauth2callback/keycloak`
    - Click save
11. Check out the credentials tab of your new `ocp` client and save the client secret for later
12. Create a user and set a temporary password for them for later testing:
![create client](assets/create-user.png)
13. To hook Keycloak up to OpenShift, you can follow [these steps](https://docs.openshift.com/container-platform/4.15/authentication/identity_providers/configuring-oidc-identity-provider.html)
    - Create a secret object of your ocp client secret (from step 10) using the following command: `oc create secret generic keycloak-client-secret --from-literal=clientSecret=<paste-client-secret-here>`
    - Obtain your default router ca.crt from the `router-ca` secret in the `openshift-ingress-operator` namespace (or use your user configured one) and create a config map using the following: `oc create configmap keycloak-cert --from-file=ca.crt=<path-to-ca.crt>`
    - A sample custom resource OAuth object is included in the documentation and under `oauth-keycloak.yml` using the parameters above
    - Once you have that configured to your parameters, throw it into OpenShift using `oc apply -f </path/to/oauth-keycloak.yml>`
    - Wait for the authentication operator to refresh and you should see an option to login using keycloak!
    - Setup passkey flow in place of the browser flow and make passwordless required for registration. Change the flow and enable registration for ease of use.
    - Go to `https://demo-helm-rh-keycloak-keycloak.apps-crc.testing/realms/openshift/account` and setup your passkey
    - Test it out by trying to log into the console!
