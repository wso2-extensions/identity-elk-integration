# Introduction to ELK

Introduction to what ELK is and what it does. ELK contains three major components called Elasticsearch, Logstash, Kibana and a new addition named Beats.

## Elasticsearch

Elasticsearch is basically the database for the ELK stack.

## Logstash

Logstash takes care of the collecting and conversion of the logs.

### Http input plugin

We are using this plugin to capture the events which are publishing via http from the WSO2 IS.

### GeoIP plugin

This plugin helps to decode the location data using the IP addresses.

## Kibana

Kibana is the component dedicated to visualizing the data in a meaningful way.

### Enhanced Data Table plugin

Enhanced data table is used to dump all the records into a table with added features for the default Data Table.

# Setting up ELK

First, you need to set up ELK on your local machine.

## Java installation

Java is required to run ELK in your machine. Skip this part if you already have Java installed. If not please follow the instructions here.

### Check if Java is already installed

Open a terminal and run &quot;java -version&quot; and if you get an output similar to this then you already have Java installed.

`avarjana@avarjana:~$ **java -version**`

`openjdk version &quot;11.0.9&quot; 2020-10-20`

`OpenJDK Runtime Environment (build 11.0.9+11-Ubuntu-0ubuntu1.18.04.1)`

`OpenJDK 64-Bit Server VM (build 11.0.9+11-Ubuntu-0ubuntu1.18.04.1, mixed mode, sharing)`

### Install Java

Please install Java by running the following command.

`sudo apt-get install default-jre`

## Install Logstash

Installing Logstash is straightforward. Just run,

sudo apt-get install logstash

## Install Elasticsearch

Installing and configuring Elasticsearch must go through a few steps. Add the repository key first,

`wget -qO -[https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch) | sudo apt-key add -`

Install the Apt Transport HTTPS since it might not be installed by default.

`sudo apt-get install apt-transport-https`

Now run the following commands to complete the installation.

`echo https://artifacts.elastic.co/packages/7.x/apt stable main&quot; | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

sudo apt-get update &amp;&amp; sudo apt-get install elasticsearch`

You have successfully installed Elasticsearch but there are few more configurations to be done. Replace the files,

- _ **/etc/elasticsearch/elasticsearch.yml** _ with [elasticsearch.yml](/Elasticsearch/elasticsearch.yml)
- _ **/etc/elasticsearch/jvm.options** _ with [jvm.options](/Elasticsearch/jvm.options)

## Install Kibana

Install Kibana bu running,

`sudo apt-get install kibana`

# Configure WSO2 Identity Server

Now we need to configure WSO2 IS to publish events to ELK via HTTP event adapter.

## Enable event publishers

Extra configurations should be added to _ **deployment.toml** _ in order to enable the event publishing. Replace the file,

- _ **IS\_HOME/repository/conf/deployment.toml** _ with [deployment.toml](/WSO2-IS-Configs/deployment.toml)

## Configure event publishers

Authentication and Session event publishers are needed to be configured to publish to Logstash. Replace the files,

- _ **IS\_HOME/repository/deployment/server/eventpublishers/IsAnalytics-Publisher-wso2event-AuthenticationData.xml** _ with [IsAnalytics-Publisher-wso2event-AuthenticationData.xml](/WSO2-IS-Configs/EventPublishers/IsAnalytics-Publisher-wso2event-AuthenticationData.xml)
- _ **IS\_HOME/repository/deployment/server/eventpublishers/IsAnalytics-Publisher-wso2event-SessionData.xml** _ with [IsAnalytics-Publisher-wso2event-SessionData.xml](/WSO2-IS-Configs/EventPublishers/IsAnalytics-Publisher-wso2event-SessionData.xml)

# Configuring ELK to Visualize the data

Configuring ELK has three components namely,

1. Logstash
2. Elasticsearch
3. Kibana

## Logstash

Configure Logstash to capture events from WSO2 IS via HTTP.

### Change the conf file

Replace the default Logstash configuration file located at _ **/etc/logstash/conf.d/apache-01.conf** _ with [logstash.conf](/Logstash/logstash.conf)

