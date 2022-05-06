# Introduction to ELK

Introduction to what ELK is and what it does. ELK contains three major components called Elasticsearch, Logstash, Kibana and a new addition named Beats. The ELK Stack helps by providing users with a powerful platform that collects and processes data from multiple data sources, stores that data in one centralized data store that can scale as data grows, and that provides a set of tools to analyze the data. 

## Elasticsearch

Elasticsearch is basically the database for the ELK stack. Elasticsearch 7.x is much easier to setup since it now ships with Java bundled. Performance improvements include a real memory circuit breaker, improved search performance and a 1-shard policy. In addition, a new cluster coordination layer makes Elasticsearch more scalable and resilient.

## Logstash

Logstash takes care of the collecting and conversion of the logs. Logstash’s Java execution engine (announced as experimental in version 6.3) is enabled by default in version 7.x. Replacing the old Ruby execution engine, it boasts better performance, reduced memory usage and overall — an entirely faster experience.

### Http input plugin

We are using this plugin to capture the events which are publishing via http from the WSO2 IS.

### GeoIP plugin

This plugin helps to decode the location data using the IP addresses.

## Kibana

Kibana is the component dedicated to visualizing the data in a meaningful way. Kibana is undergoing some major facelifting with new pages and usability improvements. The latest release includes a dark mode, improved querying and filtering and improvements to Canvas.

## Beats

Beats 7.x conform with the new Elastic Common Schema (ECS) — a new standard for field formatting. Metricbeat supports a new AWS module for pulling data from Amazon CloudWatch, Kinesis and SQS. New modules were introduced in Filebeat and Auditbeat as well.

# Project architecture
![component diagram](images/component-diagram.png)

# Configure WSO2 Identity Server

Follow the steps below to enable ELK-based analytics in WSO2 IS.

1.  Download and install WSO2 Identity Server.

2.  Open the `          deployment.toml         ` file in the
    `          <IS_HOME>/repository/conf/         ` directory.

3.  Enable the following configurations in the `deployment.toml` file.

    ```
    [analytics.elk]
    enable=true
    ```
#### Configure event publishers

We have two ways to publish the data to logstash.

1. HTTP events
In this approach, WSO2 IS will publish events via http. A Http input plugin is used for this approach.

2. Logger
WSO2 Identity Server will publish analytics data into a log file and that file will be used as the source in logstash. Filebeat is used for this approach.

We provide the support for logger approach by default. If you want to use HTTP, replace the following files,

*For HTTP*
- _ **IS\_HOME/repository/deployment/server/eventpublishers/IsAnalytics-Publisher-wso2event-AuthenticationData.xml** _ with [IsAnalytics-Publisher-wso2event-AuthenticationData.xml](/wso2-is-configs/event-publishers/http/IsAnalytics-Publisher-wso2event-AuthenticationData.xml)
- _ **IS\_HOME/repository/deployment/server/eventpublishers/IsAnalytics-Publisher-wso2event-SessionData.xml** _ with [IsAnalytics-Publisher-wso2event-SessionData.xml](/wso2-is-configs/event-publishers/http/IsAnalytics-Publisher-wso2event-SessionData.xml)

# Configuring ELK

#### Installing Elasticsearch

1. [Install Elasticsearch](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-elasticsearch)
   according to your operating system.
