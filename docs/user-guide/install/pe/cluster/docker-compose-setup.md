---
layout: docwithnav-pe
assignees:
- ashvayka
title: ThingsBoard Professional Edition cluster setup with Docker Compose guide
description: ThingsBoard Professional Edition cluster setup with Docker Compose guide
redirect_from: "/docs/user-guide/install/pe/docker-cassandra/"  

---

* TOC
{:toc}

{% assign docsPrefix = "pe/" %}

This guide will help you to setup ThingsBoard in cluster mode with Docker Compose. 
For this purpose, we will use docker container images available on [Docker Hub](https://hub.docker.com/search?q=thingsboard&type=image&image_filter=store).  

## Prerequisites

ThingsBoard Microservices are running in dockerized environment.
Before starting please make sure [Docker CE](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) are installed in your system. 

{% capture rule_engine_note %}
Please note that for the deployment of Rule Engine as a separate service, an additional separate License Key is required. 
{% endcapture %}
{% include templates/info-banner.md content=rule_engine_note %}

{% include templates/install/docker-install-note.md %}
## Step 1. Checkout all ThingsBoard PE Images

{% include templates/install/dockerhub/checkout.md %}

## Step 2. Pull ThingsBoard PE Images

{% include templates/install/dockerhub/pull.md %}

## Step 3. Clone ThingsBoard PE Docker Compose scripts

```bash
git clone -b release-{{ site.release.ce_ver }} https://github.com/thingsboard/thingsboard-pe-docker-compose.git tb-pe-docker-compose
cd tb-pe-docker-compose
```
{: .copy-code}

## Step 4. Obtain your license key

We assume you have already chosen your subscription plan or decided to purchase a perpetual license. 
If not, please navigate to [pricing](/pricing/) page to select the best license option for your case and get your license. 
See [How-to get pay-as-you-go subscription](https://www.youtube.com/watch?v=dK-QDFGxWek){:target="_blank"} or [How-to get perpetual license](https://www.youtube.com/watch?v=GPe0lHolWek){:target="_blank"} for more details.

**IMPORTANT NOTE:** if you decide to use an [advanced deployment type](/docs/user-guide/install/pe/cluster/docker-compose-setup/#step-6-configure-deployment-type), make sure you have purchased a license key for at least four instances of ThingsBoard PE. 
Otherwise, you need to modify the local copy of [docker-compose.yml](https://github.com/thingsboard/thingsboard-pe-docker-compose/blob/master/advanced/docker-compose.yml)) 
to use the number of ThingsBoard instances that you've purchased.
We will reference the license key you have obtained during this step as PUT_YOUR_LICENSE_SECRET_HERE later in this guide.


## Step 5. Configure your license key

```bash
nano tb-node.env
```
{: .copy-code}

and put the license secret parameter instead of "PUT_YOUR_LICENSE_SECRET_HERE":

```bash
# ThingsBoard server configuration
...
TB_LICENSE_SECRET=PUT_YOUR_LICENSE_SECRET_HERE
```

## Step 6. Configure deployment type

Starting ThingsBoard v2.2, it is possible to install ThingsBoard cluster using new microservices architecture and docker containers. 
See [**microservices**](/docs/reference/msa/) architecture page for more details.

The docker compose scripts support three deployment modes. In order to set the deployment mode, change the value of `TB_SETUP` variable in `.env` file to one of the following:

- `basic` **(recommended, set by default)** - ThingsBoard Core and Rule Engine are launched inside one JVM (requires only one license).
  MQTT, CoAP and HTTP transports are launched in separate containers.
- `monolith` - ThingsBoard Core and Rule Engine are launched inside one JVM (requires only one license). 
  MQTT, CoAP and HTTP transports are also launched in the same JVM to minimize memory footprint and server requirements.
- `advanced`- ThingsBoard Core and Rule Engine are launched in separate containers and are replicated one JVM (requires 4 licenses).  
  
All deployment modes support separate JS executors, Redis, and different [queues](/docs/user-guide/install/pe/cluster/docker-compose-setup/#step-8-choose-thingsboard-queue-service).

## Step 7. Configure ThingsBoard database

{% include templates/install/configure-db-docker-compose.md %}

## Step 8. Choose ThingsBoard queue service 

{% include templates/install/install-queue-docker-compose.md %}

{% capture contenttogglespecqueue %}
Kafka <small>(default, recommended for on-prem, production installations)</small>%,%kafka%,%templates/install/cluster-queue-kafka.md%br%
AWS SQS <small>(managed service from AWS)</small>%,%aws-sqs%,%templates/install/cluster-queue-aws-sqs.md%br%
Google Pub/Sub <small>(managed service from Google)</small>%,%pubsub%,%templates/install/cluster-queue-pub-sub.md%br%
Azure Service Bus <small>(managed service from Azure)</small>%,%service-bus%,%templates/install/cluster-queue-service-bus.md%br%
RabbitMQ <small>(for small on-prem installations)</small>%,%rabbitmq%,%templates/install/cluster-queue-rabbitmq.md%br%
Confluent Cloud <small>(Event Streaming Platform based on Kafka)</small>%,%confluent-cloud%,%templates/install/cluster-queue-confluent-cloud.md{% endcapture %}

{% include content-toggle.html content-toggle-id="ubuntuThingsboardQueue" toggle-spec=contenttogglespecqueue %}

## Step 9. Enable monitoring (optional)

{% include templates/install/configure-monitoring-docker-compose.md %}

## Step 10. Running

Execute the following command to create log folders for the services and chown of these folders to the docker container users. 
To be able to change user, **chown** command is used, which requires sudo permissions (script will request password for a sudo access): 

```bash
./docker-create-log-folders.sh
```
{: .copy-code}

Execute the following command to run installation:

```bash
./docker-install-tb.sh --loadDemo
```
{: .copy-code}

Where:

- `--loadDemo` - optional argument. Whether to load additional demo data.

Execute the following command to start services:

```bash
./docker-start-services.sh
```
{: .copy-code}

After a while when all services will be successfully started you can open `http://{your-host-ip}` in you browser (for ex. `http://localhost`).
You should see ThingsBoard login page.

Use the following default credentials:

- **System Administrator**: sysadmin@thingsboard.org / sysadmin

If you installed DataBase with demo data (using `--loadDemo` flag) you can also use the following credentials:

- **Tenant Administrator**: tenant@thingsboard.org / tenant
- **Customer User**: customer@thingsboard.org / customer

In case of any issues you can examine service logs for errors.
For example to see ThingsBoard node logs execute the following command:

```bash
docker-compose -f $TB_SETUP/docker-compose.yml logs -f tb-core1 tb-rule-engine1
```
{: .copy-code}


Or use the following command to see the state of all the containers:

```bash
docker-compose -f $TB_SETUP/docker-compose.yml ps
```
{: .copy-code}

Use the following command to inspect the logs of all running services.

```bash
docker-compose -f $TB_SETUP/docker-compose.yml logs -f
```
{: .copy-code}

See [docker-compose logs](https://docs.docker.com/compose/reference/logs/) command reference for details.

Execute the following command to stop services:

```bash
./docker-stop-services.sh
```
{: .copy-code}

Execute the following command to stop and completely remove deployed docker containers:

```bash
./docker-remove-services.sh
```
{: .copy-code}

Execute the following command to update particular or all services (pull newer docker image and rebuild container):

```bash
./docker-update-service.sh [SERVICE...]
```
{: .copy-code}

Where:

- `[SERVICE...]` - list of services to update (defined in docker-compose configurations). If not specified all services will be updated.

{% include templates/install/upgrade-docker-compose.md %}

{% include templates/install/generate_certificate_docker-compose.md %}

## Next steps

{% assign currentGuide = "InstallationGuides" %}{% include templates/guides-banner.md %}
