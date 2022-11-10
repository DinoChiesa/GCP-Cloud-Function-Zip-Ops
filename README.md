# Zip Operations function

This example uses nodejs and the @google-cloud/functions-framework to show how
to build a function that unzips a zipfile, and sends back a JSON representation
of the contents of the zipfile.

## How it works

This function has a single handler that accepts as input a base64-encoded ZIP file, then 

- decodes it

- unzips it

- and creates a JSON response containing an array of entries, one for each
  compressed file in the zip.  the content of the "unzipped" files is
  base64-encoded, unless the filetype is a known text file (like .txt, .js, and
  so on), in which case the content is plain text.

## To deploy:

Deploy the function into Google Cloud functions, this way: 

```
  FUNCNAME=zip-ops-http-function
  HANDLER=zip-ops-handler
  gcloud functions deploy $FUNCNAME \
    --gen2 \
    --runtime=nodejs16 \
    --region=us-west1 \
    --source=. \
    --entry-point=$HANDLER \
    --trigger-http \
    --allow-unauthenticated
```

The output of that command will show a URL, the endpoint at which the cloud function is reachable. 
You may wish to convert it to authenticated access. (remove `--allow-unauthenticated`). 

## To inquire the endpoint uri after deployment

If you have previously deployed and don't remember th eURI, you can check it. 
```
  gcloud functions describe  $FUNCNAME \
    --gen2 \
    --region us-west1 \
    --format="value(serviceConfig.uri)"
```

## To delete the cloud function

To avoid costs, clean up when you're finished testing.

```
  gcloud functions delete  $FUNCNAME \
    --gen2 \
    --region us-west1
```

## Testing locally

To run the service locally, you can use this command:
```
  npm run local
```

Then, invoke the local service with a text/plain content-type at 0:8080:
```
  curl -i 0:8080  -H content-type:text/plain -d @zip1.b64encoded
```

To produce the encoded zip file used in the above:

```
  cat anyzipfile.zip | openssl base64 -A > zip1.b64encoded
```

An alternative content-type is JSON: 
```
  curl -i 0:8080  -H content-type:application/json -d @zip1.json
```

In this case, the json payload has one member, "contents", which contains the
b64encoded zip.  IT might look like this:

```
{"contents" : "UEsDBBQAA..."}
```

## Other Information

Relevant docs fo Google Cloud functions
  https://cloud.google.com/functions/docs/running/function-frameworks

## Disclaimer

This example is not an official Google product, nor is it part of an
official Google product.

## Support

This callout is open-source software, and is not a supported part of the Google App Integration product. 

If you need assistance, you can try inquiring on [Google Cloud Community
forum dedicated to Apigee](https://www.googlecloudcommunity.com/gc/Apigee/bd-p/cloud-apigee).
There is no service-level guarantee for
responses to inquiries regarding this callout.

## License

This material is [Copyright 2022 Google LLC](./NOTICE).
and is licensed under the [Apache 2.0 License](LICENSE). 


## Author
dchiesa@google.com