2. Make sure Elasticsearch is [up and running](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#_make_sure_that_elasticsearch_is_up_and_running).

In Elasticsearch version 8.x, security features are enabled by default. Basic authentication is enabled and cluster connections are encrypted. Please note down the password of the default elastic user or create a new user with required roles using the
[elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/users-command.html) for next steps.


#### Installing Logstash

1. [Install Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html) according to your operating system.
2. Create a file with '.conf'  extension in logstash directory and add the configurations in [here](logstash/logstash_filebeat.conf). Change the variable parameters according to your setup.
3. Pass this file as the CONFIG_PATH argument when [starting the logstash server](https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html#running-logstash-command-line).

#### Installing Filebeat
1. [Install Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html#installation) according to your operating system.
2. Add the configurations in [here](filebeat/filebeat.yml) to filebeat.yml file to read the log file in the `<IS_HOME>/repository/logs` folder. Change the `<LOGSTASH_HOST>` accordingly.

#### Installing Kibana
1. [Install Kibana](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#install-kibana) according to your operating system.
2. [Launch and access](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html#_access_the_kibana_web_interface) the Kibana web interface.

### Step 3 : Configuring ELK Analytics Dashboards

1. Log in to the Kibana.
2. Navigate to **Stack Management > Index Management >  Index Pattern**. If you already have any index patterns created under the following names, delete them before importing the saved artifacts.
   - auth-events*
   - session-events*
   - chart-session-events*

3. Download the artifact file [here](kibana/saved-objects/kibana-8-x-auth-and-session.ndjson).
4. Navigate to **Stack Management > Saved Object** and click on the import button and add the downloaded artifact file as an import object, and import.
   You can see the Auth, Session dashboards in the Dashboard section.


### View default dashboards

Navigate to &quot;Menu -\&gt; Kibana -\&gt; dashboard&quot; to view the default Authentication and Session dashboards. It is possible to customize these dashboards or create new ones to match your preferences. Follow the next sections if you are interested.

### Creating a new visualization.

You can create your own visualizations and add them to preferred dashboards. Please refer the **Custom Visualizations** section of the documentation to know how.

# Custom Visualizations

Any number of custom visualizations can be created to accommodate your requirements.

### Selecting the visualization type

Navigate to &quot;Menu -\&gt; Kibana -\&gt; Visualize&quot; and there you can see the list of imported visualizations. Click on the button &quot;Create visualization&quot; and you will be presented with a popup menu having many visualizations.

Hover on the visualization types to get a basic idea on it. Select the visualization which you want to create by clicking on it. Continuing with the **Vertical Bar** visualization type for this documentation.

### Selecting the source

When we click on the visualization type, it will ask to choose a source. This is the index pattern which is created on the import (or select the one if you created a custom index pattern). For this documentation, we will choose **auth\*** pattern. 

### Configure the visualization

When the index pattern is selected as the source, blank Vertical Bar chart will appear on the screen.

Configuration controls are located at the right hand side of the screen. Expand the Y-axis label to find more options. We will assign a custom label of &quot;Login Count&quot; to Y-axis.

Now we need to configure the X-axis. Click on the &quot;+Add&quot; button inside the Buckets section to configure the X-axis. Select the &quot;Filters&quot; from the dropdown to create a Kibana Query Language (KQL) based query as the filter for X-axis.

Put &quot;event.payloadData.authenticationSuccess : true&quot; as the query inside the Filter 1 section. You can have multiple filters as well. Now click on the Update button on the bottom right corner.

Now let us provide a Label for X-axis filter as well. For that, click on the label icon which is in parallel to Filter 1 title before the trash bin icon. Give it a label and click update to view it in action.

### Save and add visualization to a dashboard

On the top left corner, there is a Save button. Click on it and provide a unique name to the visualization and click on Save. 

Now we can navigate back to a dashboard and click on the Edit button on the top left corner next to the Clone button. Then click on add button to add our visualization to the dashboard and save it. You can resize and reorder components by default drag and drop options.

### Changing created visualizations

You can always change the existing visualizations by heading into the visualization section of the Kibana. Instead of clicking the &quot;Create new&quot; button, click on an existing visualization to edit the configurations. You can grab an idea on the login behind the default visualizations with that.

# Performance testing
Please follow this https://github.com/wso2/performance-is/tree/master to do the performance testing. 

Following are the results of the tests carried out during the development period. These tests are done as Single Node performance tests with an AWS hosted ELK with Ubuntu 20.04, 15GB SSD, c5.xlarge.

|           |    Scenario   |                           | Duration | Concurrent Users | Throughput (Requests/sec) | Average Response Time (ms) |
|-----------|---------------|---------------------------|----------|------------------|---------------------------|----------------------------|
| Analytics | Session Count | Event Publisher (Adapter) |          |                  |                           |                            |
| NO        | NO            | NONE                      | 10 mins  | 150              | 182.33                    | 821.97                     |
| YES       | NO            | HTTP                      | 10 mins  | 150              | 176.37                    | 849.85                     |
| YES       | NO            | LOGGER                    | 10 mins  | 150              | 161.24                    | 929.76                     |
| YES       | YES           | HTTP                      | 10 mins  | 150              | 24.85                     | 6015.59                    |
| YES       | YES           | LOGGER                    | 10 mins  | 150              | 24.74                     | 6038.08                    |
|           |               |                           |          |                  |                           |                            |
| YES       | NO            | HTTP                      | 10 mins  | 50               | 182.7                     | 272.87                     |
| NO        | NO            | NONE                      | 10 mins  | 50               | 174.59                    | 285.56                     |
| YES       | NO            | HTTP                      | 10 mins  | 100              | 184.7                     | 540.45                     |
| NO        | NO            | NONE                      | 10 mins  | 100              | 191.08                    | 522.41                     |
| YES       | NO            | HTTP                      | 10 mins  | 150              | 176.37                    | 849.85                     |
| NO        | NO            | NONE                      | 10 mins  | 150              | 186.16                    | 805.04                     |
| YES       | NO            | HTTP                      | 10 mins  | 300              | 141.94                    | 2111.59                    |
| NO        | NO            | NONE                      | 10 mins  | 300              | 155.44                    | 1927.94                    |
| YES       | NO            | HTTP                      | 10 mins  | 500              | 132.4                     | 3765.79                    |
| NO        | NO            | NONE                      | 10 mins  | 500              | 159.54                    | 3128.54                    |

# Moving to AWS

Same configurations can be done in an EC2 instance of AWS. Please follow this blog article: https://acpasavarjana.medium.com/setting-up-elk-stack-elastic-stack-on-aws-ec2-instance-fc2e1b006fe3
