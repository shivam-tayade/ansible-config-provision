# **Automated Server Provisioning and Configuration with Ansible**

### **Project Overview**

This project automates the provisioning and configuration of web and database servers using Ansible. By using Ansible playbooks, we streamline the setup process for Apache, MySQL, and other essential utilities on the target servers. This ensures that all servers are set up in a consistent manner, reducing errors and saving time.

### **Master and Node Names**

- **Master Node**: This is the control node where the Ansible playbook is executed to manage and configure the target servers.
    - Instance ID: `i-02c71bd605261254e`
    - Public IP: `13.233.126.205`
- **Web Node**: This server will have Apache installed and will serve web content.
    - Instance ID: `i-042d236889f6a3ec2`
    - Public IP: `13.126.249.128`
- **Database Node**: This server will run MySQL and host databases.
    - Instance ID: `i-04568fea43f2ed21f`
    - Public IP: `13.203.156.161`
    
    ### **Instructions**
    
    This guide covers all necessary steps, from setting up the directory structure to verifying the installation on the nodes.
    
    ---
    
    ![image.png](attachment:092c09b7-fe6d-430e-ba11-59e4d3e8567d:image.png)
    

---

**Connect Master Node to Web-SV and DB-SV Nodes**

Before proceeding with the configuration, ensure that the master node can communicate with the web and database nodes. This is a crucial step for Ansible to manage the target nodes.

Here’s a detailed guide to generating an SSH key on the master node and copying it to the web and database servers:

### **Step 1: Generate an SSH Key on the Master Node**

First, you need to generate an SSH key on the **master node** that will be used to authenticate the connection between the master node and the target nodes (web and database servers).

1. On the **master node**, open a terminal and run the following command to generate an RSA SSH key:
    
    ```bash
   
    ssh-keygen -t rsa -b 4096
    
    ```
    
    This command generates a secure RSA key pair (private and public key) with 4096 bits. You can press Enter to accept the default file location (`/home/ubuntu/.ssh/id_rsa`), and then optionally provide a passphrase to further secure your key (or leave it empty for no passphrase).
    
2. After generating the key, you'll have two files:
    - The **private key**: `/home/ubuntu/.ssh/id_rsa` (kept securely on the master node)
    - The **public key**: `/home/ubuntu/.ssh/id_rsa.pub` (this key will be copied to the target nodes)

### **Step 2: Copy the Public Key to the Web and Database Nodes**

Now, copy the generated **public key** to both the **web** and **database nodes**. This allows Ansible (running on the master node) to authenticate with the web and database servers without needing a password.

1. On the **master node**, use the `ssh-copy-id` command to copy the public key to the **web server** (`web-sv`):
    
    ```bash
 
    ssh-copy-id ubuntu@13.126.249.128   # Web Node
    
    ```
    
    Replace `13.126.249.128` with the **public IP** of your web server. You will be prompted to enter the password for the `ubuntu` user on the web server. Once the password is entered, the public key will be added to the `~/.ssh/authorized_keys` file on the web server.
    

1. Repeat the process for the **database server** (`db-sv`):
    
    ```bash
  
    ssh-copy-id ubuntu@13.203.156.161   # Database Node
    
    ```
    
    Replace `13.203.156.161` with the **public IP** of your database server. Enter the password for the `ubuntu` user on the database server to add the public key.
    

![image.png](attachment:00434439-3804-4638-8415-1ffd186a1028:image.png)

![image.png](attachment:7ba4b656-f0d7-41c2-ba68-37a00b7c54db:image.png)

### **Enable SSH and HTTP Access on Nodes**

Now that SSH access is set up, ensure that **SSH** and **HTTP** (Apache) ports are open on the target nodes.

![image.png](attachment:80a3d367-f0e6-4159-975a-664205bccb8a:image.png)

### **1. Directory Setup**

Start by creating the directory structure for your project. This setup is crucial to ensure that all tasks and roles are organized.

```bash
mkdir -p server-setup/roles/common/tasks
mkdir -p server-setup/roles/webserver/tasks
mkdir -p server-setup/roles/database/tasks
mkdir -p server-setup/roles/firewall/tasks
cd server-setup
```

