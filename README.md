# OAuth Custom Rest Connector 
    This article shows how to create custom rest connector in webMethods.io to get access token without using the default oauth account management and how to configure keycloak server with client_credentials.

# Usecase
If you are integrating with Custom Cloud Rest Application and if the OAUTH Provider for your custom application generates token with considerably small expiry and there is a need to generate the access token for every session (or service execution) instead of refreshing token on expiry. In such cases we will need to create a Cloud Integration to get the access token instead of using default OAuth Account management of the connector.

![](./images/usecase.png)

As an example we will use Keycloak as OAUTH provider to showcase this usecase.

# Prerequisite
1. Pre-instaled keycloak server. *If you need to create your own instance, you could easily spawn one docker or k8s instance from "keycloak packaged by bitnami". Refer https://bitnami.com/stack/keycloak/helm*


# Topics
1. Creating realm and client in keycloak and configuring them.
2. Test to retrive token using postman
3. webmethods.IO
   1. Creating custom rest connector & and an account
   2. Creating reource & operations for getting token
   3. Creating FlowService to get the token

# Steps

## Creating and configuring realm and client

1. Login to keycloak "Administartion console".
![](./images/2023-01-02-17-50-30.png)

2. Login
![](./images/2023-01-02-18-52-19.png)

3. Create a realm
![](./images/2023-01-02-18-53-24.png)

4. Create a client
![](./images/2023-01-02-18-54-40.png)

![](./images/2023-01-02-18-56-00.png)

![](./images/2023-01-02-18-57-03.png)

**Note: Select "Use refresh tokens for client credentials grant" in *Open ID Connect Compatibility Modes***
![](./images/2023-01-02-18-59-04.png)

## Test to retrive token using postman

1.  Create a request with below config
![](./images/2023-01-02-20-09-05.png)

## webmethods.IO
### Creating custom rest connector
1. Create / Open your project in IO tenant 
![](./images/2023-01-02-20-13-41.png)

2. Navigate to Connectors > Rest and click on **Add Connector**. provide name, URL(http://<KEYCLOAK_SERVER:PORT/>) and select **Credentials** as Authentication Type and save
![](./images/2023-01-02-20-16-42.png)

3. Navigate to Connectors, select the created connector and click on **Add Account**. Select **Authorization Type** as **none** and rest could be left default.
![](./images/2023-01-02-21-29-14.png)

### Creating reource & operations for getting token
1. Click **Add Resource** with name and Path (realms/test_realm/protocol/openid-connect/token)
![](./images/2023-01-02-20-18-58.png)

2. Add Header **Content-Type** with value **application/x-www-form-urlencoded**
3. Add request body with Content-Type **Binary**
![](./images/2023-01-02-20-21-13.png)

4. Add response body for HTTP range **200-299** and copy the json from postman test results to create the document type.
5. Add another response body for HTTP range **400-599** for Error
![](./images/2023-01-02-21-04-04.png)
![](./images/2023-01-02-21-08-36.png)

### Creating FlowService to get the token
1. Navigate to Integrations > FlowServices and click **+** to add new Flowservice
![](./images/2023-01-02-21-21-41.png)

2. Add a Transform pipeline and add pipeline variable (say input) with value **grant_type=%grant_type%&scope=%scope%&client_id=%client_id%&client_secret=%client_secret%**. Replace the %% with respective values. You could also add all 4 **form parameters** into **REFERENCE DATA** and retrive from there instead of hardcoding.
![](./images/2023-01-02-21-13-13.png)

3. Add another transform pipeline to covert the above **input** to stream.
![](./images/2023-01-02-21-23-07.png)

4. Now invoke the operation we created and pass the above stream to the body
![](./images/2023-01-02-21-24-33.png)

5. Save & Test your FlowService
![](./images/2023-01-02-21-25-45.png)

# Downloads / Assets

You could find the export of Integration, along with Connector definitions in **assets/integration**.

## How to use/test

1. Login to wM.IO tenant
2. Navigate to project where you want to import
3. Import the zip from assets/integration
4. Add the reference data with name "OAUTH_APP_DEV" and upload the file "OAUTH_APP_DEV.txt" from **assets/reference data**. ***Note: Client Secret needs to be updated accordingly***​​​​​​​
5. Add an account with name "Oauth_None" and details as provided in markdown
