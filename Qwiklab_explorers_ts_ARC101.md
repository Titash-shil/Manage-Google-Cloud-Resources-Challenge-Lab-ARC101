# Manage Google Cloud Resources: Challenge Lab || [ARC101](https://www.cloudskillsboost.google/course_templates/653/labs/461539) ||

# # Like, comment, share & Don't forget to subscribe [Qwiklab_Explorers_ts](https://youtube.com/@titashshil?si=RgamNu1dc9jVIbJN) 👍😄🤝

### Run the following Commands in CloudShell
```
export USER2=
export REGION=
export BUCKET_NAME=
export TOPIC=
export FUNCTION_NAME=
```
```
gsutil mb gs://$BUCKET_NAME

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member=user:$USER2 --role=roles/storage.objectViewer

gcloud pubsub topics create $TOPIC


mkdir Qwiklab_explorers_ts
cd Qwiklab_explorers_ts


cat > index.js <<'EOF'
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC_ID";  // Replace this with your actual topic ID
  const pubsub = new PubSub();

  if (fileName.search("64x64_thumbnail") === -1) {
    // Doesn't have a thumbnail, get the filename extension
    const filenameSplit = fileName.split('.');
    const filenameExt = filenameSplit.pop();
    const filenameWithoutExt = filenameSplit.join('.');

    if (filenameExt.toLowerCase() === 'png' || filenameExt.toLowerCase() === 'jpg') {
      // Only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      const newFilename = `${filenameWithoutExt}${size}_thumbnail.${filenameExt}`;
      const gcsNewObject = bucket.file(newFilename);
      const srcStream = gcsObject.createReadStream();
      const dstStream = gcsNewObject.createWriteStream();
      const resize = imagemagick().resize(size).quality(90);

      srcStream.pipe(resize).pipe(dstStream);

      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
            // Set the content-type
            gcsNewObject.setMetadata({
              contentType: 'image/' + filenameExt.toLowerCase()
            }, (err, apiResponse) => {});

            pubsub
              .topic(topicName)
              .publish(Buffer.from(newFilename))
              .then(messageId => {
                console.log(`Message ${messageId} published.`);
                resolve();
              })
              .catch(err => {
                console.error('ERROR:', err);
                reject(err);
              });
          });
      });
    } else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
EOF

sed -i "s/REPLACE_WITH_YOUR_TOPIC_ID/$TOPIC/" index.js


cat > package.json <<'EOF_CP'
{
    "name": "thumbnails",
    "version": "1.0.0",
    "description": "Create Thumbnail of uploaded image",
    "scripts": {
      "start": "node index.js"
    },
    "dependencies": {
      "@google-cloud/pubsub": "^2.0.0",
      "@google-cloud/storage": "^5.0.0",
      "fast-crc32c": "1.0.4",
      "imagemagick-stream": "4.1.1"
    },
    "devDependencies": {},
    "engines": {
      "node": ">=4.3.2"
    }
  }
EOF_CP



export PROJECT_NUMBER=$(gcloud projects describe $DEVSHELL_PROJECT_ID --format="value(projectNumber)")
SERVICE_ACCOUNT=$(gsutil kms serviceaccount -p $PROJECT_NUMBER)

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT --role roles/artifactregistry.reader

sleep 45



#!/bin/bash

# Loop until the function deployment succeeds
while true; do
    gcloud functions deploy $FUNCTION_NAME --gen2 --runtime nodejs20 --entry-point thumbnail --source . --region $REGION --trigger-http --timeout 600s --max-instances 2 --min-instances 1 --quiet

    # Check the exit status of the last command
    if [ $? -eq 0 ]; then
        echo "Deployment successful, subscribe to techcps"
        break
    else
        echo "Deployment failed. Retrying..."
        sleep 17
    fi
done

# gcloud functions deploy $FUNCTION_NAME --region=$REGION --runtime=nodejs20 --entry-point=thumbnail --trigger-bucket=$BUCKET_NAME --source=.



curl -LO raw.githubusercontent.com/Techcps/ARC/master/Monitor%20and%20Manage%20Google%20Cloud%20Resources%3A%20Challenge%20Lab/travel.jpg

gsutil cp travel.jpg gs://$BUCKET_NAME


cat > app-engine-error-percent-policy.json <<EOF_CP
{
    "displayName": "Active Cloud Function Instances",
    "userLabels": {},
    "conditions": [
      {
        "displayName": "Cloud Function - Active instances",
        "conditionThreshold": {
          "filter": "resource.type = \"cloud_function\" AND metric.type = \"cloudfunctions.googleapis.com/function/active_instances\"",
          "aggregations": [
            {
              "alignmentPeriod": "300s",
              "crossSeriesReducer": "REDUCE_NONE",
              "perSeriesAligner": "ALIGN_MEAN"
            }
          ],
          "comparison": "COMPARISON_GT",
          "duration": "0s",
          "trigger": {
            "count": 1
          },
          "thresholdValue": 1
        }
      }
    ],
    "alertStrategy": {
      "autoClose": "604800s"
    },
    "combiner": "OR",
    "enabled": true,
    "notificationChannels": [],
    "severity": "SEVERITY_UNSPECIFIED"
  }
EOF_CP


gcloud alpha monitoring policies create --policy-from-file="app-engine-error-percent-policy.json"
```

# Congratulations ..!!🎉  You completed the lab shortly..😃💯

# *Well done..!* 👏

# Thank you for visiting.... :) 🗯️

# [Qwiklab_Explorers_ts](https://youtube.com/@titashshil?si=RgamNu1dc9jVIbJN)
