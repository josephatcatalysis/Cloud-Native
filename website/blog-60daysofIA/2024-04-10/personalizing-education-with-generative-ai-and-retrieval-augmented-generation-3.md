---
date: 2024-04-10T09:00
slug: personalizing-education-with-generative-ai-and-retrieval-augmented-generation-3
title: "7.3 Personalizing Education with Generative AI and Retrieval Augmented Generation Part 3"
authors: [cnteam]
draft: false
hide_table_of_contents: false
toc_min_heading_level: 2
toc_max_heading_level: 3
keywords: [Cloud, Data, AI, AI/ML, intelligent apps, cloud-native, 60-days, enterprise apps, digital experiences, app modernization, serverless, ai apps]
image: https://github.com/Azure/Cloud-Native/blob/main/website/static/img/ogImage.png
description: "In this three-part series, you’ll use Azure Container Apps, Azure OpenAI Service, and Retrieval Augmented Generation to create a personal tutor chatbot that dynamically adjusts educational materials and quizzes based on user interactions. This final article demonstrates how to deploy using Azure Container Apps." 
tags: [Build-Intelligent-Apps, 60-days-of-IA, learn-live, hack-together, community-buzz, ask-the-expert, azure-kubernetes-service, azure-functions, azure-openai, azure-container-apps, azure-cosmos-db, github-copilot, github-codespaces, github-actions]
---

<head> 
  <meta property="og:url" content="https://azure.github.io/cloud-native/60daysofia/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-3"/>
  <meta property="og:type" content="website"/> 
  <meta property="og:title" content="Build Intelligent Apps | AI Apps on Azure"/> 
  <meta property="og:description" content="Join us on a learning journey to build intelligent apps on Azure. Read all about the upcoming #BuildIntelligentApps initiative on this post!"/> 
  <meta property="og:image" content="https://github.com/Azure/Cloud-Native/blob/main/website/static/img/ogImage.png"/> 
  <meta name="twitter:url" content="https://azure.github.io/Cloud-Native/60daysofIA/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-3" /> 
  <meta name="twitter:title" content="Build Intelligent Apps | AI Apps on Azure" />
 <meta name="twitter:description" content="Azure and platform engineering pave the way for the efficient development, deployment, and maintenance of Intelligent Apps, triumphing over traditional approaches." />
  <meta name="twitter:image" content="https://azure.github.io/Cloud-Native/img/ogImage.png" /> 
  <meta name="twitter:card" content="summary_large_image" /> 
  <meta name="twitter:creator" content="@devanshidiaries" /> 
  <link rel="canonical" href="https://azure.github.io/Cloud-Native/60daysofIA/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-3" /> 
</head> 

<!-- End METADATA -->

![Graphic with a chat bubble-meets-robot head in the top right corner. At the bottom of the graphic is text that reads, "Personalizing Education with Generative AI and Retrieval Augmented Generation: Creating the Chatbot."](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-1.jpeg)

## Personalizing Education with Generative AI and Retrieval Augmented Generation Part 3: Deploying a Web Interface

