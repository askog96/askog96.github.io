# Create and Configure new users using htpassd
Create a new user new_admin with the password redhat in the existing file ~/auth-providers/htpasswd:
<div class="copy-to-clipboard">
  <pre>
  <code>htpasswd -c -B -b ~/auth-providers/htpasswd new_admin redhat</code>
  </pre>
</div>
-c: Create a new file. If the file already exists, it will be overwritten.
-B: Use bcrypt encryption for the password, which is more secure.
-b: Use the password from the command line rather than prompting for it.
~/auth-providers/htpasswd: The path to the file where the user credentials will be stored.
new_admin: The username of the new user.
redhat: The password for the new user.
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