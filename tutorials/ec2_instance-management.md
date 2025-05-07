# EC2 Instance Management.

EC2 (Elastic Compute Cloud) is:
A core infrastructure-as-a-service (IaaS) offering that provides virtual servers (instances) to run applications.

Widely used to host websites, apps, databases, and backend services.

In this guide, are some very valuable EC2 instance management tutorials.

## Upgrading An EC2 Instance.

Sometimes, it becomes necessary to upgrade a VM on AWS EC2 to a higher instance type. You can resize an EC2 instance to a more powerful instance with better specs without losing your data. Below is a step-by-step guide:

### **Steps to Upgrade Your EC2 Instance**

1. **Stop the Instance**

Before resizing an EC2 instance, you need to **stop the instance** (but not terminate it, so your data is preserved).

* Go to the **EC2 Dashboard** in AWS Management Console.
* Find your instance in the **Instances** section.
* Click or select the 'Instance state' drop-down/menu and choose **"Stop instance"**.
* Wait until the status changes to "stopped."

> **Note**: Even though I'm not sure about this, I believe you'll still be charged for the storage even when the instance is stopped - but not for the instance usage.

2. **Change the Instance Type**

Once your instance is stopped, you can modify the instance type to a more powerful one.

* Select your stopped instance.
* Click on **"Actions"** > **"Instance Settings"** > **"Change Instance Type"**.
* In the **Instance Type** dropdown, select a new instance type that meets your needs (e.g., from `t2.micro` to `t3.medium`, `t3.large`, or any other higher spec).
* Proceed to change the instance type.

3. **Review Other Configurations (Optional)**

This step is optional but good to consider:

* Check **Elastic IPs** (if applicable) to make sure they are still associated correctly after the resize.
* Review **Security Groups** and **Key Pairs** for any potential modifications.
* If you have any specific network requirements (like using a **VPC**), double-check them as well.

4. **Start the Instance**

After selecting the new instance type, **Start** the instance to reboot it.

### **Important Notes:**

* **Instance Store vs. EBS**: If your instance uses **instance store** volumes (ephemeral storage), the data will be lost when you stop and restart. For **EBS-backed instances**, your data remains intact.

* **Free Tier**: Be aware that some instance types (e.g., `t3.micro`) might **no longer qualify for the Free Tier**, which means you'll be charged for the upgraded instance. Be sure to check the instance type’s eligibility for the Free Tier before proceeding.

* **Elastic IP**: If you’re using an **Elastic IP (EIP)**, it should stay the same as long as the instance is stopped and restarted. But if you're changing regions or terminating the instance, the EIP may need to be reassigned.

> **Generally, on stopping and upgrading an EC2 instance, the instance IP will be automatically changed on re-start**

### **Choosing the Right Instance Type**

If your EC2 instance's performance is limited on the Free Tier (`t2.micro`, `t3.micro`), consider the following:

* **t3.medium**: If you need a balance of compute, memory, and networking.
* **m5.large**: A general-purpose instance with better overall performance.
* **c5.large**: If you need more CPU power for compute-heavy tasks.
* **r5.large**: If your workload is memory-intensive.

### **Cost Considerations**

Upgrading your EC2 instance might **increase costs**:

* Review your **billing and usage** after the upgrade to ensure that you're comfortable with the new charges.
* If you no longer need the Free Tier, consider selecting a new instance type that offers a better cost-to-performance ratio based on your needs.

