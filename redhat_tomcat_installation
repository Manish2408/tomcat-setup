Here's a step-by-step guide to install Java 17, set up Apache Tomcat with server manager and manager host access, and configure Apache (httpd) as a reverse proxy on your Red Hat server:

---

### 1. **Update the System**
Ensure the system is up-to-date:
```bash
sudo yum update -y
```

---

### 2. **Install Java 17**
Install Java 17 (OpenJDK):
```bash
sudo yum install -y java-17-openjdk java-17-openjdk-devel
```

Verify the installation:
```bash
java -version
```

---

### 3. **Download and Install Apache Tomcat**
1. Download the latest stable version of Tomcat (9.x or 10.x):
   ```bash
   wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.33/bin/apache-tomcat-10.1.33.tar.gz
   ```

2. Extract the Tomcat package:
   ```bash
   sudo tar -xvzf apache-tomcat-10.1.33.tar.gz -C /opt/
   ```

3. Rename the folder for simplicity:
   ```bash
   sudo mv /opt/apache-tomcat-10.1.33 /opt/tomcat
   ```

4. Set appropriate permissions:
   ```bash
   sudo chmod -R 755 /opt/tomcat
   sudo chown -R $(whoami):$(whoami) /opt/tomcat
   ```

---

### 4. **Configure Tomcat Server**
1. Edit `server.xml` to configure ports:
   ```bash
   sudo nano /opt/tomcat/conf/server.xml
   ```
   Ensure the following configurations are present:
   - **HTTP Connector**: Default port is 8080.
   - **AJP Connector**: Uncomment and configure if needed.

2. Enable the **Manager App** and **Host Manager**:
   - Edit `tomcat-users.xml`:
     ```bash
     sudo nano /opt/tomcat/conf/tomcat-users.xml
     ```
     Add the following:
     ```xml
     <role rolename="manager-gui"/>
     <role rolename="admin-gui"/>
     <user username="admin" password="your-password" roles="manager-gui,admin-gui"/>
     ```

---

### 5. **Start Tomcat**
1. Create a script to manage Tomcat:
   ```bash
   sudo nano /etc/systemd/system/tomcat.service
   ```
   Add the following:
   ```ini
   [Unit]
   Description=Apache Tomcat Web Application Container
   After=network.target

   [Service]
   Type=forking

   Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk
   Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
   Environment=CATALINA_HOME=/opt/tomcat
   Environment=CATALINA_BASE=/opt/tomcat
   ExecStart=/opt/tomcat/bin/startup.sh
   ExecStop=/opt/tomcat/bin/shutdown.sh

   User=root
   Group=root

   [Install]
   WantedBy=multi-user.target
   ```

2. Reload systemd and enable Tomcat:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable tomcat
   sudo systemctl start tomcat
   ```

3. Verify Tomcat is running:
   ```bash
   sudo systemctl status tomcat
   ```

---

### 6. **Install Apache HTTP Server**
Install Apache (httpd):
```bash
sudo yum install -y httpd
```

Start and enable Apache:
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

### 7. **Configure Reverse Proxy**
1. Enable required modules:
   ```bash
   sudo dnf install -y mod_proxy mod_proxy_http
   ```

2. Edit the Apache configuration:
   ```bash
   sudo nano /etc/httpd/conf.d/tomcat.conf
   ```

3. Add the following configuration:
   ```apache
   <VirtualHost *:80>
       ServerName your-domain.com

       ProxyPreserveHost On
       ProxyPass / http://localhost:8080/
       ProxyPassReverse / http://localhost:8080/

       ErrorLog /var/log/httpd/tomcat_error.log
       CustomLog /var/log/httpd/tomcat_access.log combined
   </VirtualHost>
   ```

4. Restart Apache:
   ```bash
   sudo systemctl restart httpd
   ```

---

### 8. **Verify Configuration**
1. Access Tomcat through your browser:
   - For Manager App: `http://your-domain.com/manager`
   - For Host Manager: `http://your-domain.com/host-manager`

2. Troubleshoot logs:
   - Tomcat logs: `/opt/tomcat/logs/`
   - Apache logs: `/var/log/httpd/`

---

### 9. **Firewall Configuration**
Ensure ports 80 (HTTP) and 8080 (Tomcat) are open:
```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

Let me know if you need further assistance!
