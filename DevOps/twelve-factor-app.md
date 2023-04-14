## Codebase
Have a single codebase for the application. #git #github #vcs #version-control
	*Applications should be seperated into their own repositories*

## Dependencies
Explicitly declare and isolate your dependencies #virtual-environments #venv #docker #containerization
	*Never rely on implicit existence of system-wide packages*

## Config
Store the config in the environment #env #variables 
	*This makes the application deployable to multiple environments while only changing the environment variables*

## Backing Services
Treat backing services as attached resources #redis #s3 #nsmtp
	*Wherever the service is hosted, you shouldn't have to change your code to access the service*

## Build, Release, Run
Strict separation between the build, release, and run stages. The build phase consists of creating the binary or docker image. The release phase combines that image with the environment variables and is versioned. The run phase is actually deploying the application.
	*A 12 factor app should be able to rollback to a previous release or redeploy as needed*

## Processes
Execute the app as one or more stateless processes #backend #database #caching #redis #postgresql
	*Processes should be stateless and share nothing.*

## Port Binding
Export services via port binding
	*The 12 factor app does not rely on any specific port binding and is entirely self contained*

## Concurrency
Scale out via the process model #loadbalancers #container-orchestration #kube
	*Applications should scale out horizontally, not vertically*

## Disposability
Fast startup and graceful shutdown
	*The 12 factor app's processes are disposable, meaning they can be started or stopped at a moment's notice*

## Dev/Prod Parity
Keep the development and production environments as similar as possible
	*The 12  factor app is designed for continuous deployment by keeping the gap between development and production small*

## Logs
Treat logs as event streams
	*A 12 factor app never concerns itself with routing or storage of its output stream*
In practice, the 12 factor app writes all its logs to standard out, or to a local file in a structured json format that can then be transferred to an agent and centralized.

## Admin Processes
Run admin tasks as one off processes
	*Administrative tasks should be kept separate from the application process and should be run in an identical setup, be automated, scalable and reproducible*
For example. You have a website service that runs in a docker container, and a redis cache that stores the total number of visitors to your site. You need to reset the visitor count. Instead of including the code for this in the website code, you would have it run in another container and close after running. 
