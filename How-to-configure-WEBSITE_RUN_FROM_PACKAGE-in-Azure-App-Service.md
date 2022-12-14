# Introduction
In Azure App Service, you can run your apps directly from a deployment ZIP package file. This article shows how to enable/disable this functionality in your app.

# Advantages and Limitations
When you enable this functionality in your Azure App Service, the files in the package are not copied to the wwwroot directory. Instead, the ZIP package itself gets mounted directly as the read-only wwwroot directory. There are several benefits to running directly from a package:

* Eliminates file lock conflicts between deployment and runtime.
* Ensures only full-deployed apps are running at any time.
* Can be deployed to a production app (with restart).
* Improves the performance of Azure Resource Manager deployments.
* May reduce cold-start times, particularly for JavaScript functions with large npm package trees.

There are also some limitations to running from a package:
* The wwwroot folder becomes read-only and you'll receive an error when writing files to this directory. Files are also read-only in the Azure portal.
* The maximum size for a deployment package file is currently 1 GB.
* Can't use local cache or local DB when running from a deployment package.
* Can't use remote build

You can find more details in the following links:

https://learn.microsoft.com/en-us/azure/app-service/deploy-run-package

https://github.com/projectkudu/kudu/wiki/WEBSITE_RUN_FROM_PACKAGE-and-WEBSITE_CONTENTAZUREFILECONNECTIONSTRING-Best-Practices

# How to enable run from package
1. Enable `WEBSITE_RUN_FROM_PACKAGE` app setting in your Azure App Service
   * Add WEBSITE_RUN_FROM_PACKAGE falg to your bicep file.
   * Follow [the documentation](https://learn.microsoft.com/en-us/azure/app-service/deploy-run-package#enable-running-from-package) do it manually.
1. **Make sure there is no `SCM_SCRIPT_GENERATOR_ARGS` setting in your Azure App Service**. If your project is upgraded from the old version, this setting may be set to `--node`. Remove this setting before deploying your project.
1. Put the `web.config` file if your project using nodeJS. You can find the template file from [here](https://github.com/projectkudu/kudu/blob/master/Kudu.Core/Scripts/iisnode.config.template) and example from [here](https://github.com/Azure-Samples/nodejs-docs-hello-world/blob/master/web.config)