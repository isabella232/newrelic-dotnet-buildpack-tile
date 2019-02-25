---
# Using New Relic Dotnet Extension Buildpack for PCF

---

<br/><br/>

This topic describes how to use New Relic Dotnet Buildpacks for Pivotal Cloud Foundry (PCF).


## <a id='using'></a> Using New Relic Dotnet Extension

New Relic extension buildpacks allow you to use Dotnet agents in various ways to make it easy and convenient for users to bind to New Relic service.

<br/>


### <a id='how-it-works'></a> How the Extension Buildpack Works

The buildpack extensions bind to New Relic Dotnet agent using one of the following ways as fits in your environment:

1. If an environment variable named <strong>"NEW_RELIC_DOWNLOAD_URL"</strong> is defined in the application's environment (manifest file, cf set-env, or set in AppsMgr), its value is used as the location to download the agent. This is used for environments where you have your own repository for downloading dependencies (i.e. Artifactory). If a second environment variable called <strong>"NEW_RELIC_DOWNLOAD_SHA256"</strong> is set in the application environment, it is expected to hold the downloaded agent's binary file SHA256 checksum. In this case the SHA256 checksum of the downloaded agent binary file is compared with this value, and in case of mismatch, the buildpack reports an error.

1. You could create a cached version of the buildpack using the <strong>"--cached"</strong> switch with New Relic agent embedded in the buildpack. This is mainly used in disconnected (isolated) environments where PCF does not have access to the outside world, and the buildpack cannot download the agent from New Relic's download site.

1. Use a specific version of New Relic agent by specifying the version in the buildack's manifest file at the time of packaging.

1. Set the version of the agent to <strong>"0.0.0.0"</strong>, <strong>"latest"</strong>, or <strong>"current"</strong> in buildpack's manifest file to download the latest version of the agent.

Except in the first case when the download url is specified, but sha-256 code is not available, in all other cases the buildpack checks the SHA256 checksum of the downloaded (or copied) agent to validate the download.

We always encourage you to use the latest version of New Relic agents unless there is a reason that keeps you from upgrading to newer releases of the agent.


<br/>


### <a id='precedence'></a> Order of Precedence

The order of precedence for which method to use to obtain New Relic agent is from the top to bottom. If <strong>"NEW_RELIC_DOWNLOAD_URL"</strong> is specified, it precedes the other options. If this environment variable is not specified, the cached buildpack takes precedence. Otherwise, the <strong>"version"</strong> property of the agent in the buildpack's manifest is used, and one of the other two options is used to download the agent, depending on the value of <strong>"version"</strong> property of the agent dependency (explicit version or "latest").



## <a id='how-it-operates'></a> How The Extension Buildpack Binds the Apps to New Relic Agent
The buildpack looks for several environment variables and files to determine how to bind the application to the agent.


### <a id='license-key'></a> Bind the Application to New Relic Agent
The license key for New Relic account must be specified in one of the following ways:<br/><br/>
* NEW_RELIC_LICENSE_KEY env var<br/>
* License key from User-Provided-Service<br/>
* License key from Service Broker Tile in Marketplace<br/>
* License key from newrelic.config<br/>

<strong>Note:</strong> environment variables override all other options.


### <a id='app-name'></a> Application Name in New Relic UI
The application name for New Relic is determined in the following order:<br/><br/>
* NEW_RELIC_APP_NAME env var<br/>
* App name from User-Provided-Service<br/>
* App name from PCF<br/>
* <strong>Note</strong> that currently App name in newrelic.config file is not used<br/>


