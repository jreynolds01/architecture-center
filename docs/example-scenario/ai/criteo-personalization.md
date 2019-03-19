---
title: Content Personalization on Azure
description: Use machine learning to personalize content with Azure Databricks.
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
---

# Click prediction on Azure

This example scenario shows how a business can use machine learning to automate content personalization for their customers. Azure Databricks is used to train a model that predicts the probability a user will click on an item. In turn, is then used to recommend new content that has a high probability of being clicked on.

Recommendations can be useful in various industries from retail to news and media. Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations. Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers. Today, we can  provide customized recommendations at scale by utilizing Azure to train models learn from the customer's prior behavior.

## Relevant use cases

Consider this scenario for the following use cases:

* Content recommendations on a website or in a mobile application.
* News recommendations on streaming media.

## Architecture

![Architecture of a machine learning model for training content personalization][architecture]

This scenario covers the training, evaluating, and serving of a machine learning model for content personalization using the [LightGBM][lightgbm] library on spark. Specifically, a classification algorithm is used to predict an anonymized dataset of clicks. This particular scenario covers only a very small portion of the steps required to integrate into an end-to-end workload. The broader context of this scenario includes the following:

1. A front-end web-site serves rapidly-changing content to its users. This website leverages cookies and user profiles to track user information that is useful in personalizing the content for that user. In addition to user profiles, the web-site has access to a separate data store containing information about each of the items it serves to each user.

2. The sets of distinct user- and item- data get preprocessed and joined into a single anonymized table that contains a mixture of numeric and categorical features that can be used to model the user-item interactions (clicks).

3. [MMLSpark][mmlspark] is used to setup the prerequisites for and estimate a [LightGBM][lightgbm] classifier on Azure Databricks to predict the probability of a click as a function of the numeric and categorical features created in step 2.

4. Azure Machine Learning is used to create a scoring service that provides access to the estimated model on Azure Kubernetes Service to predict the probability of a click for a small set of observations that map to items for a given user.

5. An external service sorts the probability of a click, maps the features used to make the prediction back to an item id, and provides the highest probability items to the front-end to serve.

This scenario uses pre-anonymized data available from [Criteo][criteo], and only covers steps 3 and 4. 

For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].

### Components

* [Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed. 

* [Azure Machine Learning service][mls] is used in this scenario to deploy the machine learning model.

* [Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].

## Considerations

### Availability

Machine-learning-built apps are split into two resource components: resources for training, and resources for serving. Resources required for training generally do not need high availability, as live production requests do not directly hit these resources. Resources required for serving need to have high availability to serve customer requests.

### Scalability

For training, if you have a large data size, you can scale your Azure Databricks cluster by adjusting either the number or the [VM size][vm-size] of your worker nodes.

### Security

This scenario can use Azure Active Directory to authenticate users to the Databricks workspace. Permissions can be managed via Azure Active Directory authentication or role-based access control.

## Deploy this scenario

**Prerequisites**: You must have an existing Azure account. 

All the code for this scenario is available in the [Microsoft Recommenders repository][github].

Follow these steps to run the [LightGBM on Spark quickstart notebook][notebook]:

1. [Create an Azure Databricks workspace][adbcreate] from the Azure portal.

2. Follow the [setup instructions][reco-setup] for installing the [Microsoft Recommmenders repository][github] on a cluster within your workspace. Be sure to include the `--mmlspark` option in the install script.

3. Import the [LightGBM Quickstart notebook][lightgbmnotebook] into your workspace. After logging into your Azure Databricks Workspace, do the following:
  a. Click **Home** on the left side of the workspace
  b. Right-click on white space in your home directory. Select **Import**.
  c. Select URL, and paste the following into the text field:
    ```
    https://github.com/Microsoft/Recommenders/blob/master/notebooks/00_quick_start/mmlspark_lightgbm_tinycriteo.ipynb
    ```
  d. Click **Import**
4. Click on the notebook to open it, attach the configured cluster, and execute the notebook.

## Related resources

For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch]. For additional tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[adbcreate]: /azure/azure-databricks/quickstart-create-databricks-workspace-portal
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[criteo]: https://labs.criteo.com/2014/02/download-dataset/
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[lightgbm]: https://github.com/Microsoft/LightGBM
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[mmlspark]: https://aka.ms/spark
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[reco-setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#setup-guide-for-azure-databricks
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
