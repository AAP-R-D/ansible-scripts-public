### Document: Explanation of Ansible Playbooks for Monitoring and Managing EC2 Instances

---

### **Problem Statement**

Managing EC2 instances involves monitoring the health of servers to prevent potential performance issues and proactively taking actions to resolve problems. Key performance metrics such as CPU and memory utilization are crucial to ensuring optimal server performance. When these metrics exceed a threshold (e.g., 80% utilization), servers may become unresponsive or slow, leading to degraded performance and potential downtime.

Additionally, high resource utilization might require automatic cleanup of unnecessary files on the servers to free up space and improve performance. Manually checking each server’s status and performing cleanup tasks can be time-consuming, especially when managing a large number of servers.

---

### **Solution Overview**

To address this problem, two Ansible playbooks have been created:

1. **Playbook 1**: Monitors CPU and memory utilization of EC2 instances and sends a detailed report via email.
2. **Playbook 2**: Deletes specific files on overutilized servers to free up resources and prevent performance issues.

---

### **Playbook 1: Monitor CPU and Memory Utilization of EC2 Instances**

#### **Problem:**
Monitoring resource utilization across multiple EC2 instances manually can be a tedious task, and missing a critical alert can lead to system failures.

#### **Solution:**
The first playbook automates the process of gathering information about EC2 instances, checks their CPU and memory utilization, and sends an email report detailing the status of each instance. Instances with utilization exceeding a predefined threshold (80%) are highlighted for quick identification.

#### **Playbook Steps:**
1. **Gather EC2 Instance Information**: This step retrieves all instances in a specified AWS region.
2. **Filter Instances with Public IPs**: Only instances with public IP addresses are monitored.
3. **Check CPU and Memory Utilization**: For each instance, CPU and memory metrics are gathered using shell commands.
4. **Prepare Instance Utilization Data**: The gathered data is processed into a structured format, making it easy to generate a report.
5. **Send Email Report**: The data is sent as an HTML-formatted email, highlighting overutilized instances with a red background.
6. **Store Overutilized Server IPs**: The IPs of servers with high CPU or memory utilization are stored for use in the next playbook.

#### **Technical Details:**
- **AWS Interaction**: The playbook interacts with AWS to gather instance information using the `amazon.aws.ec2_instance_info` module.
- **Shell Commands**: Commands like `vmstat` and `free` are used to retrieve CPU and memory utilization from each instance.
- **Email Notification**: The `ansible.builtin.mail` module is used to send a report with system utilization details, including instances with CPU and memory utilization above 80%.

---

### **Playbook 2: Delete Files on Overutilized Servers**

#### **Problem:**
When a server becomes overutilized (high CPU or memory usage), it may be necessary to delete unnecessary files to free up space and improve system performance. Doing this manually for each server is time-consuming and prone to errors.

#### **Solution:**
The second playbook automates the deletion of specific files on servers that have been identified as overutilized. This ensures that corrective action is taken promptly, freeing up resources and potentially preventing system crashes.

#### **Playbook Steps:**
1. **Delete Files on Overutilized Servers**: The playbook deletes a specified file (e.g., `/tmp/target_file.txt`) on each server that was flagged as overutilized in the previous playbook. This action is performed using SSH with sudo privileges.

#### **Technical Details:**
- **File Deletion**: The `ansible.builtin.file` module is used to delete the target file from overutilized servers.
- **SSH and Sudo Access**: The playbook connects to the overutilized servers using SSH credentials and deletes files with elevated (sudo) permissions.
- **Dynamic Server List**: The list of overutilized servers is dynamically provided from the results of the first playbook, making the cleanup process seamless.

---

### **File Structure:**

1. **Main Playbooks:**
   - `monitor_ec2_instances.yml`: Playbook 1 for monitoring CPU/memory and sending email reports.
   - `delete_files_on_servers.yml`: Playbook 2 for deleting files on overutilized servers.

2. **Variable Files:**

   - `vars.yml`: Contains reusable variables such as AWS credentials, SSH credentials, email configuration, and the file path for deletion.
