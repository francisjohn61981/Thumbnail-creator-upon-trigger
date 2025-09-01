These are commands to run on CLI.

### Create a bucket on Google Cloud Storage
```
gcloud storage buckets create gs://$BUCKET_NAME \
  --location=$BUCKET_LOCATION
```
### Create pub/sub topic
```
gcloud pubsub topics create my-topic
```
### before creating the cloud run functions, use the following command to check the supported runtime
```
gcloud functions runtimes list
```
### Create the cloud run functions, with Nodejs22 as runtime and it needs package.json and index.js as source files. (make sure these are available locally)
### index.js must export the function whose name matches --entry-point
```
gcloud functions deploy memories-thumbnail-creator \
  --gen2 \
  --runtime=nodejs22 \
  --region=BUCKET_LOCATION \
  --source=. \
  --entry-point=memories-thumbnail-creator \
  --trigger-http \
  --allow-unauthenticated
```
### Before adding the trigger, be sure to enable Cloud API and Eventarc API from CLI
```
gcloud services enable eventarc.googleapis.com \
    pubsub.googleapis.com \
    storage.googleapis.com \
    run.googleapis.com
```
### Grant necessary roles to service account s
```
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$pubsubServiceAccount" \
  --role="roles/iam.serviceAccountTokenCreator"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$ComputeEngineServiceAccount" \
  --role="roles/eventarc.eventReceiver"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$StorageServiceAccount" \
  --role="roles/pubsub.publisher"

```
### Create the Eventarc trigger
```
gcloud eventarc triggers create trigger-rht4ulze \
 --location=us-central1 \
 --service-account=SERVICEACCOUNTNAME \
 --destination-run-service=memories-thumbnail-creator \
 --destination-run-region=$BUCKET_LOCATION \
 --destination-run-path="/" \
 --event-filters="bucket=$BUCKET_LOCATION" \
 --event-filters="type=google.cloud.storage.object.v1.finalized"
```
