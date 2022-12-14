# Introduction
In Azure App Service, you can run your apps directly from a deployment ZIP package file. This article shows how to enable/disable this functionality in your app.

# Advantages and Limitations
When you enable this functionality in your Azure App Service, the files in the package are not copied to the wwwroot directory. Instead, the ZIP package itself gets mounted directly as the read-only wwwroot directory. There are several benefits to running directly from a package:

* Eliminates file lock conflicts between deployment and runtime.
* Ensures only full-deployed apps are running at any time.
* Can be deployed to a production app (with restart).
* Improves the performance of Azure Resource Manager deployments.
* May reduce cold-start times, particularly for JavaScript functions with large npm package trees.