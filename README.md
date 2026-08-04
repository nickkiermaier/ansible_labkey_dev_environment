# LabKey Local Bootstrap (Ansible)
* You will need to install `direnv` and `ansible` on your local machine
* You will need to ensure that your ssh key is configured properly in github such that you can `git pull` the TNBRC module successfully
* From the root of the repo run `ansible-playbook ./playbooks/<your playbook>`
* If you look in the playbook directory you will see numbered playbooks which need to be run in order
* The artifactory configuration may be optional if you've already configured artifactory
