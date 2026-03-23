# Ametnes Agent Helm Chart
Ametnes cloud helps you easily manage the provisioning of cloud native data services in your private kubernetes clusters - on AWS, GCP, Azure or Private k8s clsuters. With this agent, you are able to easily manage;
1. Database services such as; Postgres, Mysql, Neo4j
2. Caching services such as; Redis
3. Monitoring tools such as; Grafana, Loki

No need to manage different helm charts, for your different k8s clusters.

## Getting started
### Ametnes Cloud account setup
1. Login into your Ametnes Cloud account.
2. Create a location and note the location ID. For example `8f3342ff`

> _OPTIONAL STEPS_: These are only needed if you wish to use client tls authentication between the agent and the Ametnes control plane.
> 
> 3. Create your location certificates as well as the docker pull secrets.
> 4. Save the certificates to files `location_id-ca.crt`, `location_id-client.crt`, `location_id-client.key`. For example `8f3342ff-ca.crt`, `8f3342ff-client.crt`, `8f3342ff-client.key`

### _Installing_
With your location setup in Ametnes Cloud, install the Ametnes Cloud agent in your kubernetes cluster.
1. Add the ametnes helm repo `helm repo add ametnes https://ametnes.github.io/helm && helm repo update`.

>_OPTIONAL STEP_: 
> This step is only needed if you wish to use client tls authentication between the agent and the Ametnes control plane.
>
> 2. Create the ametnes namespace `kubectl create namespace ametnes-system`
> 3. Create the client-tls secret with `kubectl create secret generic client-tls --from-file=ca.crt=8f3342ff-ca.crt --from-file=client.crt=8f3342ff-client.crt --from-file=client.key=8f3342ff-client.key -n ametnes-system`.

4. Install ametnes cloud agent with `helm upgrade --install --create-namespace --namespace ametnes-system --set agent.config.location=<location_id> ametnes-cloud-agent ametnes/cloud-agent`. Where `<location_id>` is the one created in your account setup above.

### Additional TLS trust CAs (optional, runtime-agnostic)
If your environment uses outbound TLS interception, configure `agent.tls.trust`. An init container copies a system CA bundle from the init image (`/cacert.pem` from `curlimages/curl`, or common distro paths), appends PEMs from a Secret, and mounts the merged file at `targetBundlePath` (default `/etc/ssl/certs/ca-certificates.crt`). Works with distroless/minimal agent images. Default `initImage` is `curlimages/curl`. No apt or root required; compatible with `runAsNonRoot` pod security.

Ensure your outbound HTTP client/proxy configuration uses the merged trust bundle path (`targetBundlePath`, default `/etc/ssl/certs/ca-certificates.crt`) for TLS verification.

You can set standard TLS trust env vars to point clients at the merged bundle:
```yaml
agent:
  extraEnvs:
    - name: SSL_CERT_FILE
      value: /etc/ssl/certs/ca-certificates.crt
    - name: REQUESTS_CA_BUNDLE
      value: /etc/ssl/certs/ca-certificates.crt
```

Create the Secret from Helm values (one or more CAs):
```yaml
agent:
  tls:
    trust:
      enabled: true
      additionalCAs:
        - crt: |
            -----BEGIN CERTIFICATE-----
            <your-interception-or-root-ca>
            -----END CERTIFICATE-----
      secret:
        create: true
        name: ""   # optional; defaults to <release>-tls-trust
```

Or use an existing Secret (all keys in that Secret are appended to the bundle):
```yaml
agent:
  tls:
    trust:
      enabled: true
      additionalCAs: []
      secret:
        create: false
        name: customer-ca
```

### Proxy settings (optional)
Configure proxy values in `agent.config.proxy`. These are written into `config.json` for the agent application to use explicitly (no pod-level proxy env vars are injected):
```yaml
agent:
  config:
    proxy:
      enabled: true
      http: http://proxy.example:8080
      https: http://proxy.example:8080
      no_proxy: 127.0.0.1,localhost,.svc,.cluster.local,10.96.0.1,kubernetes.default.svc
```

### _Create resources_.
You can now start creating resources in your kubernetes clusters such as databases, monitoring and caching services.

## Known Issues
These are some known issues that we are working to fix.
1. Location may show `offline` even after the agent is up and running. You may have to keep refreshing until the location shows as `online`.
