# Setting up B2B Analytics

> **Note:**  
> This guide is applicable for WSO2 Identity Server 7.2.0 and above.

In addition to the instructions provided in [ELK integration](https://is.docs.wso2.com/en/latest/deploy/elk-analytics-installation-guide/),
following steps should be performed to set up B2B Analytics.

## Configuring WSO2 Identity Server

1. Open the `<IS_HOME>/repository/conf/deployment.toml` file and add the following configurations to enable B2B Analytics.

    ```toml
    [identity_mgt.analytics_b2b_login_data_publisher]
    enable=true
    ```
   
2. Add the respective event publisher file to the `<IS_HOME>/repository/deployment/server/eventpublishers` directory.

    - For HTTP - [B2B Auth HTTP event publisher file](artifacts/wso2-is-configs/event-publishers/http/IsAnalytics-Publisher-wso2event-B2BAuthenticationData.xml)
    
    - For Logger - [B2B Auth Logger event publisher file](artifacts/wso2-is-configs/event-publishers/logger/IsAnalytics-Publisher-wso2event-B2BAuthenticationData.xml)

3. Add the [event stream definition file](artifacts/wso2-is-configs/event-streams/org.wso2.is.analytics.stream.B2BOverallAuthentication_1.0.0.json) to the `<IS_HOME>/repository/deployment/server/eventstreams` directory.

4. Restart WSO2 Identity Server.

## Configuring ELK Analytics

1. Update the logstash configuration file to include the B2B authentication data event stream.

    - For HTTP - [logstash-http.conf](artifacts/logstash/logstash-http.conf)
    
    - For Logger - [logstash-filebeat.conf](artifacts/logstash/logstash-filebeat.conf)

2. Restart Logstash.

3. Import the Kibana saved objects to visualize B2B Analytics data.
    - Log in to the Kibana.
    - Download the [Kibana saved objects](artifacts/kibana/kibana-8.x-b2b-auth.ndjson) file.
    - Navigate to **Stack Management > Saved Object** and click on the import button and add the downloaded artifact file as an import object, and import.
    - You can find the B2B Auth Dashboard in the **Dashboard** section of Kibana.

After completing the above steps, you will be able to view B2B Analytics data in the Kibana dashboard. Additionally, you can customize the dashboard as per your requirements.



