# Golang unit tests

### Run existing tests
- To run the existing tests you have to set two environment variables
```bash
export GOOGLE_CLOUD_PROJECT="<YOUR_GOOGLE_CLOUD_PROJECT>"
export GOOGLE_APPLICATION_CREDENTIALS="<PATH_TO_THE_SERVICE_ACCOUNT_KEY_FILE>"
```
- Move into the test directory and recursively execute the tests
```bash
cd anthos-bm-gcp-terraform/test
go test -v -timeout 30m ./...
```

### FAQ for diagnosing failed tests in the CI

These tests run on the [Github actions runners](https://console.cloud.google.com/compute/instances?project=anthos-gke-samples-ci) setup in the `anthos-gke-samples-ci` project. These runners are configured following [these steps](/.github/README.md) with the environment setup done using the [`gh_runner_dependencies.sh` script](/.github/gh_runner_dependencies.sh).

#### - OAuth token related error

```sh
Example:

Returning due to fatal error: FatalError{Underlying: error while running command: exit status 1; ╷
│ Error: Get "https://compute.googleapis.com/compute/v1/projects/anthos-gke-samples-ci/zones?alt=json&filter=&prettyPrint=false": oauth2: cannot fetch token: 400 Bad Request
│ Response: {"error":"invalid_grant","error_description":"Invalid JWT Signature."}
│
│   with module.compute_instance["terratest-kcyudp"].data.google_compute_zones.available,
│   on .terraform/modules/compute_instance/modules/compute_instance/main.tf line 32, in data "google_compute_zones" "available":
│   32: data "google_compute_zones" "available" {
```

This is probably due to the **service account key** used by the runner VMs are expired. You can validate this by visiting the [IAM -> Service Accounts page](https://console.cloud.google.com/iam-admin/serviceaccounts/details/110270208213450704617/keys?project=anthos-gke-samples-ci). If you see that the keys are expired then you must SSH into [both the runners](https://console.cloud.google.com/compute/instances?project=anthos-gke-samples-ci) used by this repository execute the following steps as seen in the [`gh_runner_dependencies.sh` script](/.github/gh_runner_dependencies.sh#L48-L53).

```sh
cd /tmp
gcloud auth login
PROJECT_ID=anthos-gke-samples-ci
SERVICE_ACCOUNT_NAME=gh-actions-anthos-samples-sa
gcloud iam service-accounts keys create sa-key.json --iam-account=${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
sudo mv ./sa-key.json /var/local/gh-runner/
sudo chmod 444 /var/local/gh-runner/sa-key.json
```
