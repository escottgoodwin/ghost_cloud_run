# Ghost 3 with Image Offload to Google Cloud Storage

This is modified version of the [Docker "Official Image"](https://github.com/docker-library/official-images#what-are-official-images) that adds offloading image or document uploads to [Google Cloud Storage](https://cloud.google.com/storage).

The original inspiration was to allow Ghost to run in a serverless environment like [Google Cloud Run](https://cloud.google.com/run), which is unable to persist uploads. 

If you are using an M1 Mac, be sure to use a ARM Ghost Docker image. This Dockerfile does not work with M1 Macs. 

The [full image description on Docker Hub](https://hub.docker.com/_/ghost/) is generated/maintained over in [the docker-library/docs repository](https://github.com/docker-library/docs), specifically in [the `ghost` directory](https://github.com/docker-library/docs/tree/master/ghost).

## Build and deploy container

* Build 

```docker build -t CONTAINER-NAME .```

* Deploy to Google Container Registry

``` docker tag CONTAINERNAME grc.io/PROJECT-ID/CONTAINER-NAME```

# For Deployment to Cloud Run

UPDATE Cloud Run has now added a [Secrets Manager](https://cloud.google.com/run/docs/configuring/secrets) for Cloud Run services. It is best practice to use secrets for sensitive information like database configurations. 

You can expose the secret values through environment variables instead of setting the environment variables with the sensitive information. 

[See More](https://cloud.google.com/run/docs/configuring/secrets) about using secrets with the console, cli and yaml files. 

## With the Console

See this article for complete instructions using the Google console. 

## With the gcloud cli

To use the gcloud cli with the Google Cloud Build service. This assumes your Cloud Run service will have the same name as your container. The url will be a place holder url until you get the service url after the service successfully deploys. The plugin also allows you set a CDN address for your images so that all images in your Ghost posts will point to the CDN. 

* Build with Cloud Build (Optional)

```gcloud builds submit --tag gcr.io/PROJECT-ID/CONTAINER-NAME```

* Deploy to Cloud Run

```gcloud run deploy CONTAINER-NAME --image grc.io/PROJECT-ID/CONTAINER-NAME --allow-unauthenticated --update-env-vars database__client=mysql,database__connection__host=MYSQL_HOST,database__connection__port=MYSQL_PORT,database__connection__user=MYSQL_USER,database__connection__password=MYSQL_PASSWORD,database__connection__database=MYSQL_GHOST_DB_NAME,storage__active=gcs,storage__gcs__bucket=GCS_BUCKET_NAME,url='https://ghost.org/'```

Optional CDN environment variable ```storage__gcs__cdn=CDN_DOMAIN```

* Redeploy with the service url. 
```gcloud run services update CONTAINER-NAME --update-env-vars url=SERVICE_URL```

When you first visit the service url, you will likely get a 503 message while ghost initializes and creates the database. Refresh and the page will load properly. Visit `SERVICE_URL/ghost` to visit the admin screen and creates your admin user. 

## With Yaml File

Modify the environment variables in `service.yaml` file with your database and storage bucket. You can also add a cdn for your images by adding: 

    name: storage__gcs__cdn
    value: CDN_DOMAIN

Leave the placeholder url. 

* Create the service:

```gcloud beta run services replace service.yaml```

* Set url

Once the service deploys successfully, get the service url:

```gcloud run services describe CONTAINER-NAME --platform managed --region REGION --format 'value(status.url)```

Copy service url and update the url environment variable with the service url: 

```gcloud run services update CONTAINER-NAME --update-env-vars url=SERVICE_URL```

* Allow unauthenticated access

```gcloud run services set-iam-policy CONTAINER-NAME policy.yaml```

In order to allow unauthenticated access, your IAM permissions must contain the run.services.setIamPolicy permission. 

## After Deploying

When you first visit the service url, you will likely get a 503 message while ghost initializes and creates the database. Refresh and the page will load properly. Visit `SERVICE_URL/ghost` to visit the admin screen and creates your admin user. 