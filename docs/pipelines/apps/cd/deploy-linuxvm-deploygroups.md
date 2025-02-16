---
title: Deploy nginx to a Linux VM (Classic)
description: Deploy a web application to an nginx web server on a Linux virtual machine using Deployment Groups in Azure Pipelines
ms.assetid: 9EBB0342-7FD2-473C-9809-9BCA2250CBC3
ms.topic: quickstart
ms.custom: seodec18
ms.author: ronai
author: RoopeshNair
ms.date: 09/07/2021
monikerRange: '<= azure-devops'
---

# Deploy a web app to an nginx web server on a Linux Virtual Machine

[!INCLUDE [version-lt-eq-azure-devops](../../../includes/version-lt-eq-azure-devops.md)]

> [!NOTE]
> If you want to deploy your application to a Linux virtual machine using YAML, see [Deploy to a Linux virtual machine](../../ecosystems/deploy-linux-vm.md).

Learn how to set up continuous deployment of your app to an nginx web server running on Ubuntu using
Azure Pipelines. You can use the steps in this
quickstart for any app as long as your continuous integration pipeline publishes a web deployment package.

After you commit and push a code change, it is automatically built and then deployed. The results will
automatically show up on your site.

## Define your CI build pipeline

You'll need a continuous integration (CI) build pipeline that publishes your web application, and
a deployment script that can be run locally on the Ubuntu server. Set up a CI build pipeline based on the runtime you want to use. 

#### [Java](#tab/java)

[!INCLUDE [include](../../ecosystems/includes/get-code-before-sample-repo-option-to-use-own-code.md)]

```
https://github.com/spring-guides/gs-spring-boot-docker.git
```
Follow more steps mentioned in [Build your Java app with Maven](../../ecosystems/java.md) for creating a build to deploy to Linux.

#### [JavaScript](#tab/javascript)

[!INCLUDE [include](../../ecosystems/includes/get-code-before-sample-repo-option-to-use-own-code.md)] 

```
https://github.com/MicrosoftDocs/pipelines-javascript-docker
```
Follow more steps mentioned in [Build your Node.js app with gulp](../../ecosystems/javascript.md) for creating a build to deploy to Linux.

* * * 

## Prerequisites for the Linux VM

The deployment scripts used in the above sample repositories have been tested on Ubuntu 16.04, and we recommend you use the same version of Linux VM for this quickstart.
Follow the extra steps described below based on the runtime stack used for the app.

#### [Java](#tab/java)

- For deploying Java Spring Boot and Spring Cloud based apps, create a Linux VM in Azure using [this](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) template, which provides a fully supported OpenJDK-based runtime.
- For deploying Java servlets on Tomcat server, create a Linux VM with Java 8 using [this](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) Azure template and [configure Tomcat 9.x as a service](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04#step-5-create-a-systemd-service-file).
- For deploying Java EE-based app, use an Azure template to create a [Linux VM + Java + WebSphere 9.x](https://azuremarketplace.microsoft.com/marketplace/apps/midvision.websphere-application-server-nde-90) or a [Linux VM + Java + WebLogic 12.x](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.20191009-arm-oraclelinux-wls-admin) or a [Linux VM +Java](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) + WildFly/JBoss 14 

#### [JavaScript](#tab/javascript)

To install a JavaScript app or a Node.js app, you'll need a Linux VM with Nginx web server to deploy the app.
If you don't already have a Linux VM with Nginx, create one now in Azure using the steps in
[this example](/azure/virtual-machines/linux/quick-create-cli).

* * * 

[!INCLUDE [create-linux-deployment-group](../includes/create-linux-deployment-group.md)]

## Define your CD release pipeline

Your CD release pipeline picks up the artifacts published by your CI build and then deploys them to your nginx servers.

1. Do one of the following to start creating a release pipeline:

   * If you've completed a CI build, in the build's **Summary** tab under **Deployments**,
     choose **Create release** followed by **Yes**. This starts a new release pipeline that's automatically linked to the build pipeline.

     ![Creating a new release pipeline from the build summary](../media/release-from-build-summary.png)

   * Open the **Releases** tab of **Azure Pipelines**, open the **+** drop down
     in the list of release pipelines, and choose **Create release pipeline**.

     ![Creating a new release pipeline in the Releases page](../media/release-from-release-page.png)

1. Choose **Start with an Empty job**.

1. If you created your new release pipeline from a build summary, check that the build pipeline and artifact
   is shown in the **Artifacts** section on the **Pipeline** tab. If you created a new release pipeline from
   the **Releases** tab, choose the **+ Add** link and select your build artifact.

   ![Checking or selecting the build pipeline and artifact](media/deploy-linuxvm-deploygroups/confirm-or-add-artifact.png)

1. Choose the **Continuous deployment** icon in the **Artifacts** section, check that the
   continuous deployment trigger is enabled, and add a filter that includes the **main** branch.

   ![Checking or setting the Continuous deployment trigger](media/deploy-linuxvm-deploygroups/confirm-or-set-cd-trigger.png)

   > Continuous deployment is not enabled by default when you create a new release pipeline from the **Releases** tab.

1. Open the **Tasks** tab, select the **Agent job**, and choose **Remove** to remove this job.

   ![Removing the Agent job from the pipeline](media/deploy-linuxvm-deploygroups/remove-agent-phase-image.png)

1. Choose **...** next to the **Stage 1** deployment pipeline and select **Add deployment group job**.

   ![Adding a Deployment group stage to the pipeline](media/deploy-linuxvm-deploygroups/add-deployment-group-phase.png)

1. For the **Deployment Group**, select the deployment group you created earlier such as **myNginx**.

   ![Selecting the deployment group](media/deploy-linuxvm-deploygroups/select-deployment-group.png)

    The tasks you add to this job will run on each of the machines in the deployment group you specified.

1. Choose **+** next to the **Deployment group job** and, in the task catalog, search for and add a
   **Bash** task.

   ![Adding a Shell Script task to the pipeline](media/deploy-linuxvm-deploygroups/add-shellscript-task.png)

1. In the properties of the **Bash** task, use the **Browse** button for the **Script Path** to select
   the path to the **deploy.sh** script in the build artifact. For example, when you use the **nodejs-sample**
   repository to build your app, the location of the script is  
   `$(System.DefaultWorkingDirectory)/nodejs-sample/drop/deploy/deploy.sh`.
   
   ![Configuring the Shell Script task](media/deploy-linuxvm-deploygroups/configure-shellscript-task.png)
   
   See a [sample deploy.sh file](https://github.com/azure-devops/fabrikam-node/blob/master/deployscript.sh) for a Node.js web app.   

1. Save the release pipeline.

   ![Saving the newly created release pipeline](media/deploy-linuxvm-deploygroups/save-definition-image.png)

## Create a release to deploy your app

You're now ready to create a release, which means to start the process of running the release pipeline
with the artifacts produced by a specific build. This will result in deploying the build.

[!INCLUDE [simple-create-release](../includes/simple-create-release.md)]

## Next steps

* [Dynamically create and remove a deployment group](howto-webdeploy-iis-deploygroups.md#depgroup)
* [Apply stage-specific configurations](howto-webdeploy-iis-deploygroups.md#envirconfig)
* [Perform a safe rolling deployment](howto-webdeploy-iis-deploygroups.md#rolling)
* [Deploy a database with your app](howto-webdeploy-iis-deploygroups.md#database)
