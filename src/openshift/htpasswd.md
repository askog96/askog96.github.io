# Manage users with htpasswd in OpenShift

These are the commands I use when I need to manage local users with `htpasswd` in OpenShift.

## Create a new htpasswd file

This creates a new file and adds a user called `new_admin`:

```bash
htpasswd -c -B -b ~/auth-providers/htpasswd new_admin redhat
```

The important part here is `-c`, since that creates a new file. If the file already exists, it will be overwritten.

## Create the secret used by OAuth

```bash
oc create secret generic localusers --from-file htpasswd=~/auth-providers/htpasswd -n openshift-config
```

## Restart OAuth

```bash
oc rollout restart deployment/oauth-openshift -n openshift-authentication
```

## Give the user cluster-admin

```bash
oc adm policy add-cluster-role-to-user cluster-admin new_admin
```

## Add a user to an existing htpasswd setup

If the secret already exists, extract the current file first:

```bash
oc extract secret/localusers -n openshift-config --to ~/auth-providers/ --confirm
```

Then add another user:

```bash
htpasswd -b ~/auth-providers/htpasswd manager redhat
```

Update the secret:

```bash
oc set data secret/localusers --from-file htpasswd=~/auth-providers/htpasswd -n openshift-config
```

And restart OAuth again:

```bash
oc rollout restart deployment/oauth-openshift -n openshift-authentication
```
