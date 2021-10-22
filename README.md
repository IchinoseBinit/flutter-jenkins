# Using dockerized container to build signed apk and a web to download the file

## Building the image

The image at the docker hub, ```ichinosebinit/jenkins-flutter-android:latest``` has installed Android SDK, flutter and an image of an Android OS. 

### Steps

* The jenkins image is pulled and flutter is installed.
* The Flutter bin folder is setup in the path.
* The default jdk is installed. 
* Android SDK is fetched and unzipped.
* The contents of the Android SDK is managed, ```moved to a directory``` to be used by the Jenkins.
* The licenses are accepted using the SDK manager.
* The Android OS image is downloaded using the SDK manager.

* The docker file is given here [Docker File for Image creation](/BaseDockerfile.dockerfile).

## Building the container to create and sign APKs 

### Pull the image from the dockerhub using,

``` docker pull ichinosebinit/jenkins-flutter-android:latest ```

### Run the docker image using,

``` docker run -p 8070:8080 -v /D:/data:/var/jenkins_home ichinosebinit/jenkins-flutter-android:latest ```
<!-- docker run -p 8081:8080 -v /home/ubuntu/data:/var/jenkins_home ichinosebinit/jenkins-flutter-android:latest -->

Here, you can replace the port number i.e. 8070 to any one which is not allocated. But do not change the latter port number.
The -v option refers to the mounted directory with the host and the container.
Mount the preferred directory to the left side before ":" and leave the right mounted directories unchanged.

### Run the docker container in the web browser

Open your web browser and open ```http://localhost:8070/ ```. Setup the Jenkins with admin password ```password given in log as initial password``` and install the necessary plugins. 

### Installing necessary plugins

* Browse over to ```Manage Jenkins``` and go to ```Manage Plugins``` page. On the Available tab, search for ```GitLab plugin``` and ```Android Signing Plugin``` and install those plugins. 

### Adding Credentials

* Go back to ```Manage Jenkins```, head over to  ```Manage Credentials``` under the ``` Security``` section.
* Under the ```Stores scoped to Jenkins``` section, click on ```Jenkins``` under the ```Store``` tab.
* Select the ```Global credentials (unrestricted)``` and select ```Add Credentials``` in the left section of the page.

#### Adding GitLab Token

* Select the ```GitLab API Token``` option under the ```Kind``` field.
* Select the ```Global``` option under the ```Scope```.
* Paste the ```API Token``` from the GitLab of your account.
* You can get your ```API Token``` from [Gitlab Documentation](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)
* Give an unique ```ID``` for the Token. 
* Give a description for the Token.

#### Adding GitLab User ID

* Select the ```Username with password``` option under the ```Kind``` field.
* Select the ```Global``` option under the ```Scope```.
* Enter your ```Username``` of your GitLab account.
* Enter your ```Password``` of your GitLab account.
* Give an unique ```ID``` for the account. 
* Give a description for the account.

Note: The Id given here is used in the Jenkins pipeline script. So remember it or copy the ID for future use.

#### Adding a Android Signing Certificate

* Select the ```Certificate``` option under the ```Kind``` field.
* Select the ```Global``` option under the ```Scope```.
* Check the ```Upload PKCS#12 certificate``` radio button.
* Select the ```file``` used for signing the APK.
* Press the ```Change Password``` button.
* Enter the ```password used while creating the certificate```.
* Give an unique ```ID``` for the account. 
* Give a description for the account.

Note: The Id given here is used in the Jenkins pipeline script. So remember it or copy the ID for future use.

### Setting Environment Variables

* Browse over to ```Manage Jenkins``` and go to ```Configure System``` page.
* Head down and Check the ```Environment Variables``` checkbox under the heading ```Global Properties```.
* Click the ```Add``` button.
* Add ```ANDROID_HOME``` as ```Name``` and ```/android-sdk``` as ```Value```.
* Click the ```Add``` button again.
* Add ```HOME``` as ```Name``` and ```/root``` as ```Value```.
* Click the ```Apply``` button

### Setting the GitLab Connection

* Head down on the same page and under the heading ```Gitlab``` check the ```Enable authentication for '/project' end-point``` checkbox.
* Enter the ```Connection Name```.
* Enter ```https://gitlab.com``` as ```Gitlab host URL```.
* Select the ID of the ```GitLab Connection using API Token``` previously saved in credentials.
* Press the ```Test Connection``` button.
* In case of ```Success```, press the ```Save``` button. Else, check for ```permissions``` in your repository.

### Changing the Jenkins file

* Copy and paste the script from the [Jenkinsfile](/jenkins.jenkinsfile) in your editor.
* In the ```stage git```, change the ```branch``` name, ```credentialsId``` previously setup and ```url```.
* In the ```stage sign Android```, change the ```keyStoreId``` previously setup in credentials.
* Use the contents of the file in the next step.

### Creating a new project

* Select ```New Item``` and select ```Pipeline``` 
* Enter the name for your project. Avoid spaces if you can as it may lead to errors while mapping directories. The name for the project would be used later again.
* Check the ```Discard old builds``` and enter 3 in ```Max # of builds to keep```.
* Under the ```GitLab Connection```, select the ```Connection Name``` previously created.
* Under the ```Build Triggers```, select the ```Build when a change is pushed to GitLab. GitLab webhook``` checkbox.
* Check the ```Poll SCM``` checkbox and enter ```* * * * *``` in the ```Schedule``` field.
* Under the ```Pipeline``` section, copy and paste the jenkins script edited previously.
* Press the ```Apply``` and ```Save``` button.

### Building the APK

* Browse the newly created ```job/item```.
* Click the ```Build Now``` option.
* Wait for the ```Build to finish```.