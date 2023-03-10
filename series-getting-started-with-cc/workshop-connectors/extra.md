
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
1. Download the Confluent CLI from [here](https://docs.confluent.io/confluent-cli/current/install.html#tarball-or-zip-installation), choose the package for your system. You can also use scripted installation.

You can also run the CLI in a docker container

```bash
docker run -it -u root --rm confluentinc/confluent-cli:latest bash
```

The confluent cli will be installed in ./bin directory, so you need to add it to your path.


2. Log in and enter your email and password.

```bash
confluent login --save
```

## <a name="step-2"></a>**Create a Cluster**

An environment contains clusters and its deployed components such as Connectors, ksqlDB, and Schema Registry. You have the ability to create different environments based on your company's requirements. Confluent has seen companies use environments to separate Development/Testing, Pre-Production, and Production clusters.

1. List all environments in your account and create a *Standard Cluster*. 

   > **Note:** Confluent Cloud clusters are available in 3 types: Basic, Standard, and Dedicated. Basic is intended for development use cases so you will use that for the workshop. Basic clusters only support single zone availability. Standard and Dedicated clusters are intended for production use and support Multi-zone deployments. If you are interested in learning more about the different types of clusters and their associated features and limits, refer to this [documentation](https://docs.confluent.io/current/cloud/clusters/cluster-types.html).


```sh
c6ebebbd8823:/# confluent env list
  Current |     ID     |  Name
----------+------------+----------
          | env-ymgzx6 | dev
  *       | env-r52q29 | default
  
c6ebebbd8823:/# confluent env use env-ymgzx6
Now using "env-ymgzx6" as the default (active) environment.

c6ebebbd8823:/# confluent kafka cluster list
None found.

c6ebebbd8823:/# confluent kafka cluster  create sandbox-east --type standard --cloud aws --region us-east-1
It may take up to 5 minutes for the Kafka cluster to be ready.
+----------------------+---------------------------------------------------------+
| Current              | false                                                   |
| ID                   | lkc-3r6vz2                                              |
| Name                 | sandbox-east                                            |
| Type                 | STANDARD                                                |
| Ingress Limit (MB/s) |                                                     250 |
| Egress Limit (MB/s)  |                                                     750 |
| Storage              | Infinite                                                |
| Provider             | aws                                                     |
| Region               | us-east-1                                               |
| Availability         | single-zone                                             |
| Status               | PROVISIONING                                            |
| Endpoint             | SASL_SSL://pkc-n00kk.us-east-1.aws.confluent.cloud:9092 |
| REST Endpoint        | https://pkc-n00kk.us-east-1.aws.confluent.cloud:443     |
+----------------------+---------------------------------------------------------+

c6ebebbd8823:/# confluent kafka cluster list
  Current |     ID     |     Name     |   Type   | Provider |  Region   | Availability | Status
----------+------------+--------------+----------+----------+-----------+--------------+---------
          | lkc-3r6vz2 | sandbox-east | STANDARD | aws      | us-east-1 | single-zone  | UP

c6ebebbd8823:/# confluent kafka cluster use lkc-3r6vz2
Set Kafka cluster "lkc-3r6vz2" as the active cluster for environment "env-ymgzx6".
```



 
## <a name="step-3"></a>**Create Service Account**
1. Now we are going to create a service account. The service account will be used to run the application.

```bash
c6ebebbd8823:/# confluent iam  service-account create app-sa --description "sa for demo app"
+-------------+-----------------+
| ID          | sa-pjj8o2       |
| Name        | app-sa          |
| Description | sa for demo app |
+-------------+-----------------+
```


## <a name="step-4"></a>**Create an API Key Pair**
1. Now you will create a service account and API key pair.
```bash
c6ebebbd8823:/# confluent  api-key create --service-account sa-pjj8o2 --resource lkc-3r6vz2
It may take a couple of minutes for the API key to be ready.
Save the API key and secret. The secret is not retrievable later.
```

## <a name="step-5"></a>**Service Account - Access Control with RBAC**
1. We created service account and API key pair with it, but we didn't grant any permission to the server account. The SA account is not able to access anything yet. We will need to grant the permission using Confleunt Cloud RBAC. Keep in mind RBAC is only available for Standard and Dedicated cluster type. This is the reason why we need create a standard cluster for this part.

```bash
c6ebebbd8823:/# confluent iam rbac role-binding create  --principal User:sa-pjj8o2 --role CloudClusterAdmin --cloud-cluster lkc-3r6vz2 --environment env-ymgzx6
+-----------+-------------------+
| Principal | User:sa-pjj8o2    |
| Role      | CloudClusterAdmin |
+-----------+-------------------+
```

## <a name="step-6"></a>**Create, Produce and Consume from a topic**
1. Now the Service Account has permission to manage the cluster. We will test it out using Confluent Cli. We will use the API key pair we created for the testing. 

```bash
confluent  api-key use api-key-xxxxxxx --resource lkc-3r6vz2
Set API Key "xxxxxxxxxx" as the active API key for "lkc-3r6vz2".
```

2. Once we set the key for cluster. Confluent CLI will use the key to access the kafka cluster

```bash
c6ebebbd8823:/# confluent kafka topic create testme
Created topic "testme".

c6ebebbd8823:/# confluent kafka topic produce testme
Starting Kafka Producer. Use Ctrl-C or Ctrl-D to exit.
hello
kakfa

c6ebebbd8823:/# confluent kafka topic consume --from-beginning testme
Starting Kafka Consumer. Use Ctrl-C to exit.
kakfa
hello
^CStopping Consumer.
```

3. Remove the role-binding and try consume from topic again(wait a few minutes for RBAC to apply before you do next command)

```bash
c6ebebbd8823:/# confluent iam rbac role-binding delete  --principal User:sa-pjj8o2 --role CloudClusterAdmin --cloud-cluster lkc-3r6vz2 --environment env-ymgzx6
Are you sure you want to delete this role binding? (y/n): y
+-----------+-------------------+
| Principal | User:sa-pjj8o2    |
| Role      | CloudClusterAdmin |
+-----------+-------------------+

c6ebebbd8823:/# confluent kafka topic consume --from-beginning testme
```
## <a name="step-12"></a>**Clean Up Resources**

1. Remove the cluster and Service account.

```bash
c6ebebbd8823:/# confluent kafka cluster list
  Current |     ID     |     Name     |   Type   | Provider |  Region   | Availability | Status
----------+------------+--------------+----------+----------+-----------+--------------+---------
  *       | lkc-3r6vz2 | sandbox-east | STANDARD | aws      | us-east-1 | single-zone  | UP

c6ebebbd8823:/# confluent kafka cluster delete lkc-3r6vz2
Are you sure you want to delete Kafka cluster "lkc-3r6vz2"?
To confirm, type "sandbox-east". To cancel, press Ctrl-C: sandbox-east
Deleted Kafka cluster "lkc-3r6vz2".

c6ebebbd8823:/# confluent iam service-account list
     ID     |  Name  |   Description
------------+--------+------------------
  sa-pjj8o2 | app-sa | sa for demo app

c6ebebbd8823:/# confluent iam service-account delete sa-pjj8o2
Are you sure you want to delete service account "sa-pjj8o2"?
To confirm, type "app-sa". To cancel, press Ctrl-C: app-sa
Deleted service account "sa-pjj8o2".
```

**Deleting the resources you created during this workshop will prevent you from incurring additional charges.**


## <a name="step-13"></a>**Confluent Resources and Further Testing**

* [Confluent Cloud Documentation](https://docs.confluent.io/cloud/current/overview.html)

* [Best Practices for Developing Apache Kafka Applications on Confluent Cloud](https://assets.confluent.io/m/14397e757459a58d/original/20200205-WP-Best_Practices_for_Developing_Apache_Kafka_Applications_on_Confluent_Cloud.pdf) 

* [Confluent Cloud Demos and Examples](https://docs.confluent.io/platform/current/tutorials/examples/ccloud/docs/ccloud-demos-overview.html)

