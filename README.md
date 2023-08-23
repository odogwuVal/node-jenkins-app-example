# Deployment strategy
This is a very basic node js application that displays "Hello world"

A continuous integration and continuous deployment (CI/CD) pipeline is a series of steps that must be performed in order to deliver a new version of software. CI/CD pipelines are a practice focused on improving software delivery throughout the software development life cycle via automation. 

By automating CI/CD throughout development, testing, production, and monitoring phases of the software development lifecycle, organizations are able to develop higher quality code, faster. Although it’s possible to manually execute each of the steps of a CI/CD pipeline, the true value of CI/CD pipelines is realized through automation.

I have created a CI/CD pipeline using github action because of its seamingly simplicity with regards to our use case.
The deployment phase involvs packaging the aplication in a docker image and deploying using docker-compose.
I have leveraged Github environments to dynamically require manual approval for deployment to production.
When a change is made to the dev branch, the entire process is automated but a change to the master branch will require a manual approval for deployment to occur

![approvals](review.png)

I have also implemented a script that checks the status of the container to see if it is in a running state and rollback if it isn't

```bash
    ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOSTNAME} '
          sleep 180
          result=$( docker inspect -f {{.State.Running}} } seamlesshr)
          echo $result
          if [$result == "true"]
          then
            echo "rollout is successful"
          else
            sed -i.back "/image:/s/${{ secrets.DOCKER_USERNAME }}\/seamlesshr:.*/${{ secrets.DOCKER_USERNAME }}\/seamlesshr:env.GITHUB_RUN_NUMBER_WITH_OFFSET/g" docker-compose.yaml
            docker-compose up -d
            echo "rollback to previous version complete"
          fi
        '
```