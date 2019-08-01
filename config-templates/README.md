# mosip-configuration-templates
Templates for mosip configuration.

These template files contain all the configuration information needed by mosip services. To use these files to setup MOSIP services manually perform following actions:

1. Create a folder named `config` and copy all the files from `config-templates` folder and paste them in the new folder `config` created.

2. Replace all the value placeholders with the actual values for your deployments, i.e. replace all the values starting and ending with double curly brackets and all the values  starting and ending with angular brackets.<br/>
  
2. Once done, rename all the files and replace the env in property files' names to the environment for which you have created the properties.
   For example if you are setting up `dev` environment, rename the property file `id-repository-env.properties` to `id-repository-dev.properties` and all other files also in the same way.  
