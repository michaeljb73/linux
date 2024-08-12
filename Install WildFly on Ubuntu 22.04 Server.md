# How to Install WildFly JAVA Application Server on Ubuntu 22.04

March 28, 2024 by [Hitesh Jethva](https://vitux.com/author/hitesh-jethva/)

WildFly, formerly JBoss, is a free and open-source application server that helps you build and deploy Java web applications. It's written in Java and compliant with Java EE (Enterprise Edition) specifications. WildFly was created by JBoss but is now developed by RedHat. It is a powerful, production-ready, modular, and lightweight application server that provides all necessary tools and features to run and deploy Java web applications.

WildFly is a cross-platform application server. It provides tools such as EJBs, JPA, Servlets, JAX-RS, Batch, and security for your Java applications.

This guide will help you install WildFly on the Ubuntu 22.04 server. You will manually install WildFly from the Binary package on the Ubuntu system.

## Prerequisites

To complete this tutorial, you will need a **Ubuntu 22.04** Server. This server should have a non-root user with root/administrator privileges. Additionally, it's recommended that you enable the "**UFW**" firewall on your Ubuntu system.

## Installing Java OpenJDK

Before starting to install WildFly, you will install the Java OpenJDK first on your Ubuntu server. At the time of this writing, the latest version of WildFly required at least Java v8. The default Ubuntu APT repository provides Java v11, which is suitable for the latest version of WildFly installation.

Run the "*apt*" command below to update and refresh your Ubuntu package index.

```
sudo apt update
```

To install Java OpenJDK, run the "*apt install*" command below. You will be prompted to confirm the installation, input **Y**, and press **ENTER** to continue, and the Java OpenJDK installation will begin.

```
sudo apt install default-jdk
```

![install java](https://vitux.com/wp-content/uploads/image-6127-1-install-java.png)

Now that Java OpenJDK is installed, check and verify the Java version using the following command. You should see that **Java OpenJDK 11** is installed on your Ubuntu 22.04 server.

```
java -version
```

![chek java version](https://vitux.com/wp-content/uploads/image-6127-2-check-java.png)

## Installing WildFly from Binary Files

After installing Java, you will install WildFly manually on your Ubuntu server. The manual WildFly installation requires you to set up a new system user, download the WildFLy Binary Package, and set up the configuration directory and systemd service for WildFly.

To create a new system user, run the "*useradd*" command below.

In the following example, the new system user will be named "**wildfly**" with the base home directory "*/opt/wildfly*". This directory also will be used as the main installation directory for WildFly.


```
sudo useradd -r -d /opt/wildfly -s /usr/sbin/nologin wildfly
```

Before downloading WildFly binary package, run the following command to create a new temporary environment variable "**WILDFLY_RELEASE**". This will assign the environment variable "**WILDFLY_RELEASE**" with the latest release of the WildFly version. At the time of this writing, the latest version of WildFly is **v26.x**.

```
WILDFLY_RELEASE=$(curl -s https://api.github.com/repos/wildfly/wildfly/releases/latest|grep tag_name|cut -d '"' -f 4)
echo $WILDFLY_RELEASE
```

To download the WildFly Binary Package, run the "*wget*" command below. You should see the new file in your current directory.

```
wget https://github.com/wildfly/wildfly/releases/download/${WILDFLY_RELEASE}/wildfly-${WILDFLY_RELEASE}.tar.gz
```

![Download WildFly](https://vitux.com/wp-content/uploads/image-6127-3-download-wildfly.png)

Next, extract the WildFLy package using the "*tar*" command below. Then, move the new extracted WildFly directory to *"/opt/wildfly"*.



```
tar -xvf wildfly-${WILDFLY_RELEASE}.tar.gz
sudo mv wildfly-${WILDFLY_RELEASE} /opt/wildfly
```

Set the correct and appropriate ownership of the WldFly installation directory to the user "**wildfly**" using the below command.

```
sudo chown -R wildfly:wildfly /opt/wildfly
```

After that, create a new default configuration directory "*/etc/wildfly*" using the following command. Then, copy the example of the WildFly configuration file into it.

```
sudo mkdir -p /etc/wildfly/
sudo cp sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/
```

Edit the default WildFly config file "*/etc/wildfly/wildfly.conf*" using the below command.

```
sudo nano /etc/wildfly/wildfly.conf
```

You can change the "**WILDFLY_BIND**" address to your local server IP address. Also, in this demonstration, you will be running WildFly in the "**standalone**" mode with the config file "**standalone.xml**".

```
# The configuration you want to run
WILDFLY_CONFIG=standalone.xml
# The mode you want to run
WILDFLY_MODE=standalone

# The address to bind to
WILDFLY_BIND=192.168.5.10
```

Save and close the file when you are done.

Next, copy the example of WildFly start script *"/opt/wildfly/docs/contrib/scripts/systemd/launch.sh*" to the directory "*/opt/wildfly/bin/*". Then, copy the WildFly systemd service file "*/opt/wildfly/docs/contrib/scripts/systemd/wildfly.service*" to the directory *"/usr/lib/systemd/system/"*.

```
sudo cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/
sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /usr/lib/systemd/system/
```

![configure wildfly](https://vitux.com/wp-content/uploads/image-6127-4-configure-wildfly.png)

Additionally, you will also need to create a new directory "*/var/run/wildfly*" that will be used for storing the PID (Process ID) file of the WildFly server. Also, be sure to change the correct ownership of that directory to the user "**wildfly**".

```
sudo mkdir /var/run/wildfly/
sudo chown -R wildfly:wildfly /var/run/wildfly/
```

Now run the "*systemctl*" command below to reload the systemd manager and apply the new WildFly service file.

```
sudo systemctl daemon-reload
```

After that, start and enable the "*wildfly*" service using the following "*systemctl*" command. And the "*wildfly*" service will be up and running, and it will be run automatically at system boot.

```
sudo systemctl start wildfly
sudo systemctl enable wildfly
```

![setup wildfly as systemd service](https://vitux.com/wp-content/uploads/image-6127-5-setupl-wildfly-systemd-service.png)

To check and verify the "*wildfly*" service status, run the below command. And you should see the "wildfly" service is "*enabled*" and the current status is 'running".

```
sudo systemctl status wildfly
```

![check wildfly service](https://vitux.com/wp-content/uploads/image-6127-6-check-wildfly-service.png)

By default, WildFly runs on the TCP port "8080." You must open that port if you have the "UFW" firewall enabled on your server.

Run the "*ufw*" command below to open the TCP port "*8080*" for WildFly. Then, check and verify the UFW firewall status. You should see that port "8080" has been added.

```
sudo ufw allow 8080/tcp
sudo ufw status
```

![ufw setup for wildfly](https://vitux.com/wp-content/uploads/image-6127-7-ufw-wildfly.png)

Open your web browser and visit the server IP address with the port "8080" (e.g., http:/192.168.5.10:8080/). You will see WildFly's default index.html page.

![wildfly default index](https://vitux.com/wp-content/uploads/image-6127-8-wildfly-default-index.png)

## Setting Up Administration User WildFly

The WildFly administration is disabled on the default installation. In this demo, you will enable the WildFly Administration dashboard that allows you to set up application deployment, runtime configuration, access control, and other WildFly settings.

You will set up a new admin user for WildFly before you start configuring and enabling the WildFly administration.

Run the WildFly add user script *"/opt/wildfly/bin/add-user.sh"* as "**wildfly**" user below.

```
sudo -u wildfly /opt/wildfly/bin/add-user.sh
```

Now you will be prompted for the admin user configuration below:

- Input "**a**" to set up the WildFly "**Management User**".
- Input the username for the WildFly administrator. In this demo, we will use the admin user "**alice**".
- Input the password for the new admin user and repeat. Be sure to use the strong password as recommended by the script.
- Leave the group configuration as default and press **ENTER**.
- Input "**yes**" to confirm the new WildFly admin configurations.
- Lastly, input "**yes**" again to enable the new user to be used to identify one WildFly process or you can just input "no" for this settings.

![create wildfly admin user](https://vitux.com/wp-content/uploads/image-6127-11-setup-wildfly-user.png)

## Enabling WildFly Administration Console

If you created the WildFly administrator user, now you will be setting up the WildFly Administration console via the configuration file "*/etc/wildfly/wildfly.conf*".

Edit the config file "*/etc/wildfly/wildfly.conf*" using the following command.

```
sudo nano /etc/wildfly/wildfly.conf
```

Add the following configuration to the file. This will create a new environment variable for the WildFly service named "**WILDFLY_CONSOLE_BIND**", which will run the WildFly administration console on the internal IP address "**192.168.5.10**".

```
# IP Address for WildFly Console
WILDFLY_CONSOLE_BIND=192.168.5.10
```

Save and close the file when you are done.

Next, edit the WildFly start script *"/opt/wildfly/bin/launch.sh*" using the following command.

```
sudo nano /opt/wildfly/bin/launch.sh
```

Change the start command as below. Add option "*-bmanagement $4*" to enable the WildFly administration console.

```
if [[ "$1" == "domain" ]]; then
    $WILDFLY_HOME/bin/domain.sh -c $2 -b $3 -bmanagement $4
else
    $WILDFLY_HOME/bin/standalone.sh -c $2 -b $3 -bmanagement $4
fi
```

Save and close the file when you are done.

Now edit the WildFly systemd service file "*/usr/lib/systemd/system/wildfly.service*" using the following command.

```
sudo nano /usr/lib/systemd/system/wildfly.service
```

Add the environment variable "**$WILDFLY_CONSOLE_BIND**" as an additional command option on the "*ExecStart*" as the following.

```
ExecStart=/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND $WILDFLY_CONSOLE_BIND
```

Save and close the file when you are done.

After editing the WildFly systemd service file, run the "*systemctl*" command below to reload the systemd manager. Then, restart the "*wildfly*" service to apply new changes and enable the administration console.

```
sudo systemctl daemon-reload
sudo systemctl restart wildfly
```

Now check and verify the "*wildfly*" service using the "*systemctl*" command below. And you should get the "*wildfly*" service is "**running**". On the start script "*launch.sh*" and "*standalone.sh*" you will see additional options for enabling the Administration console.

```
sudo systemctl status wildfly
```

![enable wildfly admin console](https://vitux.com/wp-content/uploads/image-6127-9-enabled-wildfly-admin.png)

The default WildFly administration is running on the TCP port "*9990*". To enable access to it, you will need to add the UFW firewall rule.

Run the "*ufw*" command below to add the UFW firewall rule for opening the WildFly administration console port "*9990*". Then, check and verify the UFW firewall status.

```
sudo ufw allow 9990/tcp
sudo ufw status
```

You will see an output similar to the following screenshot.

![wildfly admin ufw firewall](https://vitux.com/wp-content/uploads/image-6127-10ufw-wildfly-admin.png)

Lastly, back to your web browser and visit the server IP address followed by the port "*9990*" (i.e: http://192.168.5.10:9990/). And you will be prompted for the WildFly admin user and password.

Input your WildFly admin user and password, then click the "**Sign In**" button.

![wildfly basic auth](https://vitux.com/wp-content/uploads/image-6127-12-wildfly-basic-auth.png)

If your user and password are correct, you will see the WildFly Administration Console page below.

![WildFly admin dashboard](https://vitux.com/wp-content/uploads/image-6127-13-wildfly-admin-dashboard.png)

## Deploying Application to WildFly

Now you will deploy the "Hello World" example application to WildFly. You can deploy an application from the WildFly administration console or using the command-line interface mode.

Create a new directory *"/opt/testapp"* and download the example application "*helloworld.war*" using the "*curl*" command below.

```
mkdir -p /opt/testapp/
curl --output /opt/testapp/helloworld.war https://raw.githubusercontent.com/aeimer/java-example-helloworld-war/master/dist/helloworld.war
```

![download sample app](https://vitux.com/wp-content/uploads/image-6127-15-download-testapp-wildfly.png)

Next, run the script "*jboss-cli.sh*" to connect to your WildFly administration from the command line.

```
sudo -u wildfly /opt/wildfly/bin/jboss-cli.sh --connect
```

After you connected to the WildFly shell, run the following command to deploy the example application "*/opt/testapp/helloworld.w*ar" to WildFly.

```
deploy /opt/testapp/helloworld.war
```

Lastly, back to your web browser and visit your server IP address with the following path "*/helloworld*" (i.e: http://192.168.5.10:8080/helloworld/). And you should get the example application "*helloworld.war*".

![deploy java app to wildfly](https://vitux.com/wp-content/uploads/image-6127-14-deploy-app-to-wildfly.png)

## Conclusion

Congratulation! You have now successfully installed the WildFly on Ubuntu 22.04 server using the manual installation from Binary Package. You have also created the administrator user for WildFly and enabled the web-based Administration Console for WildFly. In the end, you have also learned how to deploy an application to WildFly from the command-line interface.