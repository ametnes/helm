# Ametnes Agent Helm Chart

## Getting started
### Ametnes Cloud account setup
1. Login into your Ametnes Cloud account.
2. Create a location and note the location ID. For example `8f3342ff`
3. Create your location certificates as well as the docker pull secrets.
4. Save the certificates to files `location_id-ca.crt`, `location_id-client.crt`, `location_id-client.key`. For example `8f3342ff-ca.crt`, `8f3342ff-client.crt`, `8f3342ff-client.key`

### Installing
With your location setup in Ametnes Cloud, install the Ametnes Cloud agent in your kubernetes cluster.
1. Add the ametnes helm repo `helm repo add ametnes https://ametnes.github.io/helm`
2. Create the ametnes namespace `kubectl create namespace ametnes-system`
3. Create docker pull secrets with `kubectl create secret docker-registry gitlab-prod-pull-secrets --docker-server=registry.gitlab.com --docker-username=<docker-secret-user> --docker-password=<docker-secret-password> --docker-email=your-email@domain.net -n ametnes-system`
4. Create the client-tls secret with `kubectl create secret generic client-tls --from-file=ca.crt=8f3342ff-ca.crt --from-file=client.crt=8f3342ff-client.crt --from-file=client.key=8f3342ff-client.key -n ametnes-system`
5. Install ametnes cloud agent `helm install --namespace ametnes-system --set agent.config.location=<location_id> --set agent.config.debug=true ametnes-cloud-agent ametnes/cloud-agent`. Where location_id is the one created in your account setup

### Create resources.
You can now start creating resources in your kubernetes clusters such as databases, monitoring and caching resources.

## Known Issues
These are some known issues that we are working to fix.
1. You may need to restart the agent deployment multiple times before it comes online with `kubectl rollout restart deploy/ametnes-cloud-agent -n ametnes-system`. This is to allow it to register with the control plane.

```
2022-11-13 19:42:01,554 INFO bin.tasks.process Setting up scheduled task fetch_accounts_certs at interval 3600
2022-11-13 19:42:01,560 ERROR bin.util.scheduler Error running function fetch_accounts_certs
Traceback (most recent call last):
  File "util/scheduler.py", line 11, in schedule
  File "tasks/account_management.py", line 34, in fetch_accounts_certs
  File "util/api.py", line 166, in get_location_resources
ValueError: client_id must be supplied
```

Once the restart is done, you should see the logs

```
2022-11-13 19:44:58,845 INFO bin.tasks.process Setting up scheduled task fetch_accounts_certs at interval 3600
2022-11-13 19:45:08,937 INFO bin.tasks.account_management Fetching certs for 0 resources 
```

2. Location may show `offline` even after the agent is up and running. You may have to keep refreshing until the location shows as `online`.

3. Sometimes when creating resource, a location may be flagged `offline` even if it is online. You just need to try creating the resource again.


# Releases
| Date | Release |
| ------ | --------- |
| 27 Nov 2022 | 0.1.037c8e7e |
| 12 Nov 2022 | 0.1.53ad9165 |
