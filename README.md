# pythonapps-ci
Continuous Integration using Jenkins to build Python applications running in GKE

# 1) Execute Shell

# Delete current project
`rm -rf /var/lib/jenkins/workspace/yourproject`

## Global variables
```
export PROJECT_ID=yourgkeprojectid
export GCLOUD_CLUSTER_NAME=yourgkeclustername
export CHART_VERSION=yourgkeversionchart
export PATH_PROJECT=yourproject
export FULL_PATH=/var/lib/jenkins/workspace/yourproject
export KUBECONFIG=$FULL_PATH/$PROJECT_ID-kubeconfig
```

## Chart variable - TASKS
`export CHART_NAME=yourchartname`

## Download project
`git clone https://git.example.com/yourproject.git`

## Setting permission to jenkins user
`chown -R jenkins:jenkins $FULL_PATH/$PATH_PROJECT`

## Open project directory
cd yourproject

## Gcloud authentication & Docker Build API
gcloud config set project yourproject
gcloud auth activate-service-account docker@${PROJECT_ID}.iam.gserviceaccount.com --key-file=$FULL_PATH/$PROJECT_ID-cloudbuild.json
docker build -t us.gcr.io/yourproject/yourapplication:latest -t us.gcr.io/yourproject/yourapplication:$BUILD_NUMBER -f deploy/dockerfiles/Dockerfile.yourapplication.prod .
gcloud docker -- push us.gcr.io/yourproject/yourapplication:latest
gcloud docker -- push us.gcr.io/yourproject/yourapplication:$BUILD_NUMBER
gcloud container clusters get-credentials $GCLOUD_CLUSTER_NAME --zone=yourzone

## Chart local variables
export CHART_FULLNAME=${CHART_NAME}-${CHART_VERSION}
export CHART_FILENAME=$FULL_PATH/${CHART_FULLNAME}.tgz
export HELM_NAME=$(helm ls | grep $CHART_FULLNAME | awk '{print $1}')
/usr/local/bin/helm upgrade $HELM_NAME $CHART_FILENAME --set image.tag=$BUILD_NUMBER
