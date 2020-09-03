# Cora: A Wellness Study for COVID-19 related Anxiety

This repository holds the code for the Rasa X deployment of the Cora wellness study.

## Deploy

Deployment notes can be found in [DEPLOY.md](DEPLOY.md)

## Action Server Docker Build

Build image locally:

```
export VERS=0.1.5
docker build -t wacovid/cora-actions:latest -t wacovid/cora-actions:${VERS} .
```

Test locally:

```
docker run -p 5005:5005 wacovid/cora-actions
```

Push to Docker hub:

```
docker login --username=wacovid --password=xyzzy
docker push wacovid/cora-actions
```

View docker hub tags to confirm it has been updated: https://hub.docker.com/r/wacovid/cora-actions/tags

## Runnning Locally

From the command line in separate windows, you can run the following:

```
rasa run actions --debug
rasa shell --conversation-id 206-555-1212
```
