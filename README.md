CI/CD Overview
----------------

This repo includes a CI/CD pipeline that triggers Jenkins when code is pushed to GitHub. Jenkins builds a Docker image and deploys the containerised app to a Kubernetes cluster.

Files added:
- .github/workflows/trigger-jenkins.yml : GitHub Action to trigger a Jenkins job on push
- Jenkinsfile : Declarative Jenkins Pipeline to build/push image and deploy to k8s
- k8s/deployment.yaml, k8s/service.yaml : Kubernetes manifests

Required setup / secrets
- GitHub Secrets (repository settings):
  - `JENKINS_URL` : Base Jenkins URL, e.g. https://jenkins.example.com
  - `JENKINS_USER` : Jenkins user with API access
  - `JENKINS_API_TOKEN` : API token for `JENKINS_USER`
  - `JENKINS_JOB` : Jenkins job name to trigger (create a pipeline job pointing at this repo)

- Jenkins credentials (configured in Jenkins Credentials store):
  - `docker-hub-creds` : username/password for Docker registry (credentialsId used in `Jenkinsfile`)
  - `kubeconfig-file` : file credential containing kubeconfig for cluster access (credentialsId used in `Jenkinsfile`)

How it works
- GitHub Action triggers on `push` and calls the Jenkins job via the API.
- The Jenkins pipeline (Jenkinsfile) checks out the repo, builds and tags a Docker image as `myorg/cloud_etp:${BUILD_NUMBER}`, pushes it to the configured registry using `docker-hub-creds`, then uses the `kubeconfig-file` credential to update the Kubernetes deployment or apply manifests in `k8s/`.

Next steps / Notes
- Update `Jenkinsfile` environment `IMAGE_NAME` and `REGISTRY` to match your registry and namespace.
- Create the Jenkins pipeline job that uses this repo (Pipeline from SCM) and ensure `JENKINS_JOB` in GitHub secrets points to the job name.
- Ensure the Jenkins instance has Docker and kubectl installed and access to Kubernetes API.
- If using a private container registry other than Docker Hub, set `REGISTRY` accordingly and provide credentials in Jenkins.
