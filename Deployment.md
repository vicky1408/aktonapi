# Deployment to Google Cloud platform

Below are the steps covered as part of deployment to Google Cloud platform from ubuntu local.
* API - Lumen
* DB - MySQL

## Prerequisites
* Create a project in the [Google Cloud Platform Console](https://console.cloud.google.com/project)
* Enable billing for your project.
* Install and initialize the [Google Cloud SDK.](https://cloud.google.com/sdk/)
* Create a [Cloud SQL Second Generation Instance](https://www.cloudbooklet.com/create-cloud-sql-instance-and-connect-to-vm-instance-in-gcp/#second-generation)
* [Composer](https://getcomposer.org/download/) installed on your local computer.

Once the API is setup, check whether it is running fine in your local with the below command.
```
php -S localhost:8000 -t public
```

Now, we are going to migrate one at a time,
* Lumen API to be deployed in App Engine Standard
* Database migration in MySQL from Lumen to Cloud SQL (MySQL)

## Lumen API deployment to Google App Engine Standard
* Create App.yaml file and mention the details as below
```
runtime: php72

env_variables:
  ## Put production environment variables here.
  APP_KEY: [Just a random 32 character string to be provided here.]
  APP_STORAGE: /tmp
  VIEW_COMPILED_PATH: /tmp
```
* Edit the bootstrap/app.php and add the below code before the return statement.
```
$app->useStoragePath(env('APP_STORAGE', base_path() . '/storage'));
```
* Now remove the laravel-dump-server repository to prevent the Laravel’s caching error.
```
composer remove --dev beyondcode/laravel-dump-server
```
Now your application is ready to be deployed to App Engine.

* Once done, run the below command to deploy the app on App Engine.
** initialise google cloud with the right account and project setup
```
gcloud init
```
** deploy the application to google cloud app engine standard, the 

```
gcloud app deploy
```

### Enable APIs and Create New Service account
* 