### <a id='agent-config'></a> New Relic Agent Configuration File
New Relic configuration file (<strong>"newrelic.config"</strong>) would allow you to set a number of agent properties, and change the behavior of the agent as you wish. Refer to [.NET agent configuration](https://docs.newrelic.com/docs/agents/net-agent/configuration/net-agent-configuration) for more information on configuring the agent. You could make a copy of this file into the application's root directory, and change any of agent's settings. The buildpack looks for the agent's config file in the following order:<br/><br/>
* App root folder<br/>
* Buildpack folder<br/>
* Agent folder<br/>


### <a id='ups'></a> New Relic User-Provided-Services
If the application binds to a User-Provided-Service with the word <strong>"newrelic"</strong> as part of its name, the buildpack sets the credentials from this service in the application environment by setting environment variable for known New Relic properties. The known properties currently are:<br/><br/>
* NEW_RELIC_LICNESE_KEY<br/>
* NEW_RELIC_APP_NAME<br/>
* NEW_RELIC_DISTRIBUTED_TRACING_ENABLED<br/>


### <a id='proxy'></a> Use of Proxy
If you're using a proxy server in your environment, you need to make a copy of <strong>"newrelic.config"</strong> file of the agent in the application directory, and specify the [proxy information](https://docs.newrelic.com/docs/agents/net-agent/configuration/net-agent-configuration#proxy) as a child of the <strong>&lt;service&gt;</strong> element.<br/>
<strong>Example:</strong><br/>
<pre>
    &lt;service licenseKey="0123456789abcdef0123456789abcdef01234567"&gt;
      &lt;proxy name="my.proxy.address" port="proxy.port" /&gt;
    &lt;/service&gt;
</pre>




## <a id='tips-tricks'></a>Tips & Tricks

### <a id='using-nr-agent'></a>Using New Relic Agent

You need to specify a New Relic account <strong>license key</strong> in one of the following ways in order to bind your application to New Relic service:

* Bind your application to New Relic using a License Key

    The quickest and easiest way to bind your app to New Relic Dotnet agent is using the license key environment variable (<strong>"NEW_RELIC_LICNENSE_KEY"</strong>) and set it to your New Relic account's license key.

<br/>

* Bind your application to a local copy of New Relic Dotnet Agent

    If you are in a disconnected (isolated) environment, the easiest way is to obtain the latest version of New Relic Dotnet agent(s) from [New Relic download site](http://download.newrelic.com/dot_net_agent/) and upload them to your local repository (i.e. Artifactory). Then in the application's manifest file set <strong>"NEW_RELIC_DOWNLOAD_URL"</strong> environment variable to the  location of the agent download.


* Bind your application to New Relic using the Agent Tile in the Marketplace<br/>
    * From Marketplace<br/>
        - click on New Relic tile
        - select the plan that is associated with your new relic account (check with PCF admin)
        - specify the instance name
        - if you already have the application running on PCF you could specify the app name as well. Otherwise, once you push the app, from the app UI you could directly bind to an existing service instance (or create a new service instance)
        - click the <strong>CREATE</strong> button to create the service instance

        - once the service instance is created, you could also use the "services" stanza in your app manifest.yml file, and specify the name of the service you just created in the marketplace
        - restage the application using "cf restage YOUR_APPNAME" from the command line


    * From "Services" tab of the application in AppMgr
        - Push your application to PCF
        - In AppMgr click on your application
        - goto "Services" tab of your application
        - If you have already created a service instance:
            - select "BIND SERVICE".
            - select the desired service instance from the list
            - click "BIND"
        - If you have not created any service instances, and this is the first time:
            - select "NEW SERVICE"
            - click on the New Relic service tile
            - select the plan that is associated with your new relic account
            - click "SELECT PLAN" button
            - specify the instance name
            - click "CREATE"
        - "restage" the application using "cf restage YOUR_APPNAME" from the command line


* Bind your application to New Relic using User-Provided-Service
    - Create a user-provided-service with the word "newrelic" embedded as part of the service name 
    - add the following credentials to the user-rpovided-service:
        - "licenseKey" This is New Relic License Key - <strong>REQUIRED</strong>
        - "appName"    If you want to change the app name in New Relic use this property - <strong>OPTIONAL</strong>
        - sample CUPS command: ```cf cups my-newrelic-svc -p "licenseKey,appName"```
            - you will be prompted to provide values for licenseKey and appName<br/>
    - push your application in one of the following ways:
        - by adding the user-provided-service to the application manifest.yml before pushing the app
        - by adding the user-provided-service in AppMgr by binding an existing service instance, and restaging it after you bind the service


* Bind your application to New Relic using CF CLI
    - create an instance of New Relic service:
        <pre>
            cmd: cf create-service newrelic &lt;NEWRELIC_PLAN_NAME&gt; &lt;YOUR_NEWRELIC_SERVICE_INSTANCE_NAME&gt;
        </pre>
    - specify the newly created service instnace name in the <strong>"services"</strong> section of the application manifest (indented by two spaces):
        <pre>
              services:
              - YOUR_NEWRELIC_SERVICE_INSTANCE_NAME
        </pre>
    - push the application
        <pre>
            cmd: cf push<br/>
        </pre>

<br/>

### <a id='envvar-and-nrconfig'></a>Using Environment Variables and Agent Configuration

* You can use a combination of "newrelic.config" file and/or environment variables to configure New Relic Dotnet agent to report your application's health and performance to the designated New Relic account.

    - A copy of the 'newrelic.config' file is provided with the buildpack. If you need to add any agent features such as proxy settings, or change any other agent settings such as logging behavior, download <strong>newrelic.config</strong> file from New Relic into the application folder, and edit as required. You can refer to [New Relic Configuration documentation](https://docs.newrelic.com/docs/agents/net-agent/configuration/net-agent-configuration) for details on what properties are available. The following are some examples you can use:

        - <strong>new relic license key:</strong> add your New Relic license key:<br/>
            ```
              <service licenseKey="0123456789abcdef0123456789abcdef01234567">
            ```

        alternatively you can add the license key to application's 'manifest.yml' file as an environment variable "NEW_RELIC_LICENSE_KEY" in the "env" section


        - <strong>new relic app name:</strong> If you want the app name in New Relic be different than the app name in PCF, add the New Relic application name as you'd like it to appear in New Relic<br/>
        <pre>
              &lt;application&gt;
                &lt;name&gt;My Application&lt;/name&gt;
              &lt;/application&gt;
        </pre>

        alternatively you can add the New Relic app name to application's 'manifest.yml' file as an environment variable "NEW_RELIC_APP_NAME" in the "env" section


        - <strong>proxy setting:</strong> add proxy settings to the "service" element as a sub-element. example:<br/>
        <pre>
              &lt;service licenseKey="0123456789abcdef0123456789abcdef01234567"&gt;
                &lt;proxy host="my_proxy_server.com" port="9090" /&gt;
              &lt;/service&gt;
        </pre>


        - <strong>logging level:</strong> change agent logging level and destination<br/>
        <pre>
              &lt;log level="info" console="true" /&gt;
        </pre>

        - <strong>non-IIS executable instrumentation:</strong> since for dotnet framework apps 'hwc.exe' is the executable running your application, the copy of <strong>"newrelic.config"</strong> file that comes with the hwc extension contains the proper settings to tell the executable name to the agent. If you provide your own <strong>"newrelic.config"</strong> file, make sure it contains the following tag:
        <pre>
              &lt;instrumentation&gt;
                &lt;applications&gt;
                  &lt;application name="hwc.exe" /&gt;
                &lt;/applications&gt;
              &lt;/instrumentation&gt;
        </pre>
        
        Note:  Depending on your CI/CD pipeline, the Application directory may be created on-the-fly as part of the pipeline.  If that is the case and you are modifying this file, your pipeline will need to copy over the file to the Application directory before deploying/pushing the app to PCF.


    - Push your application to PCF using this buildpack. To do that, edit your manifest.yml and add/update the following entry.

        <pre>
            buildpacks:
            - &lt;NEWRELIC_EXTENSION_BUILDPACK_NAME&gt;
            - hwc_buildpack
        </pre>

        Then run "cf push".<br/>

        Note: If you use bamboo as your CI/CD pipeline tool, the "cf push" may not be required as your pipeline internally uses "cf push" to push the application to PCF.


<br/>

### <a id='envvar-and-nrconfig'></a>Debugging

* Set <strong>BP_DEBUG</strong> environment variable
        For troubleshooting purposes, you can set <strong>BP_DEBUG</strong> environment variable in your application environment to get more logging during the application push.


* Check the logs

    Use <strong>cf logs &lt;APP_NAME&gt;</strong> or <strong>cf logs &lt;APP_NAME&gt; --recent</strong>   to examine the application logs. It should display New Relic agent installation progress.


* User Cloud Foundry's [<strong>CF_TRACE</strong>](https://docs.cloudfoundry.org/devguide/deploy-apps/troubleshoot-app-health.html#trace-cloud-controller-rest-api-calls) to get detailed information about errors and unexpected behavior.



## <a id='examples'></a>Examples

### <a id='example-dotnet-core'></a>Push a Sample Dotnet Core Application to PCF
The following example uses <strong>"dotnet"</strong> CLI to create a sample Dotnet Core application, configures the app manifest to bind the application with New Relic Dotnet Core agent, and pushes the app to PCF. Note that the example uses environment variable to specify the license key for the New Relic account. You could explore this page for alternative ways to bind an application to the agent.

1- Create a sample dotnet MVC application (or use you existing dotnet Core application)<br/>
```
dotnet new mvc -n myapp
```

2- Test the app locally<br/>
```
cd myapp
dotnet run
```

3- Create a manifest.yml file for the application<br/>
<pre>
    ---
    applications:
    - name: myapp
      memory: 256M
      buildpacks:
        - nr_dotnetcore_extension_buildpack
        - dotnet_core_buildpack
      env:
        NEW_RELIC_LICENSE_KEY: 0123456789abcdef0123456789abcdef01234567

</pre>


4- Push the application to PCF<br/>
```
cf push
```


<br/>


### <a id='example-dotnet-framework'></a>Push a Sample Dotnet Framework Application to PCF

The following example shows the steps to configure the app manifest to bind a Dotnet Framework application New Relic Dotnet Framework agent using New Relic HWC extension buildpack and Cloud Foundry's standard HWC buildpack, and pushes the app to PCF. Note that the example uses environment variable to specify the license key for the New Relic account. You could explore this page for alternative ways to bind an application to the agent.

1- Create a sample dotnet framework application (or pick one of your dotnet framework apps)

2- Test the app locally and make sure it runs with no errors

3- Create a manifest.yml file for the application<br/>
<pre>
    ---
    applications:
    - name: myapp
      memory: 256M  # adjust as needed
      buildpacks:
        - nr_hwc_extension_buildpack
        - hwc_buildpack
      env:
        NEW_RELIC_LICENSE_KEY: 0123456789abcdef0123456789abcdef01234567

</pre>

4- Push the application to PCF<br/>
```
cf push
```

<br/><br/><br/>
---
---
---
---
---
