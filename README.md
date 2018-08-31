# Job Step plugin to wrap common Java keytool commands

## Usage

### Pre-Requisite

The remote node's SSH server must be configured to accept RD_* environment variables. This can be read about [here](<https://linux.die.net/man/5/sshd_config>).

Summary, on the remote node:

1. Edit `/etc/ssh/sshd_config` and add this:

    ```shell
    # Accept Rundeck environment variables
    AcceptEnv RD_*
    ```
2. Restart sshd service:

    `sudo service sshd restart`

### Actions

#### java / keytool / checkExpiration

For a given keystore/alias, displays the expiration date by parsing the output from `keytool -list -v -keystore keystore.jks -alias mydomain`.
Optionally specify the Age Threshhold (in days) to make the job fail if the specified alias will expire in that many days or less. This will allow you to schedule the job and trigger notifications or workflows when your certificates are nearing their expiration dates.

#### java / keytool / list

Either checks which certificates are in a Java keystore, or gives information about a particular keystore entry.

If no alias is provided in the job definition, it will execute the following for a given keystore:
`keytool -list -v -keystore keystore.jks`

If an alias *is* provided, it will execute the following for a given keystore/alias:
`keytool -list -v -keystore keystore.jks -alias mydomain`

#### java / keytool / printcert

Gives information about a standalone certificate. Executes the following for a given certificate:
`keytool -printcert -v -file mydomain.crt`

## Building

1. To build the plugin, simply zip it but exclude the .git files:

    ```shell
    zip -r java-keytool-steps.zip java-keytool-steps -x *.git*
    ```

2. Then copy the zip file to the plugins directory:

    ```shell
    cp java-keytool-steps.zip $RDECK_BASE/libext
    ```