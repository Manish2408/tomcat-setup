Here’s a step-by-step guide to install **Java 17**, **Tomcat**, configure **Server Manager and Host Manager**, and set up a **reverse proxy** with Apache on your Ubuntu server:

---

### **1. Install Java 17**
1. **Update the System:**
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Install OpenJDK 17:**
   ```bash
   sudo apt install openjdk-17-jdk -y
   ```

3. **Verify the Installation:**
   ```bash
   java -version
   ```

   Output should display:
   ```
   openjdk version "17.x.x"
   ```

---

### **2. Install Tomcat**
1. **Create a Tomcat User:**
   ```bash
   sudo groupadd tomcat
   sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
   ```

2. **Download Tomcat:**
   Visit the [Tomcat download page](https://tomcat.apache.org/download-10.cgi) to find the latest version. Use `wget` to download:
   ```bash
   wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.0/bin/apache-tomcat-10.1.0.tar.gz
   ```

3. **Extract and Configure Tomcat:**
   ```bash
   sudo mkdir /opt/tomcat
   sudo tar -xvzf apache-tomcat-10.1.0.tar.gz -C /opt/tomcat --strip-components=1
   ```

4. **Set Permissions:**
   ```bash
   sudo chown -R tomcat:tomcat /opt/tomcat
   sudo chmod -R 755 /opt/tomcat
   ```

---

### **3. Configure Tomcat Service**
1. **Create a Systemd Service File:**
   ```bash
   sudo nano /etc/systemd/system/tomcat.service
   ```

2. **Add the Following Content:**
   ```ini
   [Unit]
   Description=Apache Tomcat Web Application Container
   After=network.target

   [Service]
   Type=forking

   Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
   Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
   Environment=CATALINA_HOME=/opt/tomcat
   Environment=CATALINA_BASE=/opt/tomcat
   Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
   Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

   ExecStart=/opt/tomcat/bin/startup.sh
   ExecStop=/opt/tomcat/bin/shutdown.sh

   User=tomcat
   Group=tomcat
   UMask=0007
   RestartSec=10
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

3. **Reload and Start the Service:**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start tomcat
   sudo systemctl enable tomcat
   ```

---

### **4. Enable Manager and Host-Manager Apps**
1. **Edit `context.xml`:**
   Remove access restrictions in:
   ```bash
   sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
   sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
   ```
   Comment out or remove the `<Valve>` that restricts access:
   ```xml
   <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1"/> -->
   ```

2. **Add a User in `tomcat-users.xml`:**
   Edit the `tomcat-users.xml` file:
   ```bash
   sudo nano /opt/tomcat/conf/tomcat-users.xml
   ```
   Add the following:
   ```xml
   <role rolename="manager-gui"/>
   <role rolename="admin-gui"/>
   <user username="admin" password="password" roles="manager-gui,admin-gui"/>
   ```

3. **Restart Tomcat:**
   ```bash
   sudo systemctl restart tomcat
   ```

---

### **5. Set Up Reverse Proxy with Apache**
1. **Install Apache and Required Modules:**
   ```bash
   sudo apt install apache2 -y
   sudo a2enmod proxy proxy_http
   ```

2. **Configure the Reverse Proxy:**
   Create a new Apache virtual host configuration:
   ```bash
   sudo nano /etc/apache2/sites-available/tomcat.conf
   ```

   Add the following content:
   ```apache
   <VirtualHost *:80>
       ServerName yourdomain.com

       ProxyPreserveHost On
       ProxyPass / http://localhost:8080/
       ProxyPassReverse / http://localhost:8080/

       ErrorLog ${APACHE_LOG_DIR}/tomcat_error.log
       CustomLog ${APACHE_LOG_DIR}/tomcat_access.log combined
   </VirtualHost>
   ```

3. **Enable the Configuration:**
   ```bash
   sudo a2ensite tomcat.conf
   sudo systemctl restart apache2
   ```

---

### **6. Test the Setup**
1. Access Tomcat in the browser:
   - Manager App: `http://yourdomain.com/manager`
   - Host Manager App: `http://yourdomain.com/host-manager`

2. Check the reverse proxy works by visiting `http://yourdomain.com`.

---

### **7. Security Best Practices**
- **Secure Tomcat:** Use HTTPS with a valid SSL certificate. Configure SSL in Apache.
- **Restrict Access:** Configure a firewall to limit access to sensitive endpoints.
- **Harden the Server:** Disable default applications and configure strong passwords.

Let me know if you encounter any issues!
