# Dynamically Increment Application Version in Jenkins Pipeline

This project demonstrates how to dynamically increment the application version in a Jenkins pipeline, build a Java application using Maven, and push a Docker image with a dynamic tag to a private DockerHub repository.

## Technologies Used

- Jenkins
- Docker
- GitHub
- Git
- Java
- Maven

## Project Description

This project sets up a Jenkins pipeline to automate the following steps:

1. **Increment Patch Version:** The patch version of the application is dynamically incremented on every build.
2. **Build Java Application and Clean Artifacts:** The Java application is built with Maven, and old artifacts are cleaned before each build.
3. **Build Docker Image with Dynamic Tag:** The Docker image is built using a dynamic tag derived from the version number and build number.
4. **Push Docker Image to Private DockerHub Repository:** The built Docker image is pushed to a private DockerHub repository.
5. **Commit Version Update to Git Repository:** The updated version is committed back to the Git repository to keep track of the version changes.
6. **Prevent Commit Loop:** The pipeline is configured to prevent triggering new builds automatically due to commits made by Jenkins itself.

## Increment Patch Version

In the `Jenkinsfile` of the project, add a stage before the build stage to increment the patch version:

```groovy
stage('increment version') {
    steps {
        script {
            echo "Incrementing app version"
            sh 'mvn build-helper:parse-version versions:set \
               -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
               versions:commit'
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
            env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
    }
}



### Build Java Application and Clean Old Artifacts

Replace the mvn package command in the "Build Application JAR" stage with mvn clean package:

```groovy
stage('build jar') {
    steps {
        script {
            echo 'building the application...'
            sh 'mvn clean package'
        }
    }
}


This ensures that only the latest JAR file is included in the Docker image.

## Build Docker Image with Dynamic Tag and Push to DockerHub

### Set Dynamic Docker Image Tag

In the "build image" stage, replace the hardcoded image version with ${IMAGE_NAME}:

```groovy
stage('build image') {
    steps {
        script {
            echo "building the docker image..."
            withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                sh "docker build -t irschad/java-app:${IMAGE_NAME} ."
                sh 'echo $PASS | docker login -u $USER --password-stdin'
                sh "docker push irschad/java-app:${IMAGE_NAME}"
            }
        }
    }
}


## Commit Version Update to Git Repository

### Add Commit Version Update Stage

After building and pushing the Docker image, add a stage to commit the version update back to the Git repository:

```groovy
stage('commit version update') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'jenkinspush', passwordVariable: 'PAT', usernameVariable: 'USER')]) {
                sh "git remote set-url origin https://${PAT}@github.com/irschad/java-maven-app-versioning.git"
                sh 'git config --global user.email "jenkins@example.com"'
                sh 'git config --global user.name "jenkins"'
                sh 'git add .'
                sh "git commit -m 'ci: version bump'"
                sh 'git push origin HEAD:master'
            }
        }
    }
}


Note: The `jenkinspush` credential used in the above code refers to a Personal Access Token (PAT) generated from GitHub. Here's how it can be created and configured:

1. **Generate the Personal Access Token**:
   - Log in to your GitHub account and navigate to [GitHub Developer Settings - Personal Access Tokens](https://github.com/settings/tokens).
   - Click on **"Generate new token (classic)"**.
   - Select the appropriate **scopes** required for your repository. For example:
     - `repo` scope: Provides full access to private and public repositories.
     - `workflow`: If managing workflows in GitHub Actions.
   - Provide an **expiration date** for security purposes.
   - Click **Generate token** and copy the token (you won't be able to view it again later).

2. **Configure the Token in Jenkins**:
   - Open Jenkins and navigate to **Manage Jenkins > Credentials > System > Global credentials**.
   - Click **Add Credentials**.
   - Select **Kind** as **Username and Password**.
   - For **Username**, use your GitHub username.
   - For **Password**, paste the generated Personal Access Token.
   - Enter **ID** as `jenkinspush` (to match the code) and provide a meaningful description.
   - Save the credentials.

3. **Usage in the Pipeline**:
   - In the pipeline script, the `withCredentials` block fetches the stored `jenkinspush` credentials and substitutes the username and token dynamically into the Git commands. The `PAT` environment variable is used for authentication with GitHub when pushing changes.

By following these steps, Jenkins can securely interact with your GitHub repository without exposing sensitive credentials in your pipeline code.


## Prevent Commit Loop
### Step 1: Install Jenkins Plugin
Install the "Ignore Committer Strategy" plugin to prevent Jenkins from triggering builds on commits made by Jenkins.

### Step 2: Configure the Plugin
In the Jenkins configuration, in Build strategy, set the "Ignore Committer Strategy" to ignore commits from ID configured to be ignored, for example, jenkins@example.com.

![Ignore Committer Strategy](https://github.com/user-attachments/assets/da1a61a7-d9ea-44c7-8557-e0fd3e854349)


Conclusion:
This Jenkins pipeline automates the process of dynamically incrementing the application version, building the Java application, and pushing a Docker image with a dynamic tag to DockerHub, all while ensuring that version updates are tracked and preventing a commit loop.

---
