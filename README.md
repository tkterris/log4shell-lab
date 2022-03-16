# log4shell-lab: Test Log4Shell mitigation via Deployment Overlays

Author: Trevor Terris  
Level: Intermediate  
Technologies: EJB, JSF, WAR  
Summary: Small sample application to validate log4j mitigation via JBoss EAP deployment overlays  
Target Product: EAP  
Source: <https://github.com/tkterris/log4shell-lab.git>  

## WARNING

As-is, this application is vulnerable to CVE-2021-44228! DO NOT RUN THIS IN PRODUCTION, this is intended for
educational purposes only!

## What is it?

This example application is vulnerable to the "Log4Shell" vulnerability, CVE-2021-44228.

The example follows the common "Hello World" pattern. These are the steps that occur:

1. A JSF page asks the user for their name.
1. On clicking submit, the name is sent to a managed bean named `Greeter`.
1. The name is printed to log output via log4j, using a library version vulnerable to Log4Shell.

## Setup

### System requirements

The application this project produces is designed to be run on Red Hat JBoss Enterprise Application Platform 7.4 or later. 

### Configure Maven 

If you have not yet done so, you must [Configure Maven](../README.md#configure-maven) before testing the quickstarts.

### Start the JBoss Server

1. Open a command line and navigate to the root of the JBoss server directory.
2. The following shows the command line to start the server:

        For Linux:   JBOSS_HOME/bin/standalone.sh
        For Windows: JBOSS_HOME\bin\standalone.bat

### Configure and start the Huntress Vulnerability Tester

First, install and start Redis (using the default IP address and port):

```
sudo yum install redis
sudo systemctl start redis
```

Then download, build, and run the Huntress vulnerability tester:

```
git clone https://github.com/huntresslabs/log4shell-tester.git
cd log4shell-tester/
mvn clean install
java -jar target/log4shell-jar-with-dependencies.jar
```

The vulnerability tester will be available at <http://127.0.0.1:8000>.

### Build and Deploy the lab

_NOTE: The following build command assumes you have configured your Maven user settings. If you have not, you must include Maven setting arguments on the command line. See [Build and Deploy the Quickstarts](../README.md#build-and-deploy-the-quickstarts) for complete instructions and additional options._

1. Open a command line and navigate to the root directory of this quickstart.
1. Type this command to build and deploy the archive: `mvn clean install`
1. Deploy the compiled "log4shell-lab.war" artifact to the EAP server.

## Running the lab

### Access the application 

The application will be running at the following URL: <http://localhost:8080/log4shell-lab>.


### Investigate the Log4Shell vulnerability

1. Fetch a test payload from the Log4Shell tester page here: <https://127.0.0.1:8000/>. For example: `${jndi:ldap://127.0.0.1:1389/GENERATED_TOKEN_HERE}`
1. In the JSF page located at `http://localhost:8080/log4shell-lab`, enter the Huntress Log4Shell payload obtained in the previous step.
1. View the results for that payload, as described on the Huntress Log4Shell tester page. The URL will be formatted like so: <https://127.0.0.1:8000/view/GENERATED_TOKEN_HERE>. If the application is vulnerable, the test payload will result in an LDAP request sent to the Huntress page, which will be logged on the results page.

### Mitigation via Deployment Overlays

This issue can be mitigated in JBoss EAP without recompiling the artifact via [Deployment Overlays](https://access.redhat.com/solutions/383393).

1. Download a version of the `log4j-core` and `log4j-api` library JARs that have had the Log4Shell vulnerability patched. These JARs can be accessed [here](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.17.1/log4j-core-2.17.1.jar) and [here](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-api/2.17.1/log4j-api-2.17.1.jar).
1. In the JBoss CLI, execute the following command to overlay log4j-core: `deployment-overlay add --name=log4shellOverlay --content=/WEB-INF/lib/log4j-core-2.11.2.jar=/path/to/patched/log4j-core.jar --deployments=log4shell-lab.war --redeploy-affected`
1. In the JBoss CLI, execute the following command to overlay log4j-api: `deployment-overlay add --name=log4shellApiOverlay --content=/WEB-INF/lib/log4j-api-2.11.2.jar=/path/to/patched/log4j-api.jar --deployments=log4shell-lab.war --redeploy-affected`
1. Rerun the investigation steps. There should be no additional requests listed on the Huntress results page, confirming that the application is not longer vulnerable.

## Credits

- [EAP Quickstarts](https://github.com/jboss-developer/jboss-eap-quickstarts/)
- [Huntress Log4Shell Vulnerability Tester](https://log4shell.huntress.com/)
