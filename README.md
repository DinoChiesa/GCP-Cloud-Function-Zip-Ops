# Zip Operations function

This example uses a JavaScript function that relies on nodejs  and the
[@google-cloud/functions-framework](https://www.npmjs.com/package/@google-cloud/functions-framework)
to produce a function that unzips a zipfile, and sends back a JSON
representation of the contents of the zipfile.

## How it works

[GCP Cloud Functions](https://cloud.google.com/functions) is an example of a
"Functions as a Service" or FaaS offering. The idea is that instead of writing a
full application, with code dealing with concerns for startup, shutdown,
initialization, remoting, and so on...., developers can just write a simple
stateless function. This simplifies development, which can help small or large
companies build more quickly.


This repo holds a particular function example, which accepts ZIP files and
processes them. This function has a single handler that accepts as input a
base64-encoded ZIP file, then

- decodes it

- unzips it

- and creates a JSON response containing an array of entries, one for each
  compressed file in the zip.  the content of the "unzipped" files is
  base64-encoded, unless the filetype is a known text file (like .txt, .js, and
  so on), in which case the content is plain text.

For an input zipfile with two entries, one called package.json and one called index.js, the output will look like:

```
{
  "result": [
    {
      "name": "package.json",
      "stamp": "2022-11-02T00:48:20.000Z",
      "contents": "{\n  \"name\": \"zip-ops\",\n  \"version\": \"1.0.0\",\n  ... \n}\n"
    },
    {
      "name": "index.js",
      "stamp": "2022-11-02T00:47:56.000Z",
      "contents": "// index.js\n// ------------------------------------------------------------------\n//\n/* jshint esversion:9, node:true, strict:implied */\n/* global process, console, Buffer */\n\nconst functions = require('@google-cloud/functions-framework');\nconst AdmZip   = require('adm-zip');\n\nconst base64Decode = (encoded) => Buffer.from(encoded.trim(), 'base64');\n\n... \n"
    }
  ]
}
```

If you want to connect with this function from Apigee/App integration, you must
configure the Integration flow to send the appropriate HTTP request in to this
function, and then you must configure the Integration flow to handle a response
from the function that follows the form shown above.

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

NOTE: For production uses, you will want to convert this function to authenticated access. (remove `--allow-unauthenticated`).

## To inquire the endpoint uri after deployment

If you have previously deployed and don't remember the URI, you can inquire it.
```
  gcloud functions describe $FUNCNAME \
    --gen2 \
    --region us-west1 \
    --format="value(serviceConfig.uri)"
```

## To delete the cloud function

To avoid costs, clean up when you're finished testing.

```
  gcloud functions delete $FUNCNAME \
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
b64encoded zip. It might look like this:

```
{"contents" : "UEsDBBQAA..."}
```


## Limits and Security Concerns

This example has no checks on the size of the zipfile or the number of
entries. It reads in the entire ZIP file into memory and de-compresses it. This
may take some time for large zip files and will consume resources.

A better, more resilient design would insert checks and change behavior for
large zip files, or large zip entries.


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
