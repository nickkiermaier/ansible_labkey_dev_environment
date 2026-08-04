# LabKey Local Bootstrap (Ansible)

## Pre-reqs
* You will need to install `direnv` and `ansible` on your local machine
* You will need to ensure that your ssh key is configured properly in github such that you can `git pull` the TNBRC module successfully

## Configure
* You can configure all the needed variables in the `group-vars/all.yml` file. 

## To use
* From the root of the repo run `ansible-playbook ./playbooks/<your playbook>`
* If you look in the playbook directory you will see numbered playbooks which need to be run in order
* The artifactory configuration may be optional if you've already configured artifactory
* If run with default configuration this entire repo will be configure in your home directory: `~/labkey`

## After 
* You will still need to setup intellij per the Labkey developer setup docs
* Make sure to point the java location inside of intellij to the `~/labkey/dependencies/<java-version>` directory
