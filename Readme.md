
## Concourse

Now that we've moved away from Circle, here's how we develop our pipeline:


```bash
cd concourse
vagrant up
cd ..
```

grab fly from <http://192.168.100.4:8080> (concourse may take a few minutes to come up locally)
and put the binary in `/usr/local/bin` or somewhere else on your `PATH`

Now we can finish setting up concourse:
```bash
fly -t lite login -c http://192.168.100.4:8080
fly -t lite set-team -n main --basic-auth-username=foo --basic-auth-password=bar
```

Now we can deploy a pipeline! Isn't this what we've always wanted?
```bash
echo -e "concourse_url: 'http://192.168.100.4:8080'\nconcourse_user: foo\nconcourse_pw: bar" > local-concourse-config.yml
fly -t lite set-pipeline -p pgadmin -c pipelines/pgadmin4.yml -l local-concourse-config.yml
fly -t lite unpause-pipeline -p pgadmin
```

**The first time we run** we also need to create a no-op pipeline for the concourse-pipeline resource to be happy:
```bash
touch scratch.yml
fly -t lite set-pipeline -p pgadmin-feature-branches -c scratch.yml --non-interactive
fly -t lite unpause-pipeline -p pgadmin-feature-branches
```

Now go back to [your concourse](http://192.168.100.4:8080) and kick off the `manually-generate-feature-branch-pipeline` job

If that succeeds, the pgadmin-feature-branches pipeline will now contain several jobs
(as specified in `pipelines/feature-branch-pipeline-jobs.yml.erb`) for each branch.

You can find a job named `[your feature branch of interest]-branch` and trigger that.
The corresponding jobs will then run automatically (triggered by new commits pushed to that branch). 

### Create a Docker Image

```bash
$ docker build -t chrome-python-2.7.10 .

$ docker tag chrome-python-2.7.10 joaopapereira/chrome-python-2.7.10

$ docker push joaopapereira/chrome-python-2.7.10
```

### Look at the container while the test running

```bash
$ ./fly -t lite intercept -j pgadmin/Python-3.6-Postgres bash
```


*The following sections may still come in handy as part of concourse setup or for pushing manually?* 

## To push to CF

Push a branch from the `pgadmin4` repo, which will trigger a git hook that updates this repository and it's submodules. 

Then login to CF using the following command:

```bash
$ cf login -a https://api.run.pivotal.io
> USERNAME
> PASSWORD
```

Run the script that will push:

```bash
$ ./push_cf.sh
```


### Push to CF didn't work

If the following message is displayed:

`Not deploying because couldn't find the branch name in the git message`

Check the last git message, it should look like this:

`[branch-name-id] Run tests for.....`

If it doesn't, the commit was not generated by the Git Hook in the `pgadmin4` repo and will not be pushed to CF
