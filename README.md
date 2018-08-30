# Job Step plugin to wrap common Java keytool commands

## Pre-Requisite

The remote node's SSH server must be configured to accept RD_* environment variables. This can be read about [here|<https://linux.die.net/man/5/sshd_config]>.

Summary, on the remote node:

1. Edit /etc/ssh/sshd_config and add this:

    ```shell
    # Accept Rundeck environment variables
    AcceptEnv RD_*
    ```
2. Restart sshd service:

    `sudo service sshd restart`

## Building

To build the plugin, simply zip it but exclude the .git files:

    ```shell
    zip -r java-keytool-steps.zip java-keytool-steps -x *.git*
    ```

Then copy the zip file to the plugins directory:

    ```shell
    cp java-keytool-steps.zip $RDECK_BASE/libext
    ```
