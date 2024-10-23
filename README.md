# **SOC Lab at Home with Splunk: Comprehensive SIEM, Threat Detection, and Response Guide**

## **Overview**
This guide integrates practical exercises and advanced concepts related to Security Operations Centers (SOCs) to facilitate mastery of Splunk as a Security Information and Event Management (SIEM) tool. It provides a structured approach to simulating real-world cybersecurity incidents, enhancing threat detection, investigation, and response skills. By emphasizing current SOC best practices and advanced techniques such as optimized Search Processing Language (SPL), real-time monitoring, and dynamic dashboard creation, this guide aims to provide a comprehensive learning experience.

---

## **Table of Contents**
1. [Splunk Enterprise Installation](#1-splunk-enterprise-installation)
2. [Data Ingestion and Index Creation](#2-data-ingestion-and-index-creation)
3. [Real-time Event Monitoring](#3-real-time-event-monitoring)
4. [Optimized Search Processing with SPL](#4-optimized-search-processing-with-spl)
5. [Use Case: Brute Force Attack Detection](#5-use-case-brute-force-attack-detection)
6. [Advanced Threat Detection Techniques](#6-advanced-threat-detection-techniques)
7. [Configuring Alerts for Threat Detection](#7-configuring-alerts-for-threat-detection)
8. [Creating Dynamic Dashboards for Incident Response](#8-creating-dynamic-dashboards-for-incident-response)
9. [Validation and Hands-On Exercises](#9-validation-and-hands-on-exercises)
10. [Resources](#10-resources)

---

## **1. Splunk Enterprise Installation**

Follow these steps to install and configure Splunk to set up a SOC environment.

1. **Download Splunk Enterprise** from the [Splunk website](https://www.splunk.com/en_us/download.html).
2. **Install Splunk** and access it via `http://localhost:8000` after installation.
3. **Familiarize yourself with features** like **Search**, **Alerts**, and **Dashboards** for effective use in the lab environment.

---

## **2. Data Ingestion and Index Creation**

To simulate SOC data, it is essential to ingest relevant security logs, such as network traffic or syslog data.

### **Steps to Ingest Data**:

1. **Create a New Index**:
   - Navigate to `Settings` > `Indexes` > `New Index`.
   - Name it `soc_lab`.

2. **Upload or Simulate Data**:
   - Use the **BOTS v3** dataset for a realistic dataset ([Download from GitHub](https://github.com/splunk/botsv3)).
   - Alternatively, set up a syslog generator to simulate real-time data ingestion.

3. **Verify Data**:
   - Use a query to confirm that the data has been successfully ingested:
     ```splunk
     index="soc_lab" | head 10
     ```

---

## **3. Real-time Event Monitoring**

In this section, learn how to monitor live data streams for suspicious activity in a SOC environment.

### **Real-Time Monitoring Example**:
Run the following query to detect failed login attempts:
```splunk
index=soc_lab "failed login"
| stats count by user
| where count > 5
```
Apply filters to identify potential brute force attacks in real-time.

---

## **4. Optimized Search Processing with SPL**

Optimizing searches is crucial for performance in a high-volume SOC environment. Utilize SPL best practices for faster and more efficient queries.

### **Key SPL Techniques**:
- **Use Time Constraints** to narrow down the dataset:
   ```splunk
   index=soc_lab earliest=-1h
   ```
- **Avoid Wildcards (`*`)** to reduce query load:
   ```splunk
   index=soc_lab src_ip="192.168.1.*"
   ```
- **Event Sampling** for handling large data volumes:
   ```splunk
   index=soc_lab | sample ratio=0.1
   ```

---

## **5. Use Case: Brute Force Attack Detection**

### **Scenario**:
Detect multiple failed login attempts, which are indicative of brute force attacks.

1. **Search Query**:
   ```splunk
   index=soc_lab sourcetype=access_combined_wcookie action=login
   | stats count by src_ip
   | where count > 100
   ```
2. **Analyze**: Investigate further by identifying IP addresses with over 100 failed login attempts.
3. **Remediation**: Consider blocking the suspicious IP or conducting additional investigations.

---

## **6. Advanced Threat Detection Techniques**

### **Insider Threat Detection**:
Monitor for potential insider threats by identifying abnormal login behavior:
```splunk
index=soc_lab action="login attempt"
| stats count by user status
| where status="failed" AND count > 3
| table user count
```

### **Lateral Movement Detection**:
Identify lateral movement, which can indicate compromised internal systems:
```splunk
index=soc_lab dest_ip="10.0.0.*"
| stats count by src_ip, dest_ip
| where count > 50
| table src_ip dest_ip count
```

### **Geolocation-based Threat Hunting**:
Track activity from suspicious regions using geolocation data:
```splunk
index=soc_lab | iplocation src_ip
| stats count by Country
| where Country="China"
| table src_ip Country count
```

---

## **7. Configuring Alerts for Threat Detection**

Alerts are essential for proactive threat detection. Here’s how to set one up for detecting a potential DoS attack:

### **Steps**:
1. **Search Query**:
   ```splunk
   index=soc_lab sourcetype=firewall action=deny
   | stats count by src_ip
   | where count > 100
   ```

2. **Configure Alert**:
   - Save the search as an alert.
   - Trigger the alert when the count exceeds 100 within a 5-minute window.

---

## **8. Creating Dynamic Dashboards for Incident Response**

Dashboards facilitate real-time, visual monitoring of SOC data.

### **Example Dashboard Panel**:
Create a panel that tracks the top 10 blocked IP addresses:
```splunk
index=soc_lab sourcetype="firewall" action="deny"
| stats count by src_ip
| sort -count
| head 10
| table src_ip count
```
Add interactive filters for easier analysis of specific events.

---

## **9. Validation and Hands-On Exercises**

### **Exercise 1**:
Set up an alert to detect non-US outbound connections using the BOTS v3 dataset.

1. **Access the Splunk Search Interface**:
   - Log in to your Splunk instance and navigate to the **Search** app.

2. **Run a Query to Identify Non-US IP Addresses**:
   - Use the following query to filter outbound connections that are not from the US:
   ```splunk
   index=soc_lab sourcetype=botsv3 
   | iplocation src_ip 
   | search Country!="United States" 
   | stats count by src_ip, Country
   ```

3. **Create an Alert**:
   - Click on the **Save As** dropdown in the search results and select **Alert**.
   - Name the alert (e.g., "Non-US Outbound Connection Alert").
   - Configure the alert conditions (e.g., trigger when the count is greater than 0).
   - Set the alert to trigger every time results are returned.

4. **Configure Alert Actions**:
   - Choose how you want to be notified (email, webhook, etc.) when the alert is triggered.

5. **Save the Alert**: 
   - Click on **Save** to finalize the alert configuration.

### **Exercise 2**:
Create a dashboard panel displaying the top 10 source IP addresses responsible for failed logins.

1. **Access the Splunk Dashboard**:
   - Navigate to the **Dashboards** app and click on **Create New Dashboard**.

2. **Create a New Panel**:
   - Choose **Add Panel** and select **New from Search**.

3. **Run a Query for Failed Login Attempts**:
   - Input the following SPL query to get the top 10 source IP addresses responsible for failed logins:
   ```splunk
   index=soc_lab sourcetype=access_combined_wcookie action=login
   | stats count by src_ip
   | sort -count
   | head 10
   ```

4. **Configure the Panel**:
   - Name the panel (e.g., "Top 10 Failed Login IPs").
   - Select the panel type (e.g., **Table** or **Bar Chart**) based on your preference.

5. **Add the Panel to Your Dashboard**:
   - Click on **Add to Dashboard** to finalize the process.

6. **Save the Dashboard**: 
   - Don’t forget to save your dashboard after adding the panel.

---

## **10. Resources**
- [Splunk Documentation](https://docs.splunk.com/Documentation/Splunk)
- [Splunk Security Essentials](https://splunkbase.splunk.com/app/3435/)
- [BOTS v3 Dataset](https://github.com/splunk/botsv3)

---

### **Conclusion**

This guide provides a comprehensive framework for setting up a SOC lab at home using Splunk as a SIEM tool. It covers essential topics including installation, data ingestion, real-time monitoring, optimized search processing, and advanced threat detection techniques. By engaging with practical exercises and real-world scenarios, individuals can enhance their skills in cybersecurity and incident response, ultimately fostering a deeper understanding of SOC operations.
