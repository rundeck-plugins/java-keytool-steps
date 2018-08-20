This code example shows how to wrap the Java keytool command as a job step plugin.

## Building

To build the plugin, simply zip it but exclude the .git files:

```
zip -r java-keytool-steps.zip java-keytool-steps -x *.git*
```

Then copy the zip file to the plugins directory:

```
cp java-keytool-steps.zip $RDECK_BASE/libext
```
