# Job Step plugin to wrap common Java keytool commands (Linux/Unix)

## Usage

---

### Pre-Requisites

#### Remote node setup

The remote node's SSH server must be configured to accept RD_* environment variables. This can be read about [here](<https://linux.die.net/man/5/sshd_config>).

Summary, on the remote node:

1. Edit `/etc/ssh/sshd_config` and add this:

    ```shell
    # Accept Rundeck environment variables
    AcceptEnv RD_*
    ```
2. Restart sshd service:

    `sudo service sshd restart`

#### Rundeck setup

Create an item in Rundeck's [Key Storage](<https://www.rundeck.com/blog/use-rundecks-key-storage-to-manage-passwords-and-secrets>) that houses your keystore password (and private key password, if applicable).

### Actions

This plugin includes support for the following keytool actions:

#### java / keytool / checkExpiration

For a given keystore/alias, displays the expiration date by parsing the output from `keytool -list -v -keystore keystore.jks -alias mydomain`.
Optionally specify the Age Threshhold (in days) to make the job fail if the specified alias will expire in that many days or less. This will allow you to schedule the job and trigger notifications or workflows when your certificates are nearing their expiration dates.

#### java / keytool / create

Creates a new Java keystore (JKS) or updates an existing one with a new key pair in the location specified - and optionally creates a certificate request. After providing the necessary inputs in the job step properties, uses the following actions:
`keytool -genkeypair`

and if specified:
`keytool -certreq`

#### java / keytool / delete

Deletes an alias from the specified keystore. After providing the necessary inputs in the job step properties, uses the following action:
`keytool -delete`

#### java / keytool / import

Imports certificate into the specified keystore. After providing the necessary inputs in the job step properties, uses the following action for the alias specified:
`keytool -import`

#### java / keytool / list

Either checks which certificates are in a Java keystore, or gives information about a particular keystore entry.

If no alias is provided in the job definition, it will execute the following for a given keystore:
`keytool -list -v -keystore keystore.jks`

If an alias *is* provided, it will execute the following for a given keystore/alias:
`keytool -list -v -keystore keystore.jks -alias mydomain`

#### java / keytool / printcert

Gives information about a standalone certificate. Executes the following for a given certificate:
`keytool -printcert -v -file mydomain.crt`

### Usage Examples

---
The included actions can be used on their own or in combination with each other or other steps. Here are a couple of examples:

**Monitoring Your SSL Certificates - simple method**
For a job that will notify you when an SSL certificate is nearing it's expiration date, create a job with the following step(s):

1. `java / keytool / checkExpiration`
    * Set job step properties to point to your keystore and alias, and specify a notification threshhold (e.g. 30 days)
    * Email on failure to you or your team's inbox

    Schedule the job to run on a schedule, e.g. every Sunday night @ 23:59.

**Monitoring Your SSL Certificates - advanced method**
For a workflow that will automatically create a new keystore & certificate request when your certs hit a certain threshhold, plus a job that will import the newly signed certs into your keystore, create a workflow like this:

1. Schedule a job with the following steps to monitor your certificate:
    1.1. `java / keytool / checkExpiration`
    * Set job step properties to point to your keystore and alias, and specify a notification threshhold (e.g. 30 days)
    * On error, add a `java / keytool / create` step to create a new keystore and certificate request
        * Set job step properties to specify the new keystore and private key details. It's recommended that you set the keystore path to a temporary location
        * Specify a directory to generate a certificate request into
    * Email on job failure to you or your team's inbox (attach output log to email to receive the contents of the new certreq via email)

2. When the certificate nears the age threshhold the job will fail, generate a certificate request, and email it to you.
3. Process the certificate request with your Certificate Authority of choice. Save the resulting root, intermediate, and signed server certificates to a directory that the target server has access to.
4. Execute/schedule a job with the following steps to process the updated certs:
    4.1 `java / keytool / import` - root
    * In the job step properties, list:
        * the path to the temporary keystore created in step 1.1
        * the path to the root certificate file
        * the alias to be used for the root certificate
        * "None" in the Private Key Password property

    4.2 `java / keytool / import` - intermediate
    * In the job step properties, list:
        * the path to the temporary keystore created in step 1.1
        * the path to the intermediate certificate file
        * the alias to be used for the int certificate
        * "None" in the Private Key Password property

    4.3 `java / keytool / import` - signed server certificate
    * In the job step properties, list:
        * the path to the temporary keystore created in step 1.1
        * the path to the signed server certificate file
        * the alias to be used for the server certificate
        * the Private Key Password

    4.4 Step to copy the new keystore to the "live" location.
    4.5 Step to restart the web server

## Building

---

1. To build the plugin, simply zip it but exclude the .git files:

    ```shell
    zip -r java-keytool-steps.zip java-keytool-steps -x *.git*
    ```

2. Then copy the zip file to the plugins directory:

    ```shell
    cp java-keytool-steps.zip $RDECK_BASE/libext
    ```