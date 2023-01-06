# openshift-pipeline


Install CRC (Code-Ready Containers) in local environment
1.Go to cloud.redhat.com/openshift/install.
2.Sign in with Red Hat username and password.
3.Select Run on Laptop.
4.Select corresponding Operating System and select Download Code-Ready Containers. This will download a file such as crc-windows-amd64.zip.
5.Select Download pull secret. This will download a file such as pull-secret.txt.
6.Extract (unzip) crc-windows-amd64.zip. This will create a new folder, such as crc-windows-<version>-amd64. Follow the prompts of the installer.
7.Run crc setup and crc start -p pull-secret.text in the directory that pull secret file located.


Install related operators in CRC web console
1.Go to https://console-openshift-console.apps-crc.testing and login with kubeadmin
2.Select OperatorHub under Operator 
3.Search and Install Red Hat OpenShift Pipelines, Red Hat OpenShift GitOps, Nexus Repository Operator respectively



Setup Nexus Repository for a project
1.Create a project with command oc new-project <project-name> if there is no project created
2.Select Installed Operators under Operator and select Nexus Repository Operator
3.Change to created project and click Create instance
4.Edit the environment variable INSTALL4J_ADD_VM_PARAMS as following and click save
  nexus:
    env:
      - name: INSTALL4J_ADD_VM_PARAMS
        value: >-
          -Xms2703m -Xmx2703m -Djava.util.prefs.userRoot=/nexus-data/javaprefs
          -Dnexus.datastore.enabled=true
          -Dnexus.datastore.nexus.jdbcUrl=jdbc:postgresql://postgresql-host:5432/nexusdb
          -Dnexus.datastore.nexus.username=nexus
          -Dnexus.datastore.nexus.password=nexus123
          -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap
          -XX:ActiveProcessorCount=4
5.Select Routes under Networking and click Create Route 
6.Input the following information and click Create
    Name: <route-name>
    Hostname: <host-name>
    Service: select <nexusRepoName-sonatype-nexus-service>
    Target port: 8081 -> 8081(TCP)
7.Wait for status of nexus pod changed to running and browser to Sonatype Nexus Repository Manager with the host name




Setup a self-hosted gitlab server
1.Download ubuntu desktop image file from https://ubuntu.com/download/desktop
2.Follow this website instruction to create vm for ubuntu desktop https://ubuntu.com/tutorials/how-to-run-ubuntu-desktop-on-a-virtual-machine-using-virtualbox#2-create-a-new-virtual-machine
    *remark: at least 4GB for memory and bridged-adapter for network
3.Open command prompt on vm and follow this website to install gitlab
https://linuxhint.com/installing_gitlab_ubuntu/
    *remark: may need to fix locale issue
    https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue



Create a pipeline demo for maven repo
1.Apply yaml files to openshift using command oc create -f <yaml-file-name>.yaml
    1.Clone git resource from TODO
    2.Apply gitlab credentials such as Personal Access Tokens
    3.Add Secret for Service Account pipeline
    4.Apply two tasks: maven-task and git-clone
    5.Apply pipeline: deploy-nexus-pipeline

2.Create webhook for gitops triggering
    1.Click Add Trigger under Action dropdown box in pipeline details 
    2.Choose gitlab-push for Git provider type
    3.Input $(tt.params.git-repo-url) for APP_SOURCE_GIT
    4.Input $(tt.params.git-revision) for APP_SOURCE_REVISION
    5.Input image-registry.openshift-image-registry.svc:5000/demo-dev/$(tt.params.git-repo-name) for APP_IMAGE
    6.Choose PersistentVolumeClaim <nexusRepoName-sonatype-nexus-data> for workspace and maven-settings
    7.Choose Secret <gitlab-secret-name> for git-credentials and click Add

3.Put trigger hook on gitlab repository
    1.Goto gitlab repository -> settings -> webhooks
    2.Input trigger route for URL and Click Add webhook
    3.Test the webhook by clicking push event
