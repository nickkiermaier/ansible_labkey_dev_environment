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
  * MSSQL setup
* Creates a global ~/.gradle/gradle.properties
* Injects Artifactory credentials for Gradle builds
* Shows verbose output for Git and Java steps

#######################################################################
## Prerequisites
#######################################################################

### Git

Git must be installed and working:
```
git --version
```
### SSH Access (IMPORTANT)

WARNING: You MUST be able to pull private GitHub repos over SSH before running this.

Verify SSH access:
```
ssh -T git@github.com
```
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
```
nix-shell -p ansible
```
### Debian / Ubuntu
```
sudo apt update
sudo apt install -y ansible python3 python3-venv python3-yaml
```
### Fedora
```
sudo dnf install -y ansible python3 python3-libselinux
```
Verify:
```
ansible --version
```
#######################################################################
## Secrets Configuration (REQUIRED)
#######################################################################

This repo includes a secrets template that is intentionally ignored by Git.

The template file:
`group_vars/secrets_template.yml`

The real secrets file (gitignored):
`group_vars/secrets.yml`

### Step 1: Create secrets.yml from the template
```
cp group_vars/secrets_template.yml group_vars/secrets.yml
```
### Step 2: Edit secrets.yml

Put your Artifactory credentials in group_vars/secrets.yml:
```
artifactory_user: "YOUR_USERNAME"
artifactory_password: "YOUR_PASSWORD"
```
These values are injected into:
```
~/.gradle/gradle.properties
```
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
```
./run.sh
```
run.sh runs the LabKey bootstrap playbook and applies all configuration.

You can re-run it at any time.

#######################################################################
## After the Run
#######################################################################

Verify Java:
```
exec $SHELL
java -version
echo $JAVA_HOME
```
Verify Gradle properties exists:
```
ls -l ~/.gradle/gradle.properties
```
Verify repo layout:

```
ls -l "$HOME/labkey"
ls -l "$HOME/labkey/dependencies"
```

#######################################################################
## Starting Labkey
#######################################################################



1) Ensure the database is running

   Make sure your database (SQL Server or PostgreSQL) is running before starting LabKey.
```
   docker compose up -d
   docker compose ps
```
   Verify the database port is listening (SQL Server default is 1433):

   ```
   sudo netstat -tulanep | grep 1433
   ```

   LabKey will fail during startup if the database is not reachable.


2) Deploy the application

   Deploy the application to the embedded Tomcat environment:

   ```
   ./gradlew deployApp
   ```

   You generally only need to rerun this if you pull new code, change modules,
   or update deployment-related configuration.

3) Start LabKey with developer logging enabled

   Start LabKey with Gradle info logging, stack traces, and LabKey developer mode enabled:
```
   ./gradlew startLabKey \
     --info \
     --stacktrace \
     -Dlabkey.devMode=true \
     -Dlabkey.developer=true
```

4) Tail the LabKey log

   In a separate terminal, follow the server log during startup:
```
   tail -f /home/nick/labkey/LabkeyServer/build/deploy/embedded/logs/labkey.log
```


5) Verify the server is running

   Once startup completes, LabKey should be available at:

```
   http://localhost:8080
```
   Confirm Tomcat is listening:
```
   sudo netstat -tulanep | grep java
```


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
