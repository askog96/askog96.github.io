# Single node OpenShift running on OpenShift Virtualization
A quite large project to get Single node OpenShift to run inside a VM on OpenShift Virtualization. The intention is to more effectivly using the hardware but also enabling an aaS concept with Single node OpenShift
# Prerequisites
Before proceeding, its recommened to use the following:
- ## **Hardware**: 
Ensure that your system is running one of the following:
      - Any server with atleast 192GB of memory.
      - Intel Xenon 6132 CPU
      - Intel xxv710 or Intel E810 NIC
  
- ## **Operating System**:
      - Red Hat OpenShift 4.15 or later.
      - OpenShift Virtualization x.zz
      - List of operators TBA
  
- ## **Access and Permissions**:
  - Admin rights to install required software
  - Access to the project repository
  
- ## **Network Requirements**:
  - Stable internet connection
  - Access to required internal/external resources
  
- ## **Additional Tools** (if needed):
  - Docker (if containerization is used)
  - Virtual environment (recommended for Python projects)
<div class="copy-to-clipboard">
  <pre>
  <code>htpasswd -c -B -b ~/auth-providers/htpasswd new_admin redhat</code>
  </pre>
</div>
<div class="caution">
  <code>&#9888; Caution: Running this command with the `-c` option will overwrite the existing htpasswd file. Make sure you do not lose existing user data.</code>
</div>
## Create new secret used by OAuth
<div class="copy-to-clipboard">
  <pre>
  <code>oc create secret generic localusers --from-file htpasswd=~/auth-providers/htpasswd -n openshift-config</code>
  </pre>
</div>
## Rollout deployment to apply configuration
<div class="copy-to-clipboard">
  <pre>
  <code>oc rollout restart deployment/oauth-openshift -n openshift-authentication</code>
  </pre>
</div>
## Add cluster-admin to user
<div class="copy-to-clipboard">
  <pre>
  <code>oc adm policy add-cluster-role-to-user cluster-admin new_admin</code>
  </pre>
</div>
# Add a user to existing OAuth config
## Extract the file data from secret
<div class="copy-to-clipboard">
  <pre>
  <code>oc extract secret/localusers -n openshift-config --to ~/auth-providers/ --confirm</code>
  </pre>
</div>
## Add new user to htpasswd called manager with password redhat
<div class="copy-to-clipboard">
  <pre>
  <code>htpasswd -b ~/auth-providers/htpasswd manager redhat</code>
  </pre>
</div>
## Update secret used by OAuth
<div class="copy-to-clipboard">
  <pre>
  <code>oc set data secret/localusers --from-file htpasswd=~/auth-providers/htpasswd -n openshift-config</code>
  </pre>
</div>
## Rollout deployment to apply configuration
<div class="copy-to-clipboard">
  <pre>
  <code>oc rollout restart deployment/oauth-openshift -n openshift-authentication</code>
  </pre>
</div>