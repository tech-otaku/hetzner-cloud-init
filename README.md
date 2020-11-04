# hetzner-cloud-init

Used in combination with the Hetzner Cloud API to configure a newly-created VPS through cloud-init.

Adapted from the Hetzner Cloud API [documentation](https://docs.hetzner.cloud/#servers-create-a-server "Create a Server"):

```
curl \
    -X POST -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -d '{"name": "dev-server", "server_type": "cx11", "location": "nbg1", "start_after_create": true, "image": "ubuntu-20.04", "ssh_keys": ["root-ssh-key"], "volumes": [], "user_data": "#include\nhttps://raw.githubusercontent.com/tech-otaku/hetzner-cloud-init/master/config.yaml", "automount": false}' \
    https://api.hetzner.cloud/v1/servers
```
<br />

Typically, the string passed to the `user_data` parameter in the code above begins with `#include` + `\n` (for a carriage return) + the permalink URL to the raw version of this configuration file:

```
, "user_data": "#include\nhttps://raw.githubusercontent.com/tech-otaku/hetzner-cloud-init/master/config.yaml",
```
<br />

BEWARE: Changes made to the configuration file may not immediately be reflected in its raw version.

<br />

### NOTES

- The JSON-encoded data included in the POST request sent to the Hetzner Cloud API contains the key/value pair `"ssh_keys": ["root-ssh-key"]`. This assumes an SSH public key for the server-to-be-created's `root` user named `root-ssh-key` exists on the *Hetzner Cloud Console* under *Security* > *SSH Keys*.

   This key can be uploaded to the *Hetzner Cloud Console* beforehand using:

    ```
    curl \
        -X POST \
        -H "Authorization: Bearer $API_TOKEN" \
        -H "Content-Type: application/json" \
        -d '{"name":"root-ssh-key","public_key":"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKqNH/3NYvHOE9gmYKCu+InAQtNLnI/a4/z65NHfrvNg dummy-root-20201104-084213-gmt","labels":{"environment":"development", "server":"site.example.com"}}' \
        'https://api.hetzner.cloud/v1/ssh_keys'
    ```
<br />

- The cloud-init configuration file changes the SSH port to `5822` and disables password authentication. To connect to the newly created server use:

    ```
    ssh -p 5822 -i /path/to/user/keys/private-key user@203.0.113.5
    ```
<br />

- The encrypted password (`passwd`) was created using `perl -e 'print crypt("password","\$6\$UnXq642da9EfkQfH\$")'` on a Ubuntu client.
<br />

- The SSH keys were created using `yes "y" | ssh-keygen -o -a 100 -t ed25519 -f /path/to/user/keys -N "" -C this-is-a-comment`
