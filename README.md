# Jenkins Test

Minimal repository for testing Jenkins automatic deployment from GitHub.

## What Jenkins Runs

The included `Jenkinsfile` checks out the repository, creates a `dist` folder,
copies `index.html` into it, archives the build artifact, and prints a deploy
message. Replace the deploy stage with your real deployment command when the
pipeline trigger is working.

## Local Smoke Check

Open `index.html` in a browser. If Jenkins is configured with a GitHub webhook,
pushing a new commit to this repository should trigger the pipeline.
