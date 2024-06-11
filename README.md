# Demo Keycloak and Passkeys on OpenShift Local

1. Install and setup OpenShift Local by following [these directions]([url](https://console.redhat.com/openshift/create/local)), snag the OpenShift Client (oc cli) from the [product page]([url](https://access.redhat.com/downloads/content/290/ver=4.15/rhel---9/4.15.16/x86_64/product-software)) and get the helm cli. 
2. Get OpenShift Local started and login as a developer user using `crc start && oc login -u developer -p developer https://api.crc.testing:6443`
3. Get into a new project using `oc new-project keycloak`
4. Use the the helm CLI to deploy the Helm Chart located at `quay.io/kugupta/helm-rh-keycloak`
    - can be done using `helm install demo oci://quay.io/kugupta/helm-rh-keycloak`
5. It should look like the image below if I didn't bork the chart

6. Click the open URL button (highlighted in red above) to be redirected to the admin console (example: my admin console is located at https://demo-helm-rh-keycloak-keycloak.apps-crc.testing/admin/master/console/)
7. Use the default credentials below to login (yes I know, I love the irony)
    - Username: admin
    - Password: admin
8. Create a new realm called `openshift`. This will be the realm we hook up into OpenShift for authentication!

9. Create a new client within the `openshift` realm with a Client ID of `ocp` and click next, turn on client authentication and hit next. That should bring you to the Login settings page, where you can emulate the following closley depending on your OpenShift local configuration:
  - Root URL: `https://oauth-openshift.apps-crc.testing`
  - Home URL: `https://console-openshift-console.apps-crc.testing/`
  - Valid redirect URI: `https://oauth-openshift.apps-crc.testing/oauth2callback/keycloak`
  - Click save
10. Check out the credentials tab of your new `ocp` client and save the client secret for later
11. Create a user and set a temporary password for them for later testing:

12. To hook Keycloak up to OpenShift, you can follow [these steps]([url](https://docs.openshift.com/container-platform/4.15/authentication/identity_providers/configuring-oidc-identity-provider.html)) but I'll summarize them below
    - Create a secret object of your ocp client secret (from step 10) using the following command: `oc create secret generic keycloak-client-secret --from-literal=clientSecret=<paste-client-secret-here> -n openshift-config`
    - Obtain your default router ca.crt from the `router-ca` secret in the `openshift-ingress-operator` namespace (or use your user configured one) and create a config map using the following: `oc create configmap ca-config-map --from-file=ca.crt=/path/to/router-ca's-tls.crt -n openshift-config`
    - A sample custom resource OAuth object is included in the documentation and under `oauth-keycloak.yml` using the parameters above
    - Once you have that configured to your parameters, throw it into OpenShift using `oc apply -f </path/to/oauth-keycloak.yml>`
    - Wait for the authentication operator to refresh and you should see an option to login using keycloak!
    - Setup passkey flow in place of the browser flow and make passwordless required for registration. Change the flow and enable registration for ease of use.
    - Go to `https://demo-helm-rh-keycloak-keycloak.apps-crc.testing/realms/openshift/account` and setup your passkey
    - Test it out by trying to log into the console! 
