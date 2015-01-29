# Pushmanager

[![Build Status](https://travis-ci.org/Yelp/pushmanager.png?branch=master)](https://travis-ci.org/Yelp/Testify)

Pushmanager is a tornado web application we use to manage deployments at Yelp.
It helps pushmasters to conduct deployments by providing a centralized view of
push requests from engineers and other related information from reviews, tests,
and issue trackers.

Pushmanager has integration for:
* Reviewboard
* Buildbot
* Trac, JIRA, etc
* Git
* IRC
* XMPP
* SMTP

## Pushmanager Workflow

The following terms are used by pushmanager:

* Push - The entire process of deploying code to the production environment
* Push Request - A branch that is can be included in a push
* Pushee - A developer who has push requests in a push
* Pushmaster - The privileged user who manages a push and ultimately runs all the commands to deploy a push
* Deploy Branch - A special branch that contains all the push requests in a push
* Pickme - A push request that a developer wants to include in a push
* Certification - Once a push has been deployed to master and is accepted, it
  should be explicitly marked as good. This typically means the deploy-branch
  is merged into production and tagged.

### The Deploy Process

A deploy branch is created by merging push requests into a branch that is
based off of the master branch. Once the deploy branch is complete, it can be
tested. The deploy branch is then pushed to a staging environment where pushees
can manually verify their changes. After verification, the deploy branch is
pushed into production. If production is deemed safe, it is merged back into
master and certified.

### Pushee Perspective

1. Open a push request for a git branch to be merged to master
1. Pickme into an open push
1. Wait for the pushmaster to accept the branch and push the compiled deploy
   branch to the staging area
1. Verify and confirm verification of the branch
1. Wait for the pushmanater to push the deploy branch and certify the branch

### Pushmaster Perspective

1. Open a push
1. Assess and accept pickmes into the deploy branch
1. Build the deploy branch
1. Test the deploy branch
1. Push deploy branch to staging environment
1. Notify pushees and wait until all pushees have verified their branches
1. Push to production
1. Certify deploy branch and merge to master

## Manually Deploying Pushmanager

Pushmanager is two services - an api backend and a tornado frontend. Both must
be run from inside a virtualenv.

1. Provide a configuration file. `config.yaml.example` provides a base for you to fill out.

   The required sections must be changed:
       main_app, api_app, db_uri, username, log_path, auth_ldap, mail, xmpp, git
       All keys must exist. Pushmanager is not resilient against unexpected yaml.

1. `tox -r` to recreate the virtualenv
1. `SERVICE_ENV_CONFIG_PATH=/path/to/config.file .tox/py27/bin/pushmanager_api start`
1. `SERVICE_ENV_CONFIG_PATH=/path/to/config.file .tox/py27/bin/pushmanager_main start`

Check the error logs to verify the two services started. No output is good.

Pushmanager can then be accessed on the local ip with the configured
`main_app.port`. Pushmanager links are created by using `main_app.servername`
and `api_app.servername`.
