# API Management - Hands-on Lab Script - part 0

- Part 0 - Deploy your own Colours web/api (Optional) (You are here)
- [Part 1 - Create an API Management instance](apimanagement-1.md) 
- [Part 2 - Developer Portal and Product Management](apimanagement-2.md)
- [Part 3 - Adding API's](apimanagement-3.md)
- [Part 4 - Caching and Policy Expressions](apimanagement-4.md)
- [Part 5 - Versioning and Revisions](apimanagement-5.md)
- [Part 6 - Analytics and Monitoring](apimanagement-6.md)
- [Part 7 - Security](apimanagement-7.md)
- [Part 8 - DevOps](apimanagement-8.md)


## Provision your own instance of ColorWeb/ColorAPI

Some of the demos use the ColorWeb web application and the ColorAPI api application. In this lab we will show you how to deploy your own instances of the Color Web and Color API.

![](Images/APIMColorWebUnlimited.png)

The code for the ColorWeb / ColorAPI applications is available here:

- [ColorWeb](https://github.com/markharrison/ColoursWeb)
- [ColorApi](https://github.com/markharrison/ColoursAPI)

Docker Containers exist for these applications and so provides an easy deployment option ( IMPORTANT : due to the new pull restrictions on Docker images, in this lab we will be using the github registry):

- Github
  - docker pull ghcr.io/markharrison/coloursapi:latest
  - docker pull ghcr.io/markharrison/coloursweb:latest
- DockerHub
  - docker pull markharrison/colorweb:latest
  - docker pull markharrison/colorapi:latest

With the container we can deploy to multiple hosting options : VM's, App Services, ACI and also AKS. In this lab we are going to show you how to do it with [Azure Container Instances](https://docs.microsoft.com/en-us/azure/container-instances/).

# Deploying Web and API containers with Azure Container Instances

1. Login to Azure Portal at http://portal.azure.com.
2. Open the Azure Cloud Shell and choose Bash Shell (do not choose Powershell)

   ![Azure Cloud Shell](Images/img-cloud-shell.png "Azure Cloud Shell")

3. The first time Cloud Shell is started will require you to create a storage account. 
4. We proceed to create a unique identifier suffix for resources created in this Lab:

    ```bash
    APIMLAB_UNIQUE_SUFFIX=$USER$RANDOM
    # Remove Underscores and Dashes
    APIMLAB_UNIQUE_SUFFIX="${APIMLAB_UNIQUE_SUFFIX//_}"
    APIMLAB_UNIQUE_SUFFIX="${APIMLAB_UNIQUE_SUFFIX//-}"
    # Check Unique Suffix Value (Should be No Underscores or Dashes)
    echo $APIMLAB_UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export APIMLAB_UNIQUE_SUFFIX=$APIMLAB_UNIQUE_SUFFIX >> ~/.bashrc
    ```

5. Now we proceed to create a resource group for our ACI objects:


    ```bash
    #we define some variables first
    APIMLAB_RGNAME=myColorsAppRg-$APIMLAB_UNIQUE_SUFFIX
    APIMLAB_LOCATION=eastus
    # Persist for Later Sessions in Case of Timeout
    echo export APIMLAB_RGNAME=$APIMLAB_RGNAME >> ~/.bashrc
    echo export APIMLAB_LOCATION=$APIMLAB_LOCATION >> ~/.bashrc

    #we create the resource group
    az group create --name $APIMLAB_RGNAME --location $APIMLAB_LOCATION
    ```

6.  Now we create our ACI and specify our colors-web github container
 
    ```bash
    #we define some variables first
    APIMLAB_COLORS_WEB=mycolorsweb-$APIMLAB_UNIQUE_SUFFIX
    APIMLAB_IMAGE_WEB=ghcr.io/markharrison/coloursweb:latest
    APIMLAB_DNSLABEL_WEB=acicolorweb$APIMLAB_UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export APIMLAB_COLORS_WEB=$APIMLAB_COLORS_WEB >> ~/.bashrc
    echo export APIMLAB_IMAGE_WEB=$APIMLAB_IMAGE_WEB >> ~/.bashrc
    echo export APIMLAB_DNSLABEL_WEB=$APIMLAB_DNSLABEL_WEB >> ~/.bashrc


    #we create the container instance for the colors web
    az container create --resource-group $APIMLAB_RGNAME --name $APIMLAB_COLORS_WEB --image $APIMLAB_IMAGE_WEB --dns-name-label $APIMLAB_DNSLABEL_WEB --ports 80 --restart-policy OnFailure --no-wait
    ```

7.   Now we run the following command to check the status of the deployment and get the FQDN to access the app:

      ```bash
      #we check the status
      az container show --resource-group $APIMLAB_RGNAME --name $APIMLAB_COLORS_WEB --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
      ```

      The output should something like this:

      ```
      FQDN                                                  ProvisioningState
      ----------------------------------------------------  -------------------
      aci-color-web-fernando22287.eastus.azurecontainer.io  Succeeded
      ```

      Once we have a "Succeeded" message we proceed to navigate to the FQDN. And we should see our home page for our Colors Web:

      ![](Images/APIMACICOLORWEB.png)

8.  Now we proceed to create the ACI for the colors-api github container:

    ```bash
    #we define some variables first
    APIMLAB_COLORS_API=mycolorsapi-$APIMLAB_UNIQUE_SUFFIX
    APIMLAB_IMAGE_API=ghcr.io/markharrison/coloursapi:latest
    APIMLAB_DNSLABEL_API=aci-color-api-$APIMLAB_UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export APIMLAB_COLORS_WEB=$APIMLAB_COLORS_WEB >> ~/.bashrc
    echo export APIMLAB_IMAGE_WEB=$APIMLAB_IMAGE_WEB >> ~/.bashrc
    echo export APIMLAB_DNSLABEL_WEB=$APIMLAB_DNSLABEL_WEB >> ~/.bashrc


    #we create the container instance for the colors api
    az container create --resource-group $APIMLAB_RGNAME --name $APIMLAB_COLORS_API --image $APIMLAB_IMAGE_API --dns-name-label $APIMLAB_DNSLABEL_API --ports 80 --restart-policy OnFailure --no-wait
    ```

9.  Now we run the following command to check the status of the deployment and get the FQDN to access the app:

    ```bash
    #we check the status
    az container show --resource-group $APIMLAB_RGNAME --name $APIMLAB_COLORS_API --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
    ```

    The output should something like this:

    ```
    FQDN                                                  ProvisioningState
    ----------------------------------------------------  -------------------
    aci-color-api-fernando22287.eastus.azurecontainer.io  Succeeded
    ```

    Once we have a "Succeeded" message we proceed to navigate to the FQDN. And we should see our home page for our Colors Web:

    ![](Images/APIMACICOLORAPI.png)

---
[Home](README.md) | [Next](apimanagement-1.md)
