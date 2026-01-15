# LabKey Local Bootstrap (Ansible)

This README documents how to run the LabKey repo bootstrap playbook (labkey-bootstrap.yml).


#######################################################################
## What This Playbook Does
#######################################################################

* Clones the main LabKey server repository
* Clones required server/ and server/modules/ repositories
* Checks out a pinned Git version across all repos
* Downloads and installs a pinned Temurin JDK under dependencies/
* Configures:
  * JAVA_HOME
  * PATH (Java + build/deploy/bin)
* Creates a global ~/.gradle/gradle.properties
* Injects Artifactory credentials for Gradle builds
* Shows verbose output for Git and Java steps

#######################################################################
## Prerequisites
#######################################################################

### Git

Git must be installed and working:

git --version

### SSH Access (IMPORTANT)

WARNING: You MUST be able to pull private GitHub repos over SSH before running this.

Verify SSH access:

ssh -T git@github.com

If this fails:
* your SSH key is not added to GitHub
* ssh-agent is not running
* the key does not have access to LabKey private repos

If SSH is not working, the playbook WILL fail.

#######################################################################
## Installing Ansible
#######################################################################

Ansible must be installed before running run.sh.

### NixOS

nix-shell -p ansible

### Debian / Ubuntu

sudo apt update
sudo apt install -y ansible python3 python3-venv python3-yaml

### Fedora

sudo dnf install -y ansible python3 python3-libselinux

Verify:

ansible --version

#######################################################################
## Secrets Configuration (REQUIRED)
#######################################################################

This repo includes a secrets template that is intentionally ignored by Git.

The template file:
group_vars/secrets_template.yml

The real secrets file (gitignored):
group_vars/secrets.yml

### Step 1: Create secrets.yml from the template

cp group_vars/secrets_template.yml group_vars/secrets.yml

### Step 2: Edit secrets.yml

Put your Artifactory credentials in group_vars/secrets.yml:

artifactory_user: "YOUR_USERNAME"
artifactory_password: "YOUR_PASSWORD"

These values are injected into:
~/.gradle/gradle.properties

#######################################################################
## Customize the Setup
#######################################################################

Review and customize:
group_vars/all.yml

This is where you set:
* root_dir (where everything installs)
* git_version (branch/tag/commit for all repos)
* server_repos and module_repos
* java_version_full (example: 17.0.17+10)
* other environment settings

#######################################################################
## Run the Bootstrap
#######################################################################

From the ansible repo root:

./run.sh

run.sh runs the LabKey bootstrap playbook and applies all configuration.

You can re-run it at any time.

#######################################################################
## After the Run
#######################################################################

Verify Java:

java -version
echo $JAVA_HOME

Verify Gradle properties exists:

ls -l ~/.gradle/gradle.properties

Verify repo layout:

ls -l "$HOME/labkey"
ls -l "$HOME/labkey/dependencies"

#######################################################################
## Common Failures
#######################################################################

### Git clone failures
* SSH not configured
* no access to private repos
* wrong GitHub key loaded in ssh-agent

### Gradle failures
* secrets.yml not created
* Artifactory credentials wrong

### Java issues
* java_version_full is wrong
* download blocked by proxy/firewall
