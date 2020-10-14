# OpenShift Installer Wrapper
## Index
1. [Description](#description)
2. [Preparation](#preparation)
3. [Usage](#usage)
4. [Adding customizations](#adding-customization-scripts)
5. [TODO](#todo)

## Description
This is a wrapper for the official `openshift-install` binary to perform IPI (Installer Provided Infrastructure) installations of OpenShift 4.

Features:
 - downloads the installer and client for the desired version
 - creates sample cloud credential files in the right location for each cloud provider
 - creates install-config.yaml for each cloud provider
 - allows customize a previously installed cluster with some pre-made scripts
 
## Preparation
The wrapper leverages in `openshift-install` all the tasks but it requires some pre-requisites:
  - proper credentials for every cloud provider
  - RSA key
  - pull secret

### Script installation
This will copy the script to the default location `$HOME/.local/ocp4`:
```
$ make install         
```

For other locations set `TARGETDIR` at your convenience:
```
$ make TARGETDIR=/opt/ocp4 install         
```

Add the directory to your `$PATH` variable so you can invoke it directly:
```
$ export PATH=$PATH:$HOME/.local/ocp4/bin
```

### Cloud credentials
If you don't have such credentials yet (ie. if you don't use aws-cli at all), you can invoke the wrapper for every platform so it will create the sample file and then proceed to edit it with your own credentials:
```
$ openshift-install-wrapper --init --platform aws
→ Validating environment...
→ Creating target directory...
→ Creating sample cloud credentials file for aws...
 ✔  Created sample file /home/sgarcia/.aws/credentials. Please edit it to add the proper credentials for each provider before trying to install any cluster or it will fail.
``` 

### Other files
- Copy the RSA public key that will be injected in the instances to `$HOME/.local/ocp4/config/ssh-key.pub`
- Copy the pull secret to `$HOME/.local/ocp4/config/pull-secret.json`

## Usage
The list of features and options is increasing as changes are made. Check the `--help` parameter for the newest list.

```
$ openshift-install-wrapper --help
OpenShift installation wrapper for IPI installations. Version: 1.0.0

Usage: openshift-install-wrapper [--init|--install|--destroy|--customize] [options]

Options:
  --name <name>              - name of the cluster
  --domain <domain>          - name of the domain for the cluster
  --version <version>        - version to install
  --platform <name>          - cloud provider (only aws supported for now)
  --region <name>            - cloud provider region

  --force                    - force installation (cleanup files if required)
  --init                     - initialize the tool and credentials
  --install                  - install the cluster
  --destroy                  - destroy the cluster
  --customize                - customize the cluster with some post-install actions
  --list                     - lists all existing clusters

  --verbose                  - shows more information during the execution
  --quiet                    - quiet mode (no output at all)
  --help|-h                  - shows this message
```

### Install a cluster
```
$ openshift-install-wrapper --install \
                            --name sgarcia-ocp447 \
                            --domain aws.gmbros.net \
                            --version 4.4.7 \
                            --platform aws \
                            --region eu-west-1
→ Validating environment...
→ Checking if installer for 4.4.7 is already present...
 ✔  Installer for 4.4.7 found. Continuing.
→ Checking if the cluster directory already exists...
→ Creating install-config.yaml file...
→ Running "openshift-install" to create a cluster...
 ✔  Cluster created!
To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/sgarcia/.local/ocp4/clusters/sgarcia-ocp447-3.aws.gmbros.net/auth/kubeconfig'
Access the OpenShift web-console here: https://console-openshift-console.apps.sgarcia-ocp447.aws.gmbros.net
Login to the console with user: kubeadmin, password: m2MsJ-vKNcn-8zTHM-A9NVU
```

### Destroy a cluster
```
$ openshift-install-wrapper --destroy \
                            --name sgarcia-ocp447 \
                            --domain aws.gmbros.net
→ Validating environment...
→ Finding version in cluster directory...
 ✔  Version detected: 4.4.7.
→ Checking if installer for 4.4.7 is already present...
 ✔  Installer for 4.4.7 found. Continuing.
→ Running "openshift-install" to destroy a cluster...
 ✔  Cluster destroyed!
→ Removing directory...
```

### Customize a cluster
```
$ openshift-install-wrapper --customize delete-kubeadmin-user \
                            --name sgarcia-ocp447 \
                            --domain emeashift.support \
                            --platform aws
→ Validating environment...
→ Finding version in cluster directory...
 ✔  Version detected: 4.4.7.
→ Checking if client binaries for 4.4.7 are already present...
 ✔  Client binaries for 4.4.7 are found. Continuing.
→ Running delete-kubeadmin-user...
secret "kubeadmin" deleted
```

### Troubleshooting
- Use `--verbose` to get extra information during the execution, including the full output of `openshift-install`
- Review `$HOME/.local/ocp4/clusters/<cluster_name>/.openshift_install_wrapper.log` for useful output

## Adding customization scripts
In order to add new scripts, they must meet some requisites:
 - they must be created in the `scripts/` directory
 - they must have a descriptive name and do not overlap with any existing command or function in the system
 - optionally they can have a name and a description (single line)
 - they must contain some markers and a `main()` function. Use any existing script as a baseline or use this one:
   ```sh
   #!/bin/bash

   # script name: get-cluster-version
   # script description: Runs an oc get clusterversion

   # start main - do not remove this line and do not change the function name
   main() {
     _oc="${3} --kubeconfig=${2}/auth/kubeconfig"
     ${_oc} get clusterversion
   }
   # end main - do not remove this line

   # Keep this if you want to run your script manually or for testing.
   main $@
   ```

On the other hand, in order to provide flexibility, every script will receive the next parameters in this order:
 - name of the customization being executed
 - the full path to the cluster installation directory
 - the full path to the right `oc` client binary
 - verbose mode flag (`0` or `1`)
 - quiet mode flag (`0` or `1`)
 - cluster version
 - cluster name
 - cluster subdomain
 - cloud platform

Finally, after adding your new script, remember to run `make install` in order to install a new version of `openshift-install-wrapper` with your script.

## TODO
- Add GCP support
- Error handling when the cloud credentials are invalid
- Improve `--list` output
- Improve console output (ie. include timestamps)
- Implement `--expire` parameter to delete (using cron) a cluster after a certain time

## Contact
Reach me in [Twitter](http://twitter.com/soukron) or email in soukron _at_ gmbros.net

## License
Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

[here]:http://gnu.org/licenses/gpl.html