### Start the Logstash service

Logstash is running as a linux service and can be started by executing,

`sudo service logstash start`

It will take around 30 seconds to 1 minute to complete the startup. You can always check the status by running,

`sudo service logstash status`

## Kibana

Configure Kibana to get the index patterns from Elasticsearch and create visualizations to present them.

### Start Kibana service

Kibana is also running as a linux service and can be started by executing,

`sudo service kibana start`

Navigate to [_ **http://localhost:5601** _](http://localhost:5601/)and wait for the page to be available.

### Import the saved objects to Kibana

Navigate to &quot;Menu -> Management -> Stack Management -> Kibana -> Saved Objects&quot; and click on the import button on the top right hand corner. Provide the following [ndjson file](/Kibana/Exports) and click import.

### Install the enhanced data table plugin

Get the plugin zip file from \&lt;\&lt;git url\&gt;\&gt; and install it by executing the following command.

KIBANA\_HOME/bin/kibana-plugin install file:///path/to/enhanced-table-X.Y.Z\_A.B.C.zip

### View default dashboards

Navigate to &quot;Menu -\&gt; Kibana -\&gt; dashboard&quot; to view the default Authentication and Session dashboards. It is possible to customize these dashboards or create new ones to match your preferences. Follow the next sections if you are interested.

(Authentication dashboard)

(Session dashboard)

### Creating a new visualization.

You can create your own visualizations and add them to preferred dashboards. Please refer the **Custom Visualizations** section of the documentation to know how.

## Elasticsearch

Configure the Elasticsearch to have the required indices. When you import the ndjson for Kibana, it will automatically create the required indices for default dashboards.

# Custom Visualizations

Any number of custom visualizations can be created to accommodate your requirements.

### Selecting the visualization type

Navigate to &quot;Menu -\&gt; Kibana -\&gt; Visualize&quot; and there you can see the list of imported visualizations. Click on the button &quot;Create visualization&quot; and you will be presented with a popup menu having many visualizations.

(Visualization types)

Hover on the visualization types to get a basic idea on it. Select the visualization which you want to create by clicking on it. Continuing with the **Vertical Bar** visualization type for this documentation.

### Selecting the source

When we click on the visualization type, it will ask to choose a source. This is the index pattern which is created on the import (or select the one if you created a custom index pattern). For this documentation, we will choose **auth\*** pattern. 

(Index pattern sources)

### Configure the visualization

When the index pattern is selected as the source, blank Vertical Bar chart will appear on the screen.

(Blank Vertical Bar chart)

Configuration controls are located at the right hand side of the screen. Expand the Y-axis label to find more options. We will assign a custom label of &quot;Login Count&quot; to Y-axis.

(Custom label assigned)

Now we need to configure the X-axis. Click on the &quot;+Add&quot; button inside the Buckets section to configure the X-axis. Select the &quot;Filters&quot; from the dropdown to create a Kibana Query Language (KQL) based query as the filter for X-axis.


(Filter based on KQL)

Put &quot;event.payloadData.authenticationSuccess : true&quot; as the query inside the Filter 1 section. You can have multiple filters as well. Now click on the Update button on the bottom right corner.

(After providing the query)

Now let us provide a Label for X-axis filter as well. For that, click on the label icon which is in parallel to Filter 1 title before the trash bin icon. Give it a label and click update to view it in action.


(After providing a label)

### Save and add visualization to a dashboard

On the top left corner, there is a Save button. Click on it and provide a unique name to the visualization and click on Save. 

(Saving the visualization)

Now we can navigate back to a dashboard and click on the Edit button on the top left corner next to the Clone button. Then click on add button to add our visualization to the dashboard and save it. You can resize and reorder components by default drag and drop options.

### Changing created visualizations

You can always change the existing visualizations by heading into the visualization section of the Kibana. Instead of clicking the &quot;Create new&quot; button, click on an existing visualization to edit the configurations. You can grab an idea on the login behind the default visualizations with that.
