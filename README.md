# Jenkins Test

Minimal repository for testing Jenkins automatic deployment from GitHub.

## What Jenkins Runs

The included `Jenkinsfile` checks out the repository, creates a `dist` folder,
copies `index.html` into it, starts or reuses a Docker nginx container named
`jenkins-test-nginx`, and copies the built files into nginx.

The deployed page is available at:

```text
http://localhost:8081/
```

## Local Smoke Check

Start Docker Desktop, then run the Jenkins pipeline. If Jenkins is configured
with a GitHub webhook, pushing a new commit to this repository should trigger
the pipeline and refresh the nginx page.

The pipeline expects Jenkins to be able to run Docker commands on the machine
where the job executes.
