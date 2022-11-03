# Ametnes Agent Helm Chart

## Getting started
### Ametnes Cloud account setup
1. Login into your Ametnes Cloud account.
2. Create a location and note the location ID. For example `8f3342ff`
3. Create your location certificates as well as the docker pull secrets.
4. Save the certificates to files `location_id-ca.crt`, `location_id-client.crt`, `location_id-client.key`. For example `8f3342ff-ca.crt`, `8f3342ff-client.crt`, `8f3342ff-client.key`

### Installing
1. Add the ametnes helm repo `helm repo add ametnes https://ametnes.github.io/helm`
2. Create the ametnes namespace `kubectl create namespace ametnes-system`
3. Create docker pull secrets with `kubectl create secret docker-registry gitlab-prod-pull-secrets --docker-server=registry.gitlab.com --docker-username=<docker-secret-user> --docker-password=<docker-secret-password> --docker-email=your-email@domain.net -n ametnes-system`
4. Create package pull secrets with `kubectl create secret generic package-repo-secret --from-literal=PACKAGE_REPO_USERNAME=<docker-secret-user> --from-literal=PACKAGE_REPO_PASSWORD=<docker-secret-password> -n ametnes-system`
5. Create the client-tls secret with `kubectl create secret generic client-tls --from-file=ca.crt=8f3342ff-ca.crt --from-file=client.crt=8f3342ff-client.crt --from-file=client.key=8f3342ff-client.key -n ametnes-system`
6. Install ametnes cloud agent `helm install --namespace ametnes-system --set agent.config.location=<location_id> --set agent.config.debug=true ametnes-cloud-agent ametnes/cloud-agent`. Where location_id is the one created in your account setup

### Create resources.
You can now start creating resources in your kubernetes clusters such as databases, monitoring and caching resources.
