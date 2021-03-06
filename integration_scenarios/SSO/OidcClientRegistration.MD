# Register an OIDC Client Using the OidcClientRegistration Custom-Resource
## Example
```
apiVersion: integrations.sdk.management.ibm.com/v1beta1
kind: OidcClientRegistration
metadata:
  name: oidcclientregistration-sample
  namespace: default
spec:
  registration:
    post_logout_redirect_uris:
      - https://ibm.com
    trusted_uri_prefixes:
      - https://ibm.com
    redirect_uris:
      - https://ibm.com
    client_secret:
      secretName: client-reg-secret
      secretKey: client_secret
    apply_client_secret: true
```
This example shows the minimum set of values required to create the oidc-client registration. For information about all available oidc-client options see documentation [here](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_client_registration.html).
### Templated URLs
As with the [`NavMenuEntry`](../navmenu_integration/README.MD) resource, the urls in this resource can be templated. Any usage of `{{ .OpenShiftBaseUrl }}` will be automatically replaced with the OpenShift Base-URL for the cluster. If using Helm, the `{` and `}` characters must be escaped to prevent Helm from interpreting them as templates. Escaping can be done like so: `"{{ "{{" }} .OpenShiftBaseUrl {{ "}}" }}"`
## Managing the Client-Secret in Kubernetes
In the above example, these fields define the Kubernetes secret that will hold the value of the oidc-client secret:
```
client_secret:
  secretName: client-reg-secret
  secretKey: client_secret
```
When `apply_client_secret` is set to `true` a new Kubernetes secret with the specified `secretName` will automatically be created in the same namespace as the OidcClientRegistration, which will contain the automatically generated client secret in the key specified in `secretKey`. When `apply_client_secret` is set to `false`, the caller is expected to create and manage this Kubernetes secret instead. The provided value in the specified key will be registered as the client-secret for the new OIDC client.
## Acquiring the ClientID
Once the custom-resource has been reconciled, a ClientID will automatically be chosen for it. The generated ClientID will follow this format: `sdk.management.ibm.com.{{ .metadata.namespace }}.{{ .metadata.name }}`, where the namespace and name refer to the OidcClientRegistration resource in Kubernetes. The ClientID will also appear in the `.status.state` field of the resource once it has been reconciled. This can be verified like so:
```
$ oc get oidc oidcclientregistration-sample

NAME                            STATUS       CLIENT_ID
oidcclientregistration-sample   CONFIGURED   sdk.management.ibm.com.default.oidcclientregistration-sample
```
## Acquiring the OIDC Provider Endpoints and ClientID
If `apply_client_secret` is set to `true` then the generated secret will also contain two additional data fields that can be referenced in order to acquire the OIDC provider URL and client-id. These fields will be `BASE_OIDC_URL`, `OIDC_ISSUER_URL`, and `CLIENT_ID` which will be set to these values:
```
BASE_OIDC_URL: https://cp-console.apps.<OpenShiftBaseUrl>/oidc/endpoint/OP
OIDC_ISSUER_URL: https://cp-console.apps.<OpenShiftBaseUrl>/idauth/oidc/endpoint/OP
CLIENT_ID: sdk.management.ibm.com.< .metadata.namespace >.< .metadata.name >
```

## CloudPak Certificate Secret
In some environments, the OIDC endpoints may be hosted using self-singed certificates. These certificates can be imported in X509 format from the `cs-ca-certificate-secret` secret in any namespace that has at least one `OidcClientRegistration` resource.
