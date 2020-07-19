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
  * Initialise google cloud with the right account and project setup
    ```
    gcloud init
    ```
  * deploy the application to google cloud app engine standard

    ```
    gcloud app deploy
    ```

The application should be live and access the project app instance url ex: https://projxxxxx.el.r.appspot.com

### Enable APIs and Create New Service account
* Go to APIs and Services and click Enable APIs and Services and enable 
```
Cloud SQL API 
Cloud SQL Admin API
```
* Go to IAM & Admin >> Service accounts and click Create service account
  * Step 1: Enter Service Account Name
  * Step 2: Provide necessary details and click create.
  * Step 3: Create the key once the account is created. Choose Keytype as JSON
  
### Install Cloud SQL Proxy in Local Machine
* Follow the instructions to install the [Cloud SQL proxy client on your local machine](https://cloud.google.com/sql/docs/mysql/connect-external-app#install). The Cloud SQL proxy is used to connect to your Cloud SQL instance when running locally.
* Start the Cloud SQL proxy and replace CLOUDSQL_CONNECTION_NAME with the connection name of your Cloud SQL Instance.
* You can get the connection name from your Cloud SQL Dashboard.
![Image of Cloud SQL Connection Instance](https://github.com/vicky1408/aktonapi/blob/master/CloudSQL.png)
* Run the below command
```
./cloud_sql_proxy -instances=CLOUDSQL_CONNECTION_NAME=tcp:3306
```
  * Somtimes, the above command may throw error, if you have a MySQL Instance running already. In that case, stop the local MySQL as per the below,
    * Error : listen tcp 127.0.0.1:3306: bind: address already in use
    ```
    sudo netstat -nlpt |grep 3306
    sudo service mysql stop
    ```
* Once the connection is established, you may see something similar to below:
```
Listening on 127.0.0.1:3306 for CLOUDSQL_CONNECTION_NAME
Ready for new connections
```
* Now edit your .env file and update the database details.
```
DB_DATABASE=CLOUD_SQL_DATABASE_NAME
DB_USERNAME=CLOUD_SQL_USERNAME
DB_PASSWORD=CLOUD_SQL_PASSWORD
```
* Important: Open a new command prompt and navigate to your Laravel’s root directory and run the database migrations for Laravel.
```
php artisan migrate --force
```
* If the migrations are completed successfully, you will see the output similar to the one below.
```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
```

Now you can deploy your Laravel application to App Engine.

### Deploy Application with CloudSQL Connection
* Edit your app.yaml and update the Cloud SQL details.
```
runtime: php72

env_variables:
   APP_KEY: YOUR_APP_KEY
   APP_STORAGE: /tmp
   CACHE_DRIVER: database
   SESSION_DRIVER: database
   DB_DATABASE: CLOUD_SQL_DATABASE_NAME
   DB_USERNAME: CLOUD_SQL_USERNAME
   DB_PASSWORD: CLOUD_SQL_PASSWORD
   DB_SOCKET: "/cloudsql/CLOUDSQL_CONNECTION_NAME"
```
* Once done, run the below command to deploy the app on App Engine.
```
gcloud app deploy
```
* Check the Project URL to fetch API details from DB. ex: https://projxxxxx.el.r.appspot.com/api/v1/users
