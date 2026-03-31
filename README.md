# jenkins-sonarqube-nexus-tomcat

## TOMCAT SETUP

## 🔹 Installation
- Tomcat was installed on the dedicated EC2 instance (`tomcat`).
- The server was started manually using the extracted binaries and the `startup.sh` script.

## 🔹 Verification
- Tomcat UI was successfully accessed at:http://184.72.215.120:8080/

![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/7d40438ff38292749dbd2ffe6a7aa7ba9fcf235e/tomcat%20access.png)
# SONARQUBE SETUP

## 🔹 Installation
- SonarQube was installed and started on the dedicated EC2 instance (`sonar`).
- The service was launched using the provided binaries and startup script.

## 🔹 Access URL
- SonarQube UI accessible at:  http://44.220.157.36:9000/
  ![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/e32b486cc91d896d45177a70720e55f99ca67e4e/Screenshot%202026-03-31%20105606.png)
# TESTING & VERIFICATION


## 🔹 Configuration Steps
- Logged into the SonarQube UI  
- Created project: **maven-web-app**  
- Generated a token/secret for Jenkins integration  
- Stored the token securely in Jenkins credentials 
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/92355ee317e75df2b36a106ea9e9837fd111c2e0/Screenshot%202026-03-31%20111315.png)
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/3de8ed5d0ab5f4c36a45b89cdd9732d93c64790f/Screenshot%202026-03-31%20111633.png)
## NEXUS SETUP

## 🔹 Installation
- Nexus Repository Manager was installed and started on the dedicated EC2 instance (`nexus`).
- The service was launched using the extracted binaries and startup script.

## 🔹 Access URL
- Nexus UI accessible at:  http://34.229.76.233:8081/

## 🔹 Configuration Steps
- Logged into the Nexus UI  
- Created Maven (hosted) repositories:  
- **scopeindia-snapshot-repository** → for snapshot builds  
- **scopeindia-release-repository** → for release builds  
- Confirmed repositories are available for publishing artifacts from Jenkins.
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/7b2495c7cd782e2a11caf48aa6a232be3a718046/Screenshot%202026-03-31%20111948.png)

## JENKINS SETUP

## 🔹 Installation
- Jenkins was installed and started on the dedicated EC2 instance (`jenkins`).

## 🔹 Access URL
- Jenkins UI accessible at:  http://34.201.151.8:8080/
  
## 🔹 Plugins Used
- **SonarQube Scanner for Jenkins** → integrates SonarQube analysis into the pipeline  
- **Nexus Artifact Uploader** → publishes build artifacts to Nexus repositories  
- **SSH Agent** → enables secure deployment to remote servers (e.g., Tomcat)

## 🔹 Global Tool Configuration
- Navigate to: **Manage Jenkins → Global Tool Configuration**  
- Configured Maven installation with the name: **Maven3**

## 🔹 Credentials Stored in Jenkins
- **Nexus credentials** → `Id: nexus-credentials`  
- **Tomcat SSH key** → `Id: Tomcat-Server-Agent`  
- **SonarQube token** → `Id: sonar`

### Purpose
- Nexus credentials: Used by Jenkins to upload artifacts (.war) into Nexus repositories.  
- Tomcat SSH key: Enables Jenkins to securely connect and deploy applications to the Tomcat server.  
- SonarQube token: Allows Jenkins to authenticate with SonarQube for static code analysis.
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/b5a6c376c763189f283201862711b5119e125e88/Screenshot%202026-03-31%20112241.png)
## SONARQUBE INTEGRATION WITH JENKINS

## 🔹 Configuration in Jenkins
- Navigate to: **Manage Jenkins → System → SonarQube Servers**
- Configured server name: **sonar**
- Added SonarQube server URL and token credential (previously created in SonarQube and stored in Jenkins credentials)
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/0b0cdce009db8a34e9dca49dfaa69bc63037ce71/Screenshot%202026-03-31%20112636.png)
## 🔹 Maven Settings Requirement
For `mvn sonar:sonar` to work in the pipeline, you need a proper Maven settings file:

**Path:** `/var/lib/jenkins/.m2/settings.xml`


<settings>
  <pluginGroups>
    <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
  </pluginGroups>
</settings>


## NEXUS INTEGRATION WITH JENKINS

## 🔹 Purpose
- Upload build artifacts (.war, .jar) to Nexus repository for versioned storage.

## 🔹 Configuration in Jenkins
- Configured in Jenkins pipeline using the **Nexus Artifact Uploader** plugin.
- Credentials stored in Jenkins: `nexus-credentials`.
- Repository setup:
  - **scopeindia-release-repository** → for release builds
  - **scopeindia-snapshot-repository** → for snapshot builds

---

## TOMCAT INTEGRATION WITH JENKINS

## 🔹 Purpose
- Securely copy `.war` files from Jenkins workspace to the Tomcat server for deployment.

## 🔹 Configuration in Jenkins
- Deployment handled via Jenkins pipeline using **SSH Agent + SCP**.
- Credentials stored in Jenkins: `Tomcat-Server-Agent`.
- WAR file copied to Tomcat `webapps/` directory on EC2 instance.
# TESTING & VERIFICATION
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/3f3dad7e96374cb728e67e181dfddb89582fbfa3/Screenshot%202026-03-31%20114335.png)
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/a8c0748e8f8ca84ecf74afacb549ebef732f9ac7/Screenshot%202026-03-31%20114630.png)
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/3b999dc431c0d0a48d1202d4a054d7bb48b082c3/Screenshot%202026-03-31%20114745.png)
# NEXUS UPLOAD – SNAPSHOT VS RELEASE

## 🔹 Current Pipeline
- By default, the pipeline uploads artifacts to the **release repository**.  
- Example: `scopeindia-release-repository` with version `1.2`.

## 🔹 Switching to Snapshot Upload
To upload to the **snapshot repository**, update the Nexus upload stage values:

stage('Nexus Upload - Snapshot') {
    steps {
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '34.229.76.233:8081',
            groupId: 'org.scopeindia',
            version: '1.2-SNAPSHOT',
            repository: 'scopeindia-snapshot-repository',
            credentialsId: 'nexus-credentials',
            artifacts: [[artifactId: 'maven-web-app', classifier: '', file: 'target/maven-web-app.war', type: 'war']]
        )
    }
}
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/ddd666e629dbdac9f5ceaf5c4e06313d3c4f5327/Screenshot%202026-03-31%20115358.png)
# APPLICATION DEPLOYED ON TOMCAT

## 🔹 Deployment Process
- The WAR file (`maven-web-app.war`) was copied into Tomcat’s `webapps/` directory on the EC2 instance.
- Verified deployment by listing contents of the `webapps/` folder:
![image alt](https://github.com/Jenifa68jeni/jenkins-sonarqube-nexus-tomcat/blob/6ee17dbc4ee3d7dd6c8395d38c273c89a2b32be9/Screenshot%202026-03-31%20115708.png)






