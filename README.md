# Configuring-CloudLogging-CloudMonitoring-on-GoogleCloud

## Objectives:
- View logs using a variety of filtering mechanisms.
- Exclude log entries and disable log ingestion.
- Export logs and run reports against exported logs.
- Create and report on logging metrics.
- Use Cloud Monitoring to monitor different Google Cloud projects.
- Create a metrics dashboard.

**Task 1. Setting up resources in the first project**

In this task, Google Cloud resources for the first project will be created.

The second project will contain the Monitoring account configuration data.

1. Activate the Cloud Shell.

2. In the Cloud Shell, download and unpack an archive that contains the setup code:
 
 ```
curl https://storage.googleapis.com/cloud-training/gcpsec/labs/stackdriver-lab.tgz | tar -zxf -
cd stackdriver-lab
```

3. Click on the Open Editor icon in the top-right corner of your Cloud Shell session.

4. Click Open in a new window if prompted.

5. Open the stackdriver-lab folder and select the linux_startup.sh file.

6. Replace the `# install logging agent` and `# install monitoring agent` sections with the following:
```
# install logging agent
curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
sudo bash add-logging-agent-repo.sh --also-install
# install monitoring agent
curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
sudo bash add-monitoring-agent-repo.sh --also-install
```

7. After pasting, make sure that your lines of code are properly indented.
	
8. Save your file.

9. Now open the setup.sh file.

10. Update the image version in # create vms section for windows-server (row 17) after --image with the following:
```
windows-server-2016-dc-core-v20210511
```
11. Add the following flags at the end of lines 17 and 18 to set the machine type for both the Linux and Windows VMs:
```
--machine-type=e2-micro
```
12. After pasting, make sure that your lines of code are properly indented.

13. Save your file.

14. In the Cloud Console, click Open Terminal in the top-right corner.

15. Now run the following command:
```
./setup.sh
```
### The created resources will include:
- Service accounts (for use by VMs).
- Role assignments (granting service accounts permissions to write to Monitoring).
- A Linux VM with Apache and the Monitoring agents installed.
- A Windows VM with the Monitoring and Logging agents installed.
- A Google Kubernetes Engine cluster with an Nginx deployment.
- A Pub/Sub Topic and Subscription.
**Note: If you get an error message similar to the following:**
  
`ERROR:(gcloud.compute.instances.create)Could not fetch resource:---code:ZONE_RESOURCE_POOL_EXHAUSTED`

Run the following command to replace the zones in the setup script with a new one:
```
sed -i 's/us-west1-b/labzone/g' setup.sh
```

And re-run the startup script command:
```
./setup.sh
```

**Task 2. View and filter logs in first project**

In this task, VM instance logs will be viewed with simple filtering.

1. In the Log fields panel, click `VM Instance`.

2. In the `Instance Id` field, select one of the instance IDs.
	
3. Click inside the Query box. This now becomes editable.
	
4. In the Query box, remove everything after line 1. You should see only line 1, which contains `resource.type="gce_instance"`.
	
5. Click `Run query`.
	
6. Turn on streaming logs by clicking `Stream logs`.
	
7. You should see new log entries showing up every 1-2 seconds as the background activity is generating unauthorized requests against your Web servers.

8. Stop log streaming by clicking on Stop stream in the top-right corner.
	
9. Click on the `Log Name` dropdown, select `syslog`, and then click `Apply`.

**Task 3. Use log exports**

In this task, test log exports to BigQuery will be configured and tested.

Cloud Logging retains log entries for 30 days. In most circumstances, you'll want to retain some log entries for an extended time (and possibly perform sophisticated reporting on the archived logs).

Configure the export to BigQuery

	1. Go to Cloud Logging Exports (Navigation menu > Logging > Log Router).
	2. Click Create Sink.
	3. For the Sink name, type vm_logs and then click Next.
	4. For Select sink service, select BigQuery dataset.
	5. For Select BigQuery dataset, select Create new BigQuery dataset.
	6. For the Dataset ID, type project_logs, and click Create Dataset.
	7. Click Next.
	8. In the Build inclusion filter list box, copy and paste resource.type="gce_instance".
	9. Click Create Sink. You will now return to a Log Router Create log sink next steps page (a message at the top may appear that says "Your log sink was successfully created. Data should be available soon.")
Note: You could also export log entries to Pub/Sub or Cloud Storage.
Exporting to Pub/Sub can be useful if you want to flow through an ETL process prior to storing in a database (Monitoring > Pub/Sub > Dataflow > BigQuery/Bigtable).
Exporting to Cloud Storage will batch up entries and write them into Cloud Storage objects approximately every hour.
Configure HTTP load balancing exports to BigQuery
You will now create an export for the HTTP load balancing logs to BigQuery.
	1. From the left-hand navigation menu, select Log Router to return to the service homepage.
	2. Click Create Sink.
	3. For the Sink name, type load_bal_logs and then click Next.
	4. For Select sink service, select BigQuery dataset.
	5. For Select BigQuery dataset, select project_logs. (You created this BigQuery dataset in the previous set of steps.)
	6. Click Next.
	7. In the Build inclusion filter list box, copy and paste resource.type="http_load_balancer".
	8. Click Create Sink.
	9. You will now be on the Create log sink next steps page for the log sink.
	10. From the left-hand navigation menu, select Log Router to return to the service homepage.