![image.png](attachment:ad39e73e-f904-4f33-be06-2af4b745c68f:image.png)

---

### **2. Create the Inventory File (inventory.ini)**

Define your target servers in the `inventory.ini` file. This file contains information on the web and database servers.

![image.png](attachment:1f65fab7-c68e-4e7e-bdf0-39b67d523026:image.png)

### **1. Run the `ansible` Command to Gather Facts (IP Check)**

You can use the `setup` module to gather facts, but you can limit the output to the IP address. The IP address can be found in the `ansible_all_ipv4_addresses` variable.

Run the following command from the **master node** to gather the facts and check the IPs:

```bash
ansible all -m setup -i inventory.ini -a "filter=ansible_all_ipv4_addresses"
```

![image.png](attachment:8ae93db0-ba36-44d2-8cef-510a12e27dd3:image.png)

---

### **3. Create the Main Playbook (site.yml)**

This is the entry point for running your entire setup. The `site.yml` file calls each role in sequence.

![image.png](attachment:8029922e-03b8-41c6-9a78-a456007cf3e4:image.png)

---

### **4. Role: common**

The **common** role handles basic setup tasks that apply to all servers. This typically includes updating the package cache and installing required utilities.

![image.png](attachment:bd3ae0f5-cd26-494f-ac58-81727659ba16:c6c1be92-e17a-42f5-b16a-2ed5c921a805.png)

---

### **5. Role: webserver**

The **webserver** role will install Apache, ensure it’s started, and deploy a basic HTML page to the web server.

![image.png](attachment:d567530e-e962-4599-977d-defab45410a1:image.png)

---

### **6. Role: database**

The **database** role installs MySQL and ensures it is running on the target server.

![image.png](attachment:e19c96f3-5b99-40ef-af73-05e83d45375b:image.png)

---

### **7. Running the Playbook**

Once all your roles are defined, you can run the playbook using the following command. This will apply the configuration to the target servers.

```bash
ansible-playbook -i inventory.ini site.yml
```

![image.png](attachment:f6adeef3-9d59-4fae-ba3b-afb9ca6b6dd3:image.png)

---

### **8. Confirming Installation on Nodes**

After running the playbook, it’s important to confirm that the installations and services are running correctly on your target nodes. Here are the steps to verify that everything has been installed and configured properly.

### **Step 1: Check Apache Installation on web-sv**

To confirm that Apache is installed and running on the web server node, use the following command:

```bash
sudo systemctl status apache2
```

![image.png](attachment:df1f8c58-0f1b-4f36-8329-198661e16b5f:image.png)

### **Open the IP in Your Browser**

Open a web browser (on any machine that has network access to the web server) and enter the IP address of the web server in the address bar:

```
http://13.203.156.161/
```

![image.png](attachment:9abd5800-050d-4652-895c-6fc74617bcd2:image.png)

**Step 2: Check MySQL Installation**

To verify that MySQL is properly installed and running on the database server node, run:

```bash
sudo systemctl status mysql
```

To confirm MySQL is functioning correctly, you can log into the MySQL shell:

```bash
sudo mysql -u root -p
```

You should be prompted for the MySQL root password, and once entered, you will be inside the MySQL shell. You can check the status of MySQL by typing:

```sql
SHOW DATABASES;
```

This will show the list of databases, confirming that MySQL is running.

![image.png](attachment:a4a59e75-281d-4c40-a333-be40e96bc9c4:image.png)

### **9. Troubleshooting**

If you encounter any issues, you can troubleshoot using the following methods:

- **Apache**: If Apache is not running, check the error logs:

```bash

sudo journalctl -u apache2
```

- **MySQL**: If MySQL fails to start, check the logs:

## **Conclusion**

This documentation has outlined the necessary steps for setting up and configuring your web and database servers using Ansible. By following this guide, you have:

- Established secure SSH connectivity between the master node and worker nodes.
- Configured firewalls to allow SSH and HTTP traffic to ensure communication and web access.
- Set up Apache on the web server and MySQL on the database server, confirming their successful installation and operation.

The playbooks, roles, and configurations provided can be customized for additional nodes or different services as needed.