Welcome to the final installment of our three-part tutorial series on building a personalized Python tutor with Generative AI and Retrieval Augmented Generation (RAG)! If you’re new to this series, be sure to check out [Part 1](https://azure.github.io/Cloud-Native/60DaysOfIA/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-1), which shows you how to set up the essential Azure resources, and [Part 2](https://azure.github.io/Cloud-Native/60DaysOfIA/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-2), which explains how to build out the chatbot’s core functionality.

In this final tutorial, you’ll take your chatbot from the development environment to a live web application where anyone can interact with it.

### Prerequisites

To follow this tutorial, ensure you have the following:

* The Azure services set up in Part 1
* The [Python web app](https://github.com/contentlab-io/Personalizing-Education-with-Generative-AI-and-RAG/blob/main/main.py) built in Part 2
* An active Azure subscription
* The [Azure command-line interface (CLI)](https://learn.microsoft.com/cli/azure/?ocid=buildia24_60days_blogs) installed
* [Docker](https://www.docker.com/get-started) installed

To preview the final application, take a look at the [complete project code](https://github.com/contentlab-io/Personalizing-Education-with-Generative-AI-and-RAG).

:::info
Register for **[Episode 4](https://aka.ms/serverless-learn-live/ep4?ocid=buildia24_60days_blogs)** of the new hands-on live learning series with an SME on **Intelligent Apps with Serverless on Azure**.
:::

### Deploy the Web Interface for the Educational Chatbot

With the chatbot’s intelligence and functionality in place, it’s time to make it accessible to the world. You’ll use [Azure Container Apps](https://azure.microsoft.com/products/container-apps?ocid=buildia24_60days_blogs) for a smooth and scalable deployment.

Azure Container Apps is a serverless platform designed to streamline the deployment and management of containerized applications. It handles infrastructure complexities for you, letting you focus on your application’s code.

Containerization packages your chatbot’s code, dependencies, and runtime into a self-contained image. This means it will run consistently across different environments. Since it’s lightweight, you can scale up or down based on demand.

#### Creating an Azure Container Registry

[Azure Container Registry](https://azure.microsoft.com/products/container-registry?ocid=buildia24_60days_blogs) (ACR) is your private storage space for container images. To get started with ACR, use the Azure CLI to log in to your Azure account by running:

```
az login
```

After authenticating, run the following, ensuring you replace `personaltutor` with your chosen registry name:

```
az acr login --name personaltutor
```

#### Building the Container Image

Next, create a file named `Dockerfile` in your project’s root directory. This file provides Docker with instructions for building your image:

```
FROM python:3.11-slim-bullseye 

WORKDIR /app

# Install Streamlit and other dependencies
COPY requirements.txt ./
RUN pip install -r requirements.txt

# Copy your application files
COPY . ./

# Expose the port used by Streamlit
EXPOSE 8501

# Command to start your app
CMD streamlit run main.py
```

Execute the following command to build the image, making sure to use your registry name:

```
az acr build --registry personaltutor --image python-tutor:latest --file Dockerfile .
```

This command instructs Azure to build a container image named `python-tutor:latest`, using your Dockerfile and the code in the current directory (indicated by the period). The image is then stored in your ACR.

Once you push your image to ACR, open the Azure portal and navigate to your container registry. Enable admin access by selecting **Access keys** under **Settings**, and then clicking **Enable** under **Admin** user.

Alternatively, run the following command from the terminal to enable admin access (again, updating `personaltutor` to reflect your selected registry name):

```
az acr update -n personaltutor --admin-enabled true
```

#### Creating an Environment

An [Azure Container Apps environment](https://learn.microsoft.com/azure/container-apps/environment?ocid=buildia24_60days_blogs) acts as a logical boundary for your apps. Think of it as the neighborhood where your chatbot will live.

To create a container app environment, run:

```
az containerapp env create \
   --name python-tutor-app-env \
   --resource-group personal-tutor \
   --location eastus
```

This command creates an environment named `python-tutor-app-env` within your existing resource group, located in the `eastus` region.

#### Creating the Container App

Now, you can deploy your application using Azure Container Apps!

Sign in to the Azure portal and search for “Azure Container Apps” in the search bar at the top. Select the service. Then, click **+ Create** to start a new container app.

![The Container Apps page in the Azure portal.](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-2.png)

On the **Basics** tab, configure the settings as follows:

* **Subscription** and **Resource group** — For consistency, ensure these match the resources you set in the first two parts of this series.
* **Container app name** — Choose a unique name, such as “personaltutor.”
* **Region** — Select the same region where you created your environment.
* **Container Apps Environment** — Select the environment you created earlier.

![Screenshot of the page to create the Container App. It has five tabs: Basics, Container, Bindings, Tags, and Review + create. Basics is open.  The page has two sections: Project details (Subscription, Resource group, and Container app name) and Container Apps Environment (Region and Container Apps Environment). At the bottom are two buttons: Review + Create and Next: Container.](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-3.png)

Click **Next: Container >** to proceed.

On the **Container** tab, configure the settings as follows:

* Uncheck **Use quickstart image**, as you’ll use your custom image.
* **Image Source** — Select **Azure Container Registry**.
* **Registry** — Choose the ACR you created previously.
* **Image** — Select the **python-tutor:latest** image you built.
* **Tag** — Keep the default **latest**.

Then, click **Environment Variables** and add the following, ensuring you replace the placeholders with your actual values:

* AOAIEndpoint
* AOAIKey
* AOAIDeploymentId
* SearchEndpoint
* SearchKey
* SearchIndex

Your chatbot needs these variables to connect to and interact with your Azure OpenAI and AI Search services. Review the section [“Retrieving Environment Variables”](https://azure.github.io/Cloud-Native/60DaysOfIA/personalizing-education-with-generative-ai-and-retrieval-augmented-generation-2#retrieving-api-keys-and-endpoints) in Part 2 of this series if you need a refresher on how to retrieve these.

![The Create Container App page lists the container details (Name, Image source, Registry, Image, Image tag, and Command override), the Container resource allocation (Workload profile and CPU and memory), and Environment variables. At the bottom are three buttons: Review + create, Previous, and Next: Bindings).](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-4.png)

Click **Next: Bindings >**. Leave the **Bindings** section with the default settings and click **Next: Ingress >**.

Configure as follows:

* **Ingress** — Select **Enabled** to make your app publicly accessible.
* **Ingress traffic** — Choose **Accepting traffic from anywhere**.
* **Ingress type** — Keep HTTP.
* **Target port** — Enter “8501” (assuming your Dockerfile specifies this).
* **Session affinity** — Select **Enabled**. This helps maintain user sessions.

![The Create Container App page lists fields for Ingress, Ingress traffic, Ingress type, selecting a Client certificate note, Transport, a checkbox for accepting Insecure connections, the option to enter a Target port, and a checkbox to enable Session affinity. At the bottom are three buttons: Review + create, Previous, and Next: Tags.](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-5.png)

Click **Review + create**.

Double-check your settings. If everything is correct, click **Create**.

![Screenshot of the Create Container App page. Here, you review the settings. At the bottom are two buttons: Create and Previous.](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-6.png)

Azure will now deploy your application as a container app. Once finished, you’ll see a URL in the **Overview** section (**Application Url**) where your app is live!

![The personaltutor app overview shows an application URL indicating that the Intelligent App has been successfully deployed.](../../static/img/60-days-of-ia/blogs/2024-04-10/7-3-7.png)

### Next Steps

Congratulations! Over this three-part series, you built an incredible AI-powered educational chatbot using Azure OpenAI, Azure AI Search, and Azure Container Apps for scalable deployment. You’ve seen how simple it is to build a dynamic, scalable, and high-impact Intelligent App with the help of Azure services.

Now that you have some boots-on-the-ground experience building an Intelligent App, test your knowledge by joining our **[Cloud Skills Challenge](https://aka.ms/intelligent-apps/apps-csc?ocid=buildia24_60days_blogs)**. And to keep the learning going, register for an upcoming demo bytes for **[Intelligent Apps with Azure Container Apps](https://developer.microsoft.com/reactor/series/S-1308/?wt.mc_id=blog_S-1308_webpage_reactor&ocid=buildia24_60days_blogs)** where the product engineering team gives a walkthrough on using open source vector databases and building a multi-LLM chat application.