The Log Router page appears, displaying a list of sinks (including the one you just created—load_bal_logs).
Investigate the exported log entries
	1. Open BigQuery (Navigation menu > BigQuery).
	2. The "Welcome to BigQuery in the Cloud Console" message box opens. This message box provides a link to the quickstart guide and lists UI updates.
	3. Click Done.
	4. In the left pane in the Explorer section, click the arrow next to your project (this starts with qwiklabs-gcp-xxx) and you should see a project_logs dataset revealed under it.
You will now verify that the BigQuery dataset has appropriate permissions to allow the export writer to store log entries.
	5. Click on the three dotted menu item ("View actions") next to the project_logs dataset and click Open.
	6. Then from the top-right hand corner of the Console, click the Sharing dropdown and select Permissions.
	7. On the Dataset permission page, you will see that your service accounts have the "BigQuery Data Editor" role.
	8. Close the dataset permissions panel.
	9. Expand the project_logs dataset to see the tables with your exported logs—you should see multiple tables (one for each type of log that's receiving entries).
	10. Click on the syslog_(1) table, then click Details to see the number of rows and other metadata. If the syslog_(1) table is not visible, try refreshing the browser.
	11. In Details tab, under the table info you will see the full table name in the Table ID, copy this table name.
Note: Because the log entries are being streamed into BigQuery as they arrive to Cloud Monitoring, they are stored in a BigQuery streaming buffer. Roughly 24 hours after arriving in the buffer, they will be moved into regular BigQuery storage. You can perform queries against the table and both the data in regular storage and the buffer will be scanned.
	12. To see a subset of your tables fields, paste the below query in the query editor tab (replacing qwiklabs-gcp-xx.project_logs.syslog_xxxxx with the table name copied in the previous step).
```
SELECTlogName, resource.type, resource.labels.zone, resource.labels.project_id,
FROM`qwiklabs-gcp-xx.project_logs.syslog_xxxxx`
```
13. Then click Run.

**Task 4. Create a logging metric**

In this task, you create a metric that you can use to generate alerts if too many web requests generate access denied log entries.

	1. Go back to the Logs Explorer page (Navigation menu > Logging > Logs Explorer).
Note: If prompted, click Leave for unsaved work.
	2. Select Create Metric (right-hand side of the Console) to create a logging metric based on this filter.
	3. In the Log-based metric Editor, set Metric Type as Counter.
	4. Under the Details section, set the Log-based metric name to 403s.
	5. Under the Filter selection for Build filter, enter the following and replace PROJECT_ID with GCP Project ID 1:
```
resource.type="gce_instance"
log_name="projects/PROJECT_ID/logs/syslog"
```
6. Leave all the other fields at their default.
	7. Click Create Metric.
	8. You will make use of this metric in the dashboarding portion of the lab.

**Task 5. Create a monitoring dashboard**

In this task, you switch to the second project created by Qwiklabs and setup a Monitoring workspace.
Switch projects
	1. Switch to the second project created by Qwiklabs (use the GCP Project ID 2 from the Qwiklabs Connection Details). The current project ID is displayed at the top of the console.

2. Click the project name at the top of the Cloud Console and click the All tab.

	3. Click the second project you want to switch to. Verify it is the GCP Project ID 2 from the Qwiklabs Connection Details.
	4. Click Open.
    
Create a Monitoring workspace

You will now setup a Monitoring workspace that's tied to your Google Cloud Project. The following steps create a new account that has a free trial of Monitoring.
	1. In the Cloud Console, click on Navigation menu > Monitoring.
	2. Wait for your workspace to be provisioned.
When the Monitoring dashboard opens, your workspace is ready.

Now add the first project to your Cloud Monitoring workspace.

3. In the left menu, click Settings and then click + Add GCP Projects.
   
4. Click Select Projects
	
 5. Select the checkmark next to your first project ID and click Select.
	
 6. Click Add Projects.
    
Create a monitoring dashboard

1. In the left pane, click Dashboards.
	
 2. Click + Create Dashboard.
	
 3. Replace the generic dashboard name at the top with Example Dashboard.
	
 4. Click Add Widget > Line.
	
 5. For Widget Title, enter in CPU Usage.
	
 6. Click the Metric dropdown.
	
 7. Turn the toggle for "Show only active resources and metrics" to the Off state.
	
 8. For Metric, select VM Instance > Instance > CPU usage. Make sure it's the one that follows the format: compute.googleapis.com/instance/cpu/usage_time.
	
 9. Click Apply.
	
 10. Now click Apply in the top-right corner.
	
 11. Click Add Widget > Line.
	
 12. For Widget Title, enter in Memory Utilization.
	
 13. Click the Metric dropdown.
	
 14. Turn the toggle for "Show only active resources and metrics" to the Off state.
	
 15. For Metric, select VM Instance > Memory > Memory Utilization. Make sure it's the one that follows the format: agent.googleapis.com/memory/percent_used.
	
 16. Click Apply.
	
 17. Now click Apply in the top-right corner.
     
You should now your two graphs—one for CPU usage and the other for memory utilization—populated.

