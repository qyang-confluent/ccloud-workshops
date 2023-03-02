
<div align="center" padding=25px>
    <img src="images/confluent.png" width=50% height=50%>
</div>

# <div align="center">Manage Confluent Cloud with CLI</div>
## <div align="center">Lab Guide</div>
<br>

## **Agenda**

1. [Download Confluent Cli](#step-1)
1. [Create a Standard Cluster](#step-2)
1. [Create a Service Account](#step-3)
1. [Create an API Key Pair](#step-4)
1. [RBAC - Create Role Binding for Service Account](#step-5)
1. [CLI - Create, Produce and Consume from Topic](#step-6)
1. [Clean Up Resources](#step-12)


## **Prerequisites**
<br>

1. Confluent Cloud Account
    - Sign-up for a Confluent Cloud account [here](https://www.confluent.io/confluent-cloud/tryfree/)
    - Once you have signed up and logged in, You are ready to create your cluster.

    
    > **Note:** You will create resources during this workshop that will incur costs. When you sign up for a Confluent Cloud account, you will get free credits to use in Confluent Cloud. This will cover the cost of resources created during the workshop. More details on the specifics can be found [here](https://www.confluent.io/confluent-cloud/tryfree/).



2. Ports 443 and 9092 need to be open to the public internet for outbound traffic. To check, try accessing the following from your web browser:
    - portquiz.net:443
    - portquiz.net:9092

3. This workshop requires access to a command line interface.
    * **Mac users:** The standard Terminal application or iTerm2 are recommended.
    * **Windows users:** The built-in Command Prompt or Git BASH are recommended.  




***

## **Objective:**
You already set up your Confluent Cloud account, including creating your first basic cluster and topic using Web UI, and setting up Schema Registry. Now, you will use Confluent CLI to set up a Standard cluster and create Service Account for your application development.

 
## <a name="step-1"></a>**Download and install Confluent CLI**
1. Download the Confluent CLI from [here](https://docs.confluent.io/confluent-cli/current/install.html#tarball-or-zip-installation), choose the package for your system. You can also use scripted installation if you are running it on Linux or Mac, here is command 
```
curl -sL --http1.1 https://cnfl.io/cli | sh -s -- latest
export PATH=$(pwd)/bin:$PATH
```

The confluent cli will be installed in ./bin directory, so you need to add it to your path.


2. Log in and enter your email and password.

```
confluent login --save
```

## <a name="step-2"></a>**Create a Cluster**

An environment contains clusters and its deployed components such as Connectors, ksqlDB, and Schema Registry. You have the ability to create different environments based on your company's requirements. Confluent has seen companies use environments to separate Development/Testing, Pre-Production, and Production clusters.

1. List all environments in your account and create a *Standard Cluster*. 

   > **Note:** Confluent Cloud clusters are available in 3 types: Basic, Standard, and Dedicated. Basic is intended for development use cases so you will use that for the workshop. Basic clusters only support single zone availability. Standard and Dedicated clusters are intended for production use and support Multi-zone deployments. If you are interested in learning more about the different types of clusters and their associated features and limits, refer to this [documentation](https://docs.confluent.io/current/cloud/clusters/cluster-types.html).


```
confluent env list
confluent use env-id
confluent kafka cluster list
confluent create kafka cluster
confleunt use kafka cluster
```



 
## <a name="step-3"></a>**Create Service Account**
```
confluent iam serviceaccount create
```

## <a name="step-4"></a>**Create an API Key Pair**
1. Now you will create a service account and API key pair.
```
confluent api-key create
confluent api-key use key --resource
```

## <a name="step-5"></a>**Service Account - Access Control with RBAC**
```
confluent iam rolebinding

```

## <a name="step-6"></a>**Create, Produce and Consume from a topic**
```
confluent kafka topic create
confluent kafka topic produce
confluent kafka topic consume --from-beginning
```

## <a name="step-12"></a>**Clean Up Resources**
```
confluent kafka cluster delete
confluent iam serviceaccount delete
```

Deleting the resources you created during this workshop will prevent you from incurring additional charges.


## <a name="step-13"></a>**Confluent Resources and Further Testing**

* [Confluent Cloud Documentation](https://docs.confluent.io/cloud/current/overview.html)

* [Confluent Connectors](https://www.confluent.io/hub/) - A recommended next step after the workshop is to deploy a connector of your choice.

* [Confluent Cloud Schema Registry](https://docs.confluent.io/cloud/current/client-apps/schemas-manage.html#)

* [Best Practices for Developing Apache Kafka Applications on Confluent Cloud](https://assets.confluent.io/m/14397e757459a58d/original/20200205-WP-Best_Practices_for_Developing_Apache_Kafka_Applications_on_Confluent_Cloud.pdf) 

* [Confluent Cloud Demos and Examples](https://docs.confluent.io/platform/current/tutorials/examples/ccloud/docs/ccloud-demos-overview.html)

* [Kafka Connect Deep Dive – Error Handling and Dead Letter Queues](https://www.confluent.io/blog/kafka-connect-deep-dive-error-handling-dead-letter-queues/)
