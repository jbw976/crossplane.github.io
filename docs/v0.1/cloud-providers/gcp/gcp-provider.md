# Adding Google Cloud Platform (GCP) to Crossplane

In this guide, we will walk through the steps necessary to configure your GCP account to be ready for integration with Crossplane.
The general steps we will take are summarized below:

* Create a new example project that all resources will be deployed to
* Enable required APIs such as Kubernetes and CloudSQL
* Create a service account that will be used to perform GCP operations from Crossplane
* Assign necessary roles to the service account
* Enable billing

For your convenience, the specific steps to accomplish those tasks are provided for you below using either the `gcloud` command line tool, or the GCP console in a web browser.
You can choose whichever you are more comfortable with.

## Option 1: gcloud Command Line Tool

If you have the `gcloud` tool installed, you can run below commands from the example directory.
It
Instructions for installing `gcloud` can be found in the [Google docs](https://cloud.google.com/sdk/install).

```bash
# list your organizations (if applicable), take note of the specific organization ID you want to use
# if you have more than one organization (not common)
gcloud organizations list

# create a new project
export EXAMPLE_PROJECT_NAME=crossplane-example-123
gcloud projects create $EXAMPLE_PROJECT_NAME --enable-cloud-apis [--organization ORGANIZATION_ID]

# record the PROJECT_ID value of the newly created project
export EXAMPLE_PROJECT_ID=$(gcloud projects list --filter NAME=$EXAMPLE_PROJECT_NAME --format="value(PROJECT_ID)")   

# enable Kubernetes API
gcloud --project $EXAMPLE_PROJECT_ID services enable container.googleapis.com
# enable CloudSQL API
gcloud --project $EXAMPLE_PROJECT_ID services enable sqladmin.googleapis.com 

# create service account
gcloud --project $EXAMPLE_PROJECT_ID iam service-accounts create example-123 --display-name "Crossplane Example"
# export service account email
export EXAMPLE_SA="example-123@$EXAMPLE_PROJECT_ID.iam.gserviceaccount.com"

# create service account key (this will create a `key.json` file in your current working directory)
gcloud --project $EXAMPLE_PROJECT_ID iam service-accounts keys create --iam-account $EXAMPLE_SA key.json 

# assign roles
gcloud projects add-iam-policy-binding $EXAMPLE_PROJECT_ID --member "serviceAccount:$EXAMPLE_SA" --role="roles/iam.serviceAccountUser"
gcloud projects add-iam-policy-binding $EXAMPLE_PROJECT_ID --member "serviceAccount:$EXAMPLE_SA" --role="roles/cloudsql.admin"
gcloud projects add-iam-policy-binding $EXAMPLE_PROJECT_ID --member "serviceAccount:$EXAMPLE_SA" --role="roles/container.admin"
```

## Option 2: GCP Console in a Web Browser

If you chose to use the `gcloud` tool, you can skip this section entirely.

Create a GCP example project which we will use to host our example GKE cluster, as well as our example CloudSQL instance.

- Login into [GCP Console](https://console.cloud.google.com)
- Create a new project (either stand alone or under existing organization)
- Create Example Service Account
    - Navigate to: [Create Service Account](https://console.cloud.google.com/iam-admin/serviceaccounts)
    - `Service Account Name`: type "example"
    - `Service Account ID`: leave auto assigned
    - `Service Account Description`: type "Crossplane example"
    - Click `Create` button
        - This should advance to the next section `2 Grant this service account to project (optional)`
    - We will assign this account 3 roles:
        - `Service Account User`
        - `Cloud SQL Admin`
        - `Kubernetes Engine Admin`
    - Click `Create` button
        - This should advance to the next section `3 Grant users access to this service account (optional)`
    - We don't need to assign any user or admin roles to this account for the example purposes, so you can leave following two fields blank:
        - `Service account users role`
        - `Service account admins role`
    - Next, we will create and export service account key
        - Click `+ Create Key` button.
            - This should open a `Create Key` side panel
        - Select `json` for the Key type (should be selected by default)
        - Click `Create`
            - This should show `Private key saved to your computer` confirmation dialog
            - You also should see `crossplane-example-1234-[suffix].json` file in your browser's Download directory
            - Save (copy or move) this file into example (this) directory, with new name `key.json`
- Enable `Cloud SQL API`
    - Navigate to [Cloud SQL Admin API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview)
    - Click `Enable`
- Enable `Kubernetes Engine API`
    - Navigate to [Kubernetes Engine API](https://console.developers.google.com/apis/api/container.googleapis.com/overview)
    - Click `Enable`

## Enable Billing

No matter what option you chose to configure the previous steps, you will need to enable billing for your account in order to create and use Kubernetes clusters with GKE.

- Go to [GCP Console](https://console.cloud.google.com)
    - Select example project
    - Click `Enable Billing`
- Go to [Kubernetes Clusters](https://console.cloud.google.com/kubernetes/list)
    - Click `Enable Billing`