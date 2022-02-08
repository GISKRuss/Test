# GeoServer

This repo contains dockerised setup files for GeoServer.

## Running the application

Clone the repo, make sure you can access it in an environment where both docker and docker-compose are already installed. 

To launch the application, run the following command
```
docker-compose up --build -d
```
with `build` flag to rebuild the whole containers images and `-d` to run them in detached mode i.e. in the background.


GeoServer will take some time to load the settings and run. Once it's ready, go to `http://localhost/geoserver` on the browser to view the application.

## Deploying changes to Azure web app

For any changes on the geoserver to take effect on the web app, the container image has to be updated and pushed to Azure container registry. Below are the steps to take:
1. Make necessary changes to the geoserver image on Dockerfile.geoserver and save the file.
2. Build the image locally using command "`docker build --file Dockerfile.geoserver --tag <local-image> .`" where 'local-image' can be any understandable value.
3. Test the local image by running `docker run -it -p 8000:8080 <local-image>`. Go to the browser and type in `http://localhost:8000/geoserver`. The image might take a while to build so keep on refreshing the page.
4. Log in to Azure container registry by running `docker login gisregistry.azurecr.io`. The credentials can be retrieved by running `az acr credential show --resource-group rg-uks-container --name gisregistry`.
5. Check the latest tag values on the registry by running `az acr repository show-tags --name gisregistry --repository geoserver --detail`. Choose next tag value appropriately based on next incremental value or specific need.
6. Once the local image is good to go, tag it for the registry with the new tag value by running `docker tag <local-image> gisregistry.azurecr.io/geoserver:<new-tag-value>`.
7. Push the image to the registry by running `docker push gisregistry.azurecr.io/geoserver:<new-tag-value>`. Wait for the image to be fully uploaded.
8. Update the compose file i.e. *docker-compose-azure.yml* to refer to the new image tag value. Save the file.
9. Push the container changes for the web app by running `az webapp config container set --resource-group rg-uks-test-geoserver --name geoserver-test --multicontainer-config-type compose --multicontainer-config-file docker-compose-azure.yml`. This will update *geoserver-test* web app so replace this with other app name if needed.
10. The same update can also be done by going to the Azure portal and from respective app service page. Click on Deployment Center link from Deployment section on the left sidebar. On Settings tab, update the image tag value with the new one. Click on Save to update the changes. The web app can be viewed from the URL link in Overview page if the deployment goes successful.

Detailed steps and information about overall Geoserver Azure architecture can be viewed here: https://wsplandservices.atlassian.net/wiki/spaces/GSSGIS/pages/2039873537/GIS+-+Azure+environments+setup