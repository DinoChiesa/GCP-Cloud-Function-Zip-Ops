
To deploy:

  gcloud functions deploy zip-ops-http-function \
    --gen2 \
    --runtime=nodejs16 \
    --region=us-west1 \
    --source=. \
    --entry-point=zip-ops-handler \
    --trigger-http \
    --allow-unauthenticated

to inquire the uri:

  gcloud functions describe zip-ops-http-function \
    --gen2 \
    --region us-west1 \
    --format="value(serviceConfig.uri)"

to delete the cloud function:

  gcloud functions delete zip-ops-http-function \
    --gen2 \
    --region us-west1

to run an test it locally:

  npm run local

invoke with 0:8080

  curl -i 0:8080  -H content-type:text/plain -d @zip1.b64encoded
  curl -i 0:8080  -H content-type:application/json -d @zip1.json


to produce the encoded zip file used in the above:

  cat myzipfile.zip | openssl base64 -A > zip1.b64encoded

The json payload has one member, "contents", which contains the b64encoded zip. 

docs:
  https://cloud.google.com/functions/docs/running/function-frameworks

