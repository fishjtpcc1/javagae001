# system-build.md

Instructions to build this lifecycle's system platform.

---

to make a system platform having:

    local-terminal (a linux machine eg cloud9 vm)
    librarian (a live remote git core repo eg a github repo)
    tier2 builder (a gcloud compute engine jenkins+java+maven vm job) "jenkins-vm-job001"
    tier2 runner (a gae project) "gae-project"

    
starting with:

    local-terminal (a linux machine eg cloud9 vm)
    librarian [name=lifecycle name]
    gae-project [name=lifecycle name]
  

human method:

    in gae-project:
        make base jenkins-vm (authorize use of google compute engine in gae-project)
    in local-terminal:
        issue the jenkins-vm completion script and follow resulting link to authorize the new running machine
    in jenkins-vm:
        issue the jenkins-vm-job001 completion script
	

jenkins-vm completion script:

    # to download and install gcloud sdk
    curl https://sdk.cloud.google.com | bash
    # get bitnami-image name
    gcloud compute images list --project bitnami-launchpad | grep jenkins
    # create a running admin account on jenkins vm (note you will be jenkins admin user un:user pw:as-set-here) 
    PASSWORD=<password>                # 12 or more chars, with letters and numbers
    PROJECT_ID=<project-id>
    BITNAMI_IMAGE=<bitnami-image>      # e.g. bitnami-jenkins-1-606-0-linux-debian-7-x86-64
    gcloud compute \
        instances create bitnami-jenkins \
        --project ${PROJECT_ID} \
        --image-project bitnami-launchpad \
        --image ${BITNAMI_IMAGE} \
        --zone us-central1-a \
        --machine-type n1-standard-1 \
        --metadata "bitnami-base-password=${PASSWORD}" \
                 "bitnami-default-user=user" \
                 "bitnami-key=jenkins" \
                 "bitnami-name=Jenkins" \
                 "bitnami-url=//bitnami.com/stack/jenkins" \
                 "bitnami-description=Jenkins." \
                 "startup-script-url=https://dl.google.com/dl/jenkins/p2dsetup/setup-script.sh" \
        --scopes "https://www.googleapis.com/auth/userinfo.email" \
               "https://www.googleapis.com/auth/devstorage.full_control" \
               "https://www.googleapis.com/auth/projecthosting" \
               "https://www.googleapis.com/auth/appengine.admin" \
        --tags "bitnami-launchpad"
    # follow returned link to authorize
    

jenkins-vm-job001 completion script:

    // create new job with option:
    from template: freestyle project
    // configure with:
    jenkins toolset (run restriction) = cloud-dev-java
    source manager: git
    source location: <core repos url>
    branch: */master
    source credentials: blank
    build trigger: poll H/5 * * * *
    build, test, and deploy execution steps: mvn gcloud:deploy // ie all in one go for java+maven
  	

References:

* https://cloud.google.com/tools/repo/push-to-deploy

