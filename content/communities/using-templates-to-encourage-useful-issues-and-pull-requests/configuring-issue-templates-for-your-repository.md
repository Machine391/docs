Development Containers

Topics 
Dev Container metadata reference
Specification
Dev Container metadata reference
The devcontainer.json file contains any needed metadata and settings required to configurate a development container for a given well-defined tool and runtime stack. It can be used by tools and services that support the dev container spec to create a development environment that contains one or more development containers.

Metadata properties marked with a 🏷️️ can be stored in the devcontainer.metadata container image label in addition to devcontainer.json. This label can contain an array of json snippets that will be automatically merged with devcontainer.json contents (if any) when a container is created.

General devcontainer.json properties
Property	Type	Description
name	string	A name for the dev container displayed in the UI.
forwardPorts 🏷️	array	An array of port numbers or "host:port" values (e.g. [3000, "db:5432"]) that should always be forwarded from inside the primary container to the local machine (including on the web). The property is most useful for forwarding ports that cannot be auto-forwarded because the related process that starts before the devcontainer.json supporting service / tool connects or for forwarding a service not in the primary container in Docker Compose scenarios (e.g. "db:5432"). Defaults to [].
portsAttributes 🏷️	object	Object that maps a port number, "host:port" value, range, or regular expression to a set of default options. See port attributes for available options. For example:
"portsAttributes": {"3000": {"label": "Application port"}}
otherPortsAttributes 🏷️	object	Default options for ports, port ranges, and hosts that aren’t configured using portsAttributes. See port attributes for available options. For example:
"otherPortsAttributes": {"onAutoForward": "silent"}
containerEnv 🏷️	object	A set of name-value pairs that sets or overrides environment variables for the container. Environment and pre-defined variables may be referenced in the values. For example:
"containerEnv": { "MY_VARIABLE": "${localEnv:MY_VARIABLE}" }
If you want to reference an existing container variable while setting this one (like updating the PATH), use remoteEnv instead.
containerEnv will set the variable on the Docker container itself, so all processes spawned in the container will have access to it. But it will also be static for the life of the container - you must rebuild the container to update the value.
We recommend using containerEnv (over remoteEnv) as much as possible since it allows all processes to see the variable and isn’t client-specific.
remoteEnv 🏷️	object	A set of name-value pairs that sets or overrides environment variables for the devcontainer.json supporting service / tool (or sub-processes like terminals) but not the container as a whole. Environment and pre-defined variables may be referenced in the values.
You may want to use remoteEnv (over containerEnv) if the value isn’t static since you can update its value without having to rebuild the full container.
remoteUser 🏷️	string	Overrides the user that devcontainer.json supporting services tools / runs as in the container (along with sub-processes like terminals, tasks, or debugging). Does not change the user the container as a whole runs as which can be set using containerUser. Defaults to the user the container as a whole is running as (often root).
You may learn more in the remoteUser section below.
containerUser 🏷️	string	Overrides the user for all operations run as inside the container. Defaults to either root or the last USER instruction in the related Dockerfile used to create the image. If you want any connected tools or related processes to use a different user than the one for the container, see remoteUser.
updateRemoteUserUID 🏷️	boolean	On Linux, if containerUser or remoteUser is specified, the user’s UID/GID will be updated to match the local user’s UID/GID to avoid permission problems with bind mounts. Defaults to true.
userEnvProbe 🏷️	enum	Indicates the type of shell to use to “probe” for user environment variables to include in devcontainer.json supporting services’ / tools’ processes: none, interactiveShell, loginShell, or loginInteractiveShell (default). The specific shell used is based on the default shell for the user (typically bash). For example, bash interactive shells will typically include variables set in /etc/bash.bashrc and ~/.bashrc while login shells usually include variables from /etc/profile and ~/.profile. Setting this property to loginInteractiveShell will get variables from all four files.
overrideCommand 🏷️	boolean	Tells devcontainer.json supporting services / tools whether they should run /bin/sh -c "while sleep 1000; do :; done" when starting the container instead of the container’s default command (since the container can shut down if the default command fails). Set to false if the default command must run for the container to function properly. Defaults to true for when using an image Dockerfile and false when referencing a Docker Compose file.
shutdownAction 🏷️	enum	Indicates whether devcontainer.json supporting tools should stop the containers when the related tool window is closed / shut down.
Values are none, stopContainer (default for image or Dockerfile), and stopCompose (default for Docker Compose).
init 🏷️	boolean	Defaults to false. Cross-orchestrator way to indicate whether the tini init process should be used to help deal with zombie processes.
privileged 🏷️	boolean	Defaults to false. Cross-orchestrator way to cause the container to run in privileged mode (--privileged). Required for things like Docker-in-Docker, but has security implications particularly when running directly on Linux.
capAdd 🏷️	array	Defaults to []. Cross-orchestrator way to add capabilities typically disabled for a container. Most often used to add the ptrace capability required to debug languages like C++, Go, and Rust. For example:
"capAdd": ["SYS_PTRACE"]
securityOpt 🏷️	array	Defaults to []. Cross-orchestrator way to set container security options. For example:
"securityOpt": [ "seccomp=unconfined" ]
mounts 🏷️	string or object	Defaults to unset. Cross-orchestrator way to add additional mounts to a container. Each value is a string that accepts the same values as the Docker CLI --mount flag. Environment and pre-defined variables may be referenced in the value. For example:
"mounts": [{ "source": "dind-var-lib-docker", "target": "/var/lib/docker", "type": "volume" }]
features	object	An object of Dev Container Feature IDs and related options to be added into your primary container. The specific options that are available varies by feature, so see its documentation for additional details. For example:
"features": { "ghcr.io/devcontainers/features/github-cli": {} }
overrideFeatureInstallOrder	array	By default, Features will attempt to automatically set the order they are installed based on a installsAfter property within each of them. This property allows you to override the Feature install order when needed. For example:
"overrideFeatureInstallОrder": [ "ghcr.io/devcontainers/features/common-utils", "ghcr.io/devcontainers/features/github-cli" ]
customizations 🏷️	object	Product specific properties, defined in supporting tools
Scenario specific properties
The focus of devcontainer.json is to describe how to enrich a container for the purposes of development rather than acting as a multi-container orchestrator format. Instead, container orchestrator formats can be referenced when needed to manage multiple containers and their lifecycles. Today, devcontainer.json includes scenario specific properties for working without a container orchestrator (by directly referencing an image or Dockerfile) and for using Docker Compose as a simple multi-container orchestrator.

Image or Dockerfile specific properties
Property	Type	Description
image	string	Required when using an image. The name of an image in a container registry (DockerHub, GitHub Container Registry, Azure Container Registry) that devcontainer.json supporting services / tools should use to create the dev container.
build.dockerfile	string	Required when using a Dockerfile. The location of a Dockerfile that defines the contents of the container. The path is relative to the devcontainer.json file.
build.context	string	Path that the Docker build should be run from relative to devcontainer.json. For example, a value of ".." would allow you to reference content in sibling directories. Defaults to ".".
build.args	Object	A set of name-value pairs containing Docker image build arguments that should be passed when building a Dockerfile. Environment and pre-defined variables may be referenced in the values. Defaults to not set. For example: "build": { "args": { "MYARG": "MYVALUE", "MYARGFROMENVVAR": "${localEnv:VARIABLE_NAME}" } }
build.options	array	An array of Docker image build options that should be passed to the build command when building a Dockerfile. Defaults to []. For example: "build": { "options": [ "--add-host=host.docker.internal:host-gateway" ] }
build.target	string	A string that specifies a Docker image build target that should be passed when building a Dockerfile. Defaults to not set. For example: "build": { "target": "development" }
build.cacheFrom	string,
array	A string or array of strings that specify one or more images to use as caches when building the image. Cached image identifiers are passed to the docker build command with --cache-from.
appPort	integer,
string,
array	In most cases, we recommend using the new forwardPorts property. This property accepts a port or array of ports that should be published locally when the container is running. Unlike forwardPorts, your application may need to listen on all interfaces (0.0.0.0) not just localhost for it to be available externally. Defaults to [].
Learn more about publishing vs forwarding ports here.
Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array properties.
workspaceMount	string	Requires workspaceFolder be set as well. Overrides the default local mount point for the workspace when the container is created. Supports the same values as the Docker CLI --mount flag. Environment and pre-defined variables may be referenced in the value. For example:
"workspaceMount": "source=${localWorkspaceFolder}/sub-folder,target=/workspace,type=bind,consistency=cached", "workspaceFolder": "/workspace"
workspaceFolder	string	Requires workspaceMount be set. Sets the default path that devcontainer.json supporting services / tools should open when connecting to the container. Defaults to the automatic source code mount location.
runArgs	array	An array of Docker CLI arguments that should be used when running the container. Defaults to []. For example, this allows ptrace based debuggers like C++ to work in the container:
"runArgs": [ "--cap-add=SYS_PTRACE", "--security-opt", "seccomp=unconfined" ] .
Docker Compose specific properties
Property	Type	Description
dockerComposeFile	string,
array	Required when using Docker Compose. Path or an ordered list of paths to Docker Compose files relative to the devcontainer.json file. Using an array is useful when extending your Docker Compose configuration. The order of the array matters since the contents of later files can override values set in previous ones.
The default .env file is picked up from the root of the project, but you can use env_file in your Docker Compose file to specify an alternate location.
Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array properties.
service	string	Required when using Docker Compose. The name of the service devcontainer.json supporting services / tools should connect to once running.
runServices	array	An array of services in your Docker Compose configuration that should be started by devcontainer.json supporting services / tools. These will also be stopped when you disconnect unless "shutdownAction" is "none". Defaults to all services.
workspaceFolder	string	Sets the default path that devcontainer.json supporting services / tools should open when connecting to the container (which is often the path to a volume mount where the source code can be found in the container). Defaults to "/".
Tool-specific properties
While most properties apply to any devcontainer.json supporting tool or service, a few are specific to certain tools. You may explore this in the supporting tools and services document.

Lifecycle scripts
When creating or working with a dev container, you may need different commands to be run at different points in the container’s lifecycle. The table below lists a set of command properties you can use to update what the container’s contents in the order in which they are run (for example, onCreateCommand will run after initializeCommand). Each command property is an string or list of command arguments that should execute from the workspaceFolder.

Property	Type	Description
initializeCommand	string,
array,
object	A command string or list of command arguments to run on the host machine during initialization, including during container creation and on subsequent starts. The command may run more than once during a given session.

⚠️ The command is run wherever the source code is located on the host. For cloud services, this is in the cloud.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
onCreateCommand 🏷️	string,
array,
object	This command is the first of three (along with updateContentCommand and postCreateCommand) that finalizes container setup when a dev container is created. It and subsequent commands execute inside the container immediately after it has started for the first time.

Cloud services can use this command when caching or prebuilding a container. This means that it will not typically have access to user-scoped assets or secrets.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
updateContentCommand 🏷️	string,
array,
object	This command is the second of three that finalizes container setup when a dev container is created. It executes inside the container after onCreateCommand whenever new content is available in the source tree during the creation process.

It will execute at least once, but cloud services will also periodically execute the command to refresh cached or prebuilt containers. Like cloud services using onCreateCommand, it can only take advantage of repository and org scoped secrets or permissions.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postCreateCommand 🏷️	string,
array,
object	This command is the last of three that finalizes container setup when a dev container is created. It happens after updateContentCommand and once the dev container has been assigned to a user for the first time.

Cloud services can use this command to take advantage of user specific secrets and permissions.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postStartCommand 🏷️	string,
array,
object	A command to run each time the container is successfully started.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postAttachCommand 🏷️	string,
array,
object	A command to run each time a tool has successfully attached to the container.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
waitFor 🏷️	enum	An enum that specifies the command any tool should wait for before connecting. Defaults to updateContentCommand. This allows you to use onCreateCommand or updateContentCommand for steps that must happen before devcontainer.json supporting tools connect while still using postCreateCommand for steps that can happen behind the scenes afterwards.
For each command property, if the value is a single string, it will be run in /bin/sh. Use && in a string to execute multiple commands. For example, "yarn install" or "apt-get update && apt-get install -y curl". The array syntax ["yarn", "install"] will invoke the command (in this case yarn) directly without using a shell. Each fires after your source code has been mounted, so you can also run shell scripts from your source tree. For example: bash scripts/install-dev-tools.sh.

If one of the lifecycle scripts fails, any subsequent scripts will not be executed. For instance, if postCreateCommand fails, postStartCommand and any following scripts will be skipped.

Minimum host requirements
While devcontainer.json does not focus on hardware or VM provisioning, it can be useful to know your container’s minimum RAM, CPU, and storage requirements. This is what the hostRequirements properties allow you to do. Cloud services can use these properties to automatically default to the best compute option available, while in other cases, you will be presented with a warning if the requirements are not met.

Property	Type	Description
hostRequirements.cpus 🏷️	integer	Indicates the minimum required number of CPUs / virtual CPUs / cores. For example: "hostRequirements": {"cpus": 2}
hostRequirements.memory 🏷️	string	A string indicating minimum memory requirements with a tb, gb, mb, or kb suffix. For example, "hostRequirements": {"memory": "4gb"}
hostRequirements.storage 🏷️	string	A string indicating minimum storage requirements with a tb, gb, mb, or kb suffix. For example, "hostRequirements": {"storage": "32gb"}
Port attributes
The portsAttributes and otherPortsAttributes properties allow you to map default port options for one or more manually or automatically forwarded ports. The following is a list of options that can be set in the configuration object assigned to the property.

Property	Type	Description
label 🏷️	string	Display name for the port in the ports view. Defaults to not set.
protocol 🏷️	enum	Controls protocol handling for forwarded ports. When not set, the port is assumed to be a raw TCP stream which, if forwarded to localhost, supports any number of protocols. However, if the port is forwarded to a web URL (e.g. from a cloud service on the web), only HTTP ports in the container are supported. Setting this property to https alters handling by ignoring any SSL/TLS certificates present when communicating on the port and using the correct certificate for the forwarded URL instead (e.g https://*.githubpreview.dev). If set to http, processing is the same as if the protocol is not set. Defaults to not set.
onAutoForward 🏷️	enum	Controls what should happen when a port is auto-forwarded once you’ve connected to the container. notify is the default, and a notification will appear when the port is auto-forwarded. If set to openBrowser, the port will be opened in the system’s default browser. A value of openBrowserOnce will open the browser only once. openPreview will open the URL in devcontainer.json supporting services’ / tools’ embedded preview browser. A value of silent will forward the port, but take no further action. A value of ignore means that this port should not be auto-forwarded at all.
requireLocalPort 🏷️	boolean	Dictates when port forwarding is required to map the port in the container to the same port locally or not. If set to false, the devcontainer.json supporting services / tools will attempt to use the specified port forward to localhost, and silently map to a different one if it is unavailable. If set to true, you will be notified if it is not possible to use the same port. Defaults to false.
elevateIfNeeded 🏷️	boolean	Forwarding low ports like 22, 80, or 443 to localhost on the same port from devcontainer.json supporting services / tools may require elevated permissions on certain operating systems. Setting this property to true will automatically try to elevate the devcontainer.json supporting tool’s permissions in this situation. Defaults to false.
Formatting string vs. array properties
The format of certain properties will vary depending on the involvement of a shell.

postCreateCommand, postStartCommand, postAttachCommand, and initializeCommand all have 3 types:

Array: Passed to the OS for execution without going through a shell
String: Goes through a shell (it needs to be parsed into command and arguments)
Object: All lifecycle scripts have been extended to support object types to allow for parallel execution
runArgs only has the array type. Using runArgs via a typical command line, you’ll need single quotes if the shell runs into parameters with spaces. However, these single quotes aren’t passed on to the executable. Thus, in your devcontainer.json, you’d follow the array format and leave out the single quotes:

"runArgs": ["--device-cgroup-rule=my rule here"]
Rather than:

"runArgs": ["--device-cgroup-rule='my rule here'"]
We can compare the string, array, and object versions of postAttachCommand as well. You can use the following string format, which will remove the single quotes as part of the shell’s parsing:

"postAttachCommand": "echo foo='bar'"
By contrast, the array format will keep the single quotes and write them to standard out (you can see the output in the dev container log):

"postAttachCommand": ["echo", "foo='bar'"]
Finally, you may use an object format:

{
  "postAttachCommand": {
    "server": "npm start",
    "db": ["mysql", "-u", "root", "-p", "my database"]
  }
}
Variables in devcontainer.json
Variables can be referenced in certain string values in devcontainer.json in the following format: ${variableName}. The following is a list of available variables you can use.

Variable	Properties	Description
${localEnv:VARIABLE_NAME}	Any	Value of an environment variable on the host machine (in the examples below, called VARIABLE_NAME). Unset variables are left blank.

⚠️ Clients (like VS Code) may need to be restarted to pick up newly set variables.

⚠️ For a cloud service, the host is in the cloud rather than your local machine.

Examples

1. Set a variable containing your local home folder on Linux / macOS or the user folder on Windows:
"remoteEnv": { "LOCAL_USER_PATH": "${localEnv:HOME}${localEnv:USERPROFILE}" }.

A default value for when the environment variable is not set can be given with ${localEnv:VARIABLE_NAME:default_value}.

2. In modern versions of macOS, default configurations allow setting local variables with the command echo 'export VARIABLE_NAME=my-value' >> ~/.zshenv.
${containerEnv:VARIABLE_NAME}	remoteEnv	Value of an existing environment variable inside the container once it is up and running (in this case, called VARIABLE_NAME). For example:
"remoteEnv": { "PATH": "${containerEnv:PATH}:/some/other/path" }

A default value for when the environment variable is not set can be given with ${containerEnv:VARIABLE_NAME:default_value}.
${localWorkspaceFolder}	Any	Path of the local folder that was opened in the devcontainer.json supporting service / tool (that contains .devcontainer/devcontainer.json).
${containerWorkspaceFolder}	Any	The path that the workspaces files can be found in the container.
${localWorkspaceFolderBasename}	Any	Name of the local folder that was opened in the devcontainer.json supporting service / tool (that contains .devcontainer/devcontainer.json).
${containerWorkspaceFolderBasename}	Any	Name of the folder where the workspace files can be found in the container.
${devcontainerId}	Any	Allow Features to refer to an identifier that is unique to the dev container they are installed into and that is stable across rebuilds.
The properties supporting it in devcontainer.json are: name, runArgs, initializeCommand, onCreateCommand, updateContentCommand, postCreateCommand, postStartCommand, postAttachCommand, workspaceFolder, workspaceMount, mounts, containerEnv, remoteEnv, containerUser, remoteUser, and customizations.
Schema
You can see the dev container schema here.

Publishing vs forwarding ports
Docker has the concept of “publishing” ports when the container is created. Published ports behave very much like ports you make available to your local network. If your application only accepts calls from localhost, it will reject connections from published ports just as your local machine would for network calls. Forwarded ports, on the other hand, actually look like localhost to the application.

remoteUser
A dev container configuration will inherit the remoteUser property from the base image it uses.

Using the images and Templates part of the spec as an example: remoteUser in these images is set to a custom value - you may view an example in the C++ image. The C++ Template will then inherit the custom remoteUser value from its base C++ image.

Microsoft
© 2024 MicrosoftDevelopment Containers

Topics 
Dev Container metadata reference
Specification
Dev Container metadata reference
The devcontainer.json file contains any needed metadata and settings required to configurate a development container for a given well-defined tool and runtime stack. It can be used by tools and services that support the dev container spec to create a development environment that contains one or more development containers.

Metadata properties marked with a 🏷️️ can be stored in the devcontainer.metadata container image label in addition to devcontainer.json. This label can contain an array of json snippets that will be automatically merged with devcontainer.json contents (if any) when a container is created.

General devcontainer.json properties
Property	Type	Description
name	string	A name for the dev container displayed in the UI.
forwardPorts 🏷️	array	An array of port numbers or "host:port" values (e.g. [3000, "db:5432"]) that should always be forwarded from inside the primary container to the local machine (including on the web). The property is most useful for forwarding ports that cannot be auto-forwarded because the related process that starts before the devcontainer.json supporting service / tool connects or for forwarding a service not in the primary container in Docker Compose scenarios (e.g. "db:5432"). Defaults to [].
portsAttributes 🏷️	object	Object that maps a port number, "host:port" value, range, or regular expression to a set of default options. See port attributes for available options. For example:
"portsAttributes": {"3000": {"label": "Application port"}}
otherPortsAttributes 🏷️	object	Default options for ports, port ranges, and hosts that aren’t configured using portsAttributes. See port attributes for available options. For example:
"otherPortsAttributes": {"onAutoForward": "silent"}
containerEnv 🏷️	object	A set of name-value pairs that sets or overrides environment variables for the container. Environment and pre-defined variables may be referenced in the values. For example:
"containerEnv": { "MY_VARIABLE": "${localEnv:MY_VARIABLE}" }
If you want to reference an existing container variable while setting this one (like updating the PATH), use remoteEnv instead.
containerEnv will set the variable on the Docker container itself, so all processes spawned in the container will have access to it. But it will also be static for the life of the container - you must rebuild the container to update the value.
We recommend using containerEnv (over remoteEnv) as much as possible since it allows all processes to see the variable and isn’t client-specific.
remoteEnv 🏷️	object	A set of name-value pairs that sets or overrides environment variables for the devcontainer.json supporting service / tool (or sub-processes like terminals) but not the container as a whole. Environment and pre-defined variables may be referenced in the values.
You may want to use remoteEnv (over containerEnv) if the value isn’t static since you can update its value without having to rebuild the full container.
remoteUser 🏷️	string	Overrides the user that devcontainer.json supporting services tools / runs as in the container (along with sub-processes like terminals, tasks, or debugging). Does not change the user the container as a whole runs as which can be set using containerUser. Defaults to the user the container as a whole is running as (often root).
You may learn more in the remoteUser section below.
containerUser 🏷️	string	Overrides the user for all operations run as inside the container. Defaults to either root or the last USER instruction in the related Dockerfile used to create the image. If you want any connected tools or related processes to use a different user than the one for the container, see remoteUser.
updateRemoteUserUID 🏷️	boolean	On Linux, if containerUser or remoteUser is specified, the user’s UID/GID will be updated to match the local user’s UID/GID to avoid permission problems with bind mounts. Defaults to true.
userEnvProbe 🏷️	enum	Indicates the type of shell to use to “probe” for user environment variables to include in devcontainer.json supporting services’ / tools’ processes: none, interactiveShell, loginShell, or loginInteractiveShell (default). The specific shell used is based on the default shell for the user (typically bash). For example, bash interactive shells will typically include variables set in /etc/bash.bashrc and ~/.bashrc while login shells usually include variables from /etc/profile and ~/.profile. Setting this property to loginInteractiveShell will get variables from all four files.
overrideCommand 🏷️	boolean	Tells devcontainer.json supporting services / tools whether they should run /bin/sh -c "while sleep 1000; do :; done" when starting the container instead of the container’s default command (since the container can shut down if the default command fails). Set to false if the default command must run for the container to function properly. Defaults to true for when using an image Dockerfile and false when referencing a Docker Compose file.
shutdownAction 🏷️	enum	Indicates whether devcontainer.json supporting tools should stop the containers when the related tool window is closed / shut down.
Values are none, stopContainer (default for image or Dockerfile), and stopCompose (default for Docker Compose).
init 🏷️	boolean	Defaults to false. Cross-orchestrator way to indicate whether the tini init process should be used to help deal with zombie processes.
privileged 🏷️	boolean	Defaults to false. Cross-orchestrator way to cause the container to run in privileged mode (--privileged). Required for things like Docker-in-Docker, but has security implications particularly when running directly on Linux.
capAdd 🏷️	array	Defaults to []. Cross-orchestrator way to add capabilities typically disabled for a container. Most often used to add the ptrace capability required to debug languages like C++, Go, and Rust. For example:
"capAdd": ["SYS_PTRACE"]
securityOpt 🏷️	array	Defaults to []. Cross-orchestrator way to set container security options. For example:
"securityOpt": [ "seccomp=unconfined" ]
mounts 🏷️	string or object	Defaults to unset. Cross-orchestrator way to add additional mounts to a container. Each value is a string that accepts the same values as the Docker CLI --mount flag. Environment and pre-defined variables may be referenced in the value. For example:
"mounts": [{ "source": "dind-var-lib-docker", "target": "/var/lib/docker", "type": "volume" }]
features	object	An object of Dev Container Feature IDs and related options to be added into your primary container. The specific options that are available varies by feature, so see its documentation for additional details. For example:
"features": { "ghcr.io/devcontainers/features/github-cli": {} }
overrideFeatureInstallOrder	array	By default, Features will attempt to automatically set the order they are installed based on a installsAfter property within each of them. This property allows you to override the Feature install order when needed. For example:
"overrideFeatureInstallОrder": [ "ghcr.io/devcontainers/features/common-utils", "ghcr.io/devcontainers/features/github-cli" ]
customizations 🏷️	object	Product specific properties, defined in supporting tools
Scenario specific properties
The focus of devcontainer.json is to describe how to enrich a container for the purposes of development rather than acting as a multi-container orchestrator format. Instead, container orchestrator formats can be referenced when needed to manage multiple containers and their lifecycles. Today, devcontainer.json includes scenario specific properties for working without a container orchestrator (by directly referencing an image or Dockerfile) and for using Docker Compose as a simple multi-container orchestrator.

Image or Dockerfile specific properties
Property	Type	Description
image	string	Required when using an image. The name of an image in a container registry (DockerHub, GitHub Container Registry, Azure Container Registry) that devcontainer.json supporting services / tools should use to create the dev container.
build.dockerfile	string	Required when using a Dockerfile. The location of a Dockerfile that defines the contents of the container. The path is relative to the devcontainer.json file.
build.context	string	Path that the Docker build should be run from relative to devcontainer.json. For example, a value of ".." would allow you to reference content in sibling directories. Defaults to ".".
build.args	Object	A set of name-value pairs containing Docker image build arguments that should be passed when building a Dockerfile. Environment and pre-defined variables may be referenced in the values. Defaults to not set. For example: "build": { "args": { "MYARG": "MYVALUE", "MYARGFROMENVVAR": "${localEnv:VARIABLE_NAME}" } }
build.options	array	An array of Docker image build options that should be passed to the build command when building a Dockerfile. Defaults to []. For example: "build": { "options": [ "--add-host=host.docker.internal:host-gateway" ] }
build.target	string	A string that specifies a Docker image build target that should be passed when building a Dockerfile. Defaults to not set. For example: "build": { "target": "development" }
build.cacheFrom	string,
array	A string or array of strings that specify one or more images to use as caches when building the image. Cached image identifiers are passed to the docker build command with --cache-from.
appPort	integer,
string,
array	In most cases, we recommend using the new forwardPorts property. This property accepts a port or array of ports that should be published locally when the container is running. Unlike forwardPorts, your application may need to listen on all interfaces (0.0.0.0) not just localhost for it to be available externally. Defaults to [].
Learn more about publishing vs forwarding ports here.
Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array properties.
workspaceMount	string	Requires workspaceFolder be set as well. Overrides the default local mount point for the workspace when the container is created. Supports the same values as the Docker CLI --mount flag. Environment and pre-defined variables may be referenced in the value. For example:
"workspaceMount": "source=${localWorkspaceFolder}/sub-folder,target=/workspace,type=bind,consistency=cached", "workspaceFolder": "/workspace"
workspaceFolder	string	Requires workspaceMount be set. Sets the default path that devcontainer.json supporting services / tools should open when connecting to the container. Defaults to the automatic source code mount location.
runArgs	array	An array of Docker CLI arguments that should be used when running the container. Defaults to []. For example, this allows ptrace based debuggers like C++ to work in the container:
"runArgs": [ "--cap-add=SYS_PTRACE", "--security-opt", "seccomp=unconfined" ] .
Docker Compose specific properties
Property	Type	Description
dockerComposeFile	string,
array	Required when using Docker Compose. Path or an ordered list of paths to Docker Compose files relative to the devcontainer.json file. Using an array is useful when extending your Docker Compose configuration. The order of the array matters since the contents of later files can override values set in previous ones.
The default .env file is picked up from the root of the project, but you can use env_file in your Docker Compose file to specify an alternate location.
Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array properties.
service	string	Required when using Docker Compose. The name of the service devcontainer.json supporting services / tools should connect to once running.
runServices	array	An array of services in your Docker Compose configuration that should be started by devcontainer.json supporting services / tools. These will also be stopped when you disconnect unless "shutdownAction" is "none". Defaults to all services.
workspaceFolder	string	Sets the default path that devcontainer.json supporting services / tools should open when connecting to the container (which is often the path to a volume mount where the source code can be found in the container). Defaults to "/".
Tool-specific properties
While most properties apply to any devcontainer.json supporting tool or service, a few are specific to certain tools. You may explore this in the supporting tools and services document.

Lifecycle scripts
When creating or working with a dev container, you may need different commands to be run at different points in the container’s lifecycle. The table below lists a set of command properties you can use to update what the container’s contents in the order in which they are run (for example, onCreateCommand will run after initializeCommand). Each command property is an string or list of command arguments that should execute from the workspaceFolder.

Property	Type	Description
initializeCommand	string,
array,
object	A command string or list of command arguments to run on the host machine during initialization, including during container creation and on subsequent starts. The command may run more than once during a given session.

⚠️ The command is run wherever the source code is located on the host. For cloud services, this is in the cloud.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
onCreateCommand 🏷️	string,
array,
object	This command is the first of three (along with updateContentCommand and postCreateCommand) that finalizes container setup when a dev container is created. It and subsequent commands execute inside the container immediately after it has started for the first time.

Cloud services can use this command when caching or prebuilding a container. This means that it will not typically have access to user-scoped assets or secrets.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
updateContentCommand 🏷️	string,
array,
object	This command is the second of three that finalizes container setup when a dev container is created. It executes inside the container after onCreateCommand whenever new content is available in the source tree during the creation process.

It will execute at least once, but cloud services will also periodically execute the command to refresh cached or prebuilt containers. Like cloud services using onCreateCommand, it can only take advantage of repository and org scoped secrets or permissions.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postCreateCommand 🏷️	string,
array,
object	This command is the last of three that finalizes container setup when a dev container is created. It happens after updateContentCommand and once the dev container has been assigned to a user for the first time.

Cloud services can use this command to take advantage of user specific secrets and permissions.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postStartCommand 🏷️	string,
array,
object	A command to run each time the container is successfully started.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
postAttachCommand 🏷️	string,
array,
object	A command to run each time a tool has successfully attached to the container.

Note that the array syntax will execute the command without a shell. You can learn more about formatting string vs array vs object properties.
waitFor 🏷️	enum	An enum that specifies the command any tool should wait for before connecting. Defaults to updateContentCommand. This allows you to use onCreateCommand or updateContentCommand for steps that must happen before devcontainer.json supporting tools connect while still using postCreateCommand for steps that can happen behind the scenes afterwards.
For each command property, if the value is a single string, it will be run in /bin/sh. Use && in a string to execute multiple commands. For example, "yarn install" or "apt-get update && apt-get install -y curl". The array syntax ["yarn", "install"] will invoke the command (in this case yarn) directly without using a shell. Each fires after your source code has been mounted, so you can also run shell scripts from your source tree. For example: bash scripts/install-dev-tools.sh.

If one of the lifecycle scripts fails, any subsequent scripts will not be executed. For instance, if postCreateCommand fails, postStartCommand and any following scripts will be skipped.

Minimum host requirements
While devcontainer.json does not focus on hardware or VM provisioning, it can be useful to know your container’s minimum RAM, CPU, and storage requirements. This is what the hostRequirements properties allow you to do. Cloud services can use these properties to automatically default to the best compute option available, while in other cases, you will be presented with a warning if the requirements are not met.

Property	Type	Description
hostRequirements.cpus 🏷️	integer	Indicates the minimum required number of CPUs / virtual CPUs / cores. For example: "hostRequirements": {"cpus": 2}
hostRequirements.memory 🏷️	string	A string indicating minimum memory requirements with a tb, gb, mb, or kb suffix. For example, "hostRequirements": {"memory": "4gb"}
hostRequirements.storage 🏷️	string	A string indicating minimum storage requirements with a tb, gb, mb, or kb suffix. For example, "hostRequirements": {"storage": "32gb"}
Port attributes
The portsAttributes and otherPortsAttributes properties allow you to map default port options for one or more manually or automatically forwarded ports. The following is a list of options that can be set in the configuration object assigned to the property.

Property	Type	Description
label 🏷️	string	Display name for the port in the ports view. Defaults to not set.
protocol 🏷️	enum	Controls protocol handling for forwarded ports. When not set, the port is assumed to be a raw TCP stream which, if forwarded to localhost, supports any number of protocols. However, if the port is forwarded to a web URL (e.g. from a cloud service on the web), only HTTP ports in the container are supported. Setting this property to https alters handling by ignoring any SSL/TLS certificates present when communicating on the port and using the correct certificate for the forwarded URL instead (e.g https://*.githubpreview.dev). If set to http, processing is the same as if the protocol is not set. Defaults to not set.
onAutoForward 🏷️	enum	Controls what should happen when a port is auto-forwarded once you’ve connected to the container. notify is the default, and a notification will appear when the port is auto-forwarded. If set to openBrowser, the port will be opened in the system’s default browser. A value of openBrowserOnce will open the browser only once. openPreview will open the URL in devcontainer.json supporting services’ / tools’ embedded preview browser. A value of silent will forward the port, but take no further action. A value of ignore means that this port should not be auto-forwarded at all.
requireLocalPort 🏷️	boolean	Dictates when port forwarding is required to map the port in the container to the same port locally or not. If set to false, the devcontainer.json supporting services / tools will attempt to use the specified port forward to localhost, and silently map to a different one if it is unavailable. If set to true, you will be notified if it is not possible to use the same port. Defaults to false.
elevateIfNeeded 🏷️	boolean	Forwarding low ports like 22, 80, or 443 to localhost on the same port from devcontainer.json supporting services / tools may require elevated permissions on certain operating systems. Setting this property to true will automatically try to elevate the devcontainer.json supporting tool’s permissions in this situation. Defaults to false.
Formatting string vs. array properties
The format of certain properties will vary depending on the involvement of a shell.

postCreateCommand, postStartCommand, postAttachCommand, and initializeCommand all have 3 types:

Array: Passed to the OS for execution without going through a shell
String: Goes through a shell (it needs to be parsed into command and arguments)
Object: All lifecycle scripts have been extended to support object types to allow for parallel execution
runArgs only has the array type. Using runArgs via a typical command line, you’ll need single quotes if the shell runs into parameters with spaces. However, these single quotes aren’t passed on to the executable. Thus, in your devcontainer.json, you’d follow the array format and leave out the single quotes:

"runArgs": ["--device-cgroup-rule=my rule here"]
Rather than:

"runArgs": ["--device-cgroup-rule='my rule here'"]
We can compare the string, array, and object versions of postAttachCommand as well. You can use the following string format, which will remove the single quotes as part of the shell’s parsing:

"postAttachCommand": "echo foo='bar'"
By contrast, the array format will keep the single quotes and write them to standard out (you can see the output in the dev container log):

"postAttachCommand": ["echo", "foo='bar'"]
Finally, you may use an object format:

{
  "postAttachCommand": {
    "server": "npm start",
    "db": ["mysql", "-u", "root", "-p", "my database"]
  }
}
Variables in devcontainer.json
Variables can be referenced in certain string values in devcontainer.json in the following format: ${variableName}. The following is a list of available variables you can use.

Variable	Properties	Description
${localEnv:VARIABLE_NAME}	Any	Value of an environment variable on the host machine (in the examples below, called VARIABLE_NAME). Unset variables are left blank.

⚠️ Clients (like VS Code) may need to be restarted to pick up newly set variables.

⚠️ For a cloud service, the host is in the cloud rather than your local machine.

Examples

1. Set a variable containing your local home folder on Linux / macOS or the user folder on Windows:
"remoteEnv": { "LOCAL_USER_PATH": "${localEnv:HOME}${localEnv:USERPROFILE}" }.

A default value for when the environment variable is not set can be given with ${localEnv:VARIABLE_NAME:default_value}.

2. In modern versions of macOS, default configurations allow setting local variables with the command echo 'export VARIABLE_NAME=my-value' >> ~/.zshenv.
${containerEnv:VARIABLE_NAME}	remoteEnv	Value of an existing environment variable inside the container once it is up and running (in this case, called VARIABLE_NAME). For example:
"remoteEnv": { "PATH": "${containerEnv:PATH}:/some/other/path" }

A default value for when the environment variable is not set can be given with ${containerEnv:VARIABLE_NAME:default_value}.
${localWorkspaceFolder}	Any	Path of the local folder that was opened in the devcontainer.json supporting service / tool (that contains .devcontainer/devcontainer.json).
${containerWorkspaceFolder}	Any	The path that the workspaces files can be found in the container.
${localWorkspaceFolderBasename}	Any	Name of the local folder that was opened in the devcontainer.json supporting service / tool (that contains .devcontainer/devcontainer.json).
${containerWorkspaceFolderBasename}	Any	Name of the folder where the workspace files can be found in the container.
${devcontainerId}	Any	Allow Features to refer to an identifier that is unique to the dev container they are installed into and that is stable across rebuilds.
The properties supporting it in devcontainer.json are: name, runArgs, initializeCommand, onCreateCommand, updateContentCommand, postCreateCommand, postStartCommand, postAttachCommand, workspaceFolder, workspaceMount, mounts, containerEnv, remoteEnv, containerUser, remoteUser, and customizations.
Schema
You can see the dev container schema here.

Publishing vs forwarding ports
Docker has the concept of “publishing” ports when the container is created. Published ports behave very much like ports you make available to your local network. If your application only accepts calls from localhost, it will reject connections from published ports just as your local machine would for network calls. Forwarded ports, on the other hand, actually look like localhost to the application.

remoteUser
A dev container configuration will inherit the remoteUser property from the base image it uses.

Using the images and Templates part of the spec as an example: remoteUser in these images is set to a custom value - you may view an example in the C++ image. The C++ Template will then inherit the custom remoteUser value from its base C++ image.

Microsoft
© 2024 MicrosoftGit--everything-is-local
Search entire site...

About
Documentation
Reference
Book
Videos
External Links
Downloads
Community
English ▾Topics ▾Version 2.44.0 ▾
gitignore last updated in 2.44.0
NAME
gitignore - Specifies intentionally untracked files to ignore

SYNOPSIS
$XDG_CONFIG_HOME/git/ignore, $GIT_DIR/info/exclude, .gitignore

DESCRIPTION
A gitignore file specifies intentionally untracked files that Git should ignore. Files already tracked by Git are not affected; see the NOTES below for details.

Each line in a gitignore file specifies a pattern. When deciding whether to ignore a path, Git normally checks gitignore patterns from multiple sources, with the following order of precedence, from highest to lowest (within one level of precedence, the last matching pattern decides the outcome):

Patterns read from the command line for those commands that support them.

Patterns read from a .gitignore file in the same directory as the path, or in any parent directory (up to the top-level of the working tree), with patterns in the higher level files being overridden by those in lower level files down to the directory containing the file. These patterns match relative to the location of the .gitignore file. A project normally includes such .gitignore files in its repository, containing patterns for files generated as part of the project build.

Patterns read from $GIT_DIR/info/exclude.

Patterns read from the file specified by the configuration variable core.excludesFile.

Which file to place a pattern in depends on how the pattern is meant to be used.

Patterns which should be version-controlled and distributed to other repositories via clone (i.e., files that all developers will want to ignore) should go into a .gitignore file.

Patterns which are specific to a particular repository but which do not need to be shared with other related repositories (e.g., auxiliary files that live inside the repository but are specific to one user’s workflow) should go into the $GIT_DIR/info/exclude file.

Patterns which a user wants Git to ignore in all situations (e.g., backup or temporary files generated by the user’s editor of choice) generally go into a file specified by core.excludesFile in the user’s ~/.gitconfig. Its default value is $XDG_CONFIG_HOME/git/ignore. If $XDG_CONFIG_HOME is either not set or empty, $HOME/.config/git/ignore is used instead.

The underlying Git plumbing tools, such as git ls-files and git read-tree, read gitignore patterns specified by command-line options, or from files specified by command-line options. Higher-level Git tools, such as git status and git add, use patterns from the sources specified above.

PATTERN FORMAT
A blank line matches no files, so it can serve as a separator for readability.

A line starting with # serves as a comment. Put a backslash ("\") in front of the first hash for patterns that begin with a hash.

Trailing spaces are ignored unless they are quoted with backslash ("\").

An optional prefix "!" which negates the pattern; any matching file excluded by a previous pattern will become included again. It is not possible to re-include a file if a parent directory of that file is excluded. Git doesn’t list excluded directories for performance reasons, so any patterns on contained files have no effect, no matter where they are defined. Put a backslash ("\") in front of the first "!" for patterns that begin with a literal "!", for example, "\!important!.txt".

The slash "/" is used as the directory separator. Separators may occur at the beginning, middle or end of the .gitignore search pattern.

If there is a separator at the beginning or middle (or both) of the pattern, then the pattern is relative to the directory level of the particular .gitignore file itself. Otherwise the pattern may also match at any level below the .gitignore level.

If there is a separator at the end of the pattern then the pattern will only match directories, otherwise the pattern can match both files and directories.

For example, a pattern doc/frotz/ matches doc/frotz directory, but not a/doc/frotz directory; however frotz/ matches frotz and a/frotz that is a directory (all paths are relative from the .gitignore file).

An asterisk "*" matches anything except a slash. The character "?" matches any one character except "/". The range notation, e.g. [a-zA-Z], can be used to match one of the characters in a range. See fnmatch(3) and the FNM_PATHNAME flag for a more detailed description.

Two consecutive asterisks ("**") in patterns matched against full pathname may have special meaning:

A leading "**" followed by a slash means match in all directories. For example, "**/foo" matches file or directory "foo" anywhere, the same as pattern "foo". "**/foo/bar" matches file or directory "bar" anywhere that is directly under directory "foo".

A trailing "/**" matches everything inside. For example, "abc/**" matches all files inside directory "abc", relative to the location of the .gitignore file, with infinite depth.

A slash followed by two consecutive asterisks then a slash matches zero or more directories. For example, "a/**/b" matches "a/b", "a/x/b", "a/x/y/b" and so on.

Other consecutive asterisks are considered regular asterisks and will match according to the previous rules.

CONFIGURATION
The optional configuration variable core.excludesFile indicates a path to a file containing patterns of file names to exclude, similar to $GIT_DIR/info/exclude. Patterns in the exclude file are used in addition to those in $GIT_DIR/info/exclude.

NOTES
The purpose of gitignore files is to ensure that certain files not tracked by Git remain untracked.

To stop tracking a file that is currently tracked, use git rm --cached to remove the file from the index. The filename can then be added to the .gitignore file to stop the file from being reintroduced in later commits.

Git does not follow symbolic links when accessing a .gitignore file in the working tree. This keeps behavior consistent when the file is accessed from the index or a tree versus from the filesystem.

EXAMPLES
The pattern hello.* matches any file or directory whose name begins with hello.. If one wants to restrict this only to the directory and not in its subdirectories, one can prepend the pattern with a slash, i.e. /hello.*; the pattern now matches hello.txt, hello.c but not a/hello.java.

The pattern foo/ will match a directory foo and paths underneath it, but will not match a regular file or a symbolic link foo (this is consistent with the way how pathspec works in general in Git)

The pattern doc/frotz and /doc/frotz have the same effect in any .gitignore file. In other words, a leading slash is not relevant if there is already a middle slash in the pattern.

The pattern foo/*, matches foo/test.json (a regular file), foo/bar (a directory), but it does not match foo/bar/hello.c (a regular file), as the asterisk in the pattern does not match bar/hello.c which has a slash in it.

    $ git status
    [...]
    # Untracked files:
    [...]
    #       Documentation/foo.html
    #       Documentation/gitignore.html
    #       file.o
    #       lib.a
    #       src/internal.o
    [...]
    $ cat .git/info/exclude
    # ignore objects and archives, anywhere in the tree.
    *.[oa]
    $ cat Documentation/.gitignore
    # ignore generated html files,
    *.html
    # except foo.html which is maintained by hand
    !foo.html
    $ git status
    [...]
    # Untracked files:
    [...]
    #       Documentation/foo.html
    [...]
Another example:

    $ cat .gitignore
    vmlinux*
    $ ls arch/foo/kernel/vm*
    arch/foo/kernel/vmlinux.lds.S
    $ echo '!/vmlinux*' >arch/foo/kernel/.gitignore
The second .gitignore prevents Git from ignoring arch/foo/kernel/vmlinux.lds.S.

Example to exclude everything except a specific directory foo/bar (note the /* - without the slash, the wildcard would also exclude everything within foo/bar):

    $ cat .gitignore
    # exclude everything except directory foo/bar
    /*
    !/foo
    /foo/*
    !/foo/bar
SEE ALSO
git-rm[1], gitrepository-layout[5], git-check-ignore[1]

GIT
Part of the git[1] suite

About this site
Patches, suggestions, and comments are welcome.
Git is a member of Software Freedom Conservancy
scroll-to-topGit--local-branching-on-the-cheap
Search entire site...

About
Documentation
Reference
Book
Videos
External Links
Downloads
Community
This book is available in English.

Full translation available in

azərbaycan dili,
български език,
Deutsch,
Español,
Français,
Ελληνικά,
日本語,
한국어,
Nederlands,
Русский,
Slovenščina,
Tagalog,
Українська
简体中文,
Partial translations available in

Čeština,
Македонски,
Polski,
Српски,
Ўзбекча,
繁體中文,
Translations started for

Беларуская,
فارسی,
Indonesian,
Italiano,
Bahasa Melayu,
Português (Brasil),
Português (Portugal),
Svenska,
Türkçe.
The source of this book is hosted on GitHub.
Patches, suggestions and comments are welcome.

Chapters ▾ 2nd Edition
3.6 Git Branching - Rebasing
Rebasing
In Git, there are two main ways to integrate changes from one branch into another: the merge and the rebase. In this section you’ll learn what rebasing is, how to do it, why it’s a pretty amazing tool, and in what cases you won’t want to use it.

The Basic Rebase
If you go back to an earlier example from Basic Merging, you can see that you diverged your work and made commits on two different branches.

Simple divergent history
Figure 35. Simple divergent history
The easiest way to integrate the branches, as we’ve already covered, is the merge command. It performs a three-way merge between the two latest branch snapshots (C3 and C4) and the most recent common ancestor of the two (C2), creating a new snapshot (and commit).

Merging to integrate diverged work history
Figure 36. Merging to integrate diverged work history
However, there is another way: you can take the patch of the change that was introduced in C4 and reapply it on top of C3. In Git, this is called rebasing. With the rebase command, you can take all the changes that were committed on one branch and replay them on a different branch.

For this example, you would check out the experiment branch, and then rebase it onto the master branch as follows:

$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
This operation works by going to the common ancestor of the two branches (the one you’re on and the one you’re rebasing onto), getting the diff introduced by each commit of the branch you’re on, saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.

Rebasing the change introduced in `C4` onto `C3`
Figure 37. Rebasing the change introduced in C4 onto C3
At this point, you can go back to the master branch and do a fast-forward merge.

$ git checkout master
$ git merge experiment
Fast-forwarding the `master` branch
Figure 38. Fast-forwarding the master branch
Now, the snapshot pointed to by C4' is exactly the same as the one that was pointed to by C5 in the merge example. There is no difference in the end product of the integration, but rebasing makes for a cleaner history. If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.

Often, you’ll do this to make sure your commits apply cleanly on a remote branch — perhaps in a project to which you’re trying to contribute but that you don’t maintain. In this case, you’d do your work in a branch and then rebase your work onto origin/master when you were ready to submit your patches to the main project. That way, the maintainer doesn’t have to do any integration work — just a fast-forward or a clean apply.

Note that the snapshot pointed to by the final commit you end up with, whether it’s the last of the rebased commits for a rebase or the final merge commit after a merge, is the same snapshot — it’s only the history that is different. Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints and merges them together.

More Interesting Rebases
You can also have your rebase replay on something other than the rebase target branch. Take a history like A history with a topic branch off another topic branch, for example. You branched a topic branch (server) to add some server-side functionality to your project, and made a commit. Then, you branched off that to make the client-side changes (client) and committed a few times. Finally, you went back to your server branch and did a few more commits.

A history with a topic branch off another topic branch
Figure 39. A history with a topic branch off another topic branch
Suppose you decide that you want to merge your client-side changes into your mainline for a release, but you want to hold off on the server-side changes until it’s tested further. You can take the changes on client that aren’t on server (C8 and C9) and replay them on your master branch by using the --onto option of git rebase:

$ git rebase --onto master server client
This basically says, “Take the client branch, figure out the patches since it diverged from the server branch, and replay these patches in the client branch as if it was based directly off the master branch instead.” It’s a bit complex, but the result is pretty cool.

Rebasing a topic branch off another topic branch
Figure 40. Rebasing a topic branch off another topic branch
Now you can fast-forward your master branch (see Fast-forwarding your master branch to include the client branch changes):

$ git checkout master
$ git merge client
Fast-forwarding your `master` branch to include the `client` branch changes
Figure 41. Fast-forwarding your master branch to include the client branch changes
Let’s say you decide to pull in your server branch as well. You can rebase the server branch onto the master branch without having to check it out first by running git rebase <basebranch> <topicbranch> — which checks out the topic branch (in this case, server) for you and replays it onto the base branch (master):

$ git rebase master server
This replays your server work on top of your master work, as shown in Rebasing your server branch on top of your master branch.

Rebasing your `server` branch on top of your `master` branch
Figure 42. Rebasing your server branch on top of your master branch
Then, you can fast-forward the base branch (master):

$ git checkout master
$ git merge server
You can remove the client and server branches because all the work is integrated and you don’t need them anymore, leaving your history for this entire process looking like Final commit history:

$ git branch -d client
$ git branch -d server
Final commit history
Figure 43. Final commit history
The Perils of Rebasing
Ahh, but the bliss of rebasing isn’t without its drawbacks, which can be summed up in a single line:

Do not rebase commits that exist outside your repository and that people may have based work on.

If you follow that guideline, you’ll be fine. If you don’t, people will hate you, and you’ll be scorned by friends and family.

When you rebase stuff, you’re abandoning existing commits and creating new ones that are similar but different. If you push commits somewhere and others pull them down and base work on them, and then you rewrite those commits with git rebase and push them up again, your collaborators will have to re-merge their work and things will get messy when you try to pull their work back into yours.

Let’s look at an example of how rebasing work that you’ve made public can cause problems. Suppose you clone from a central server and then do some work off that. Your commit history looks like this:

Clone a repository, and base some work on it
Figure 44. Clone a repository, and base some work on it
Now, someone else does more work that includes a merge, and pushes that work to the central server. You fetch it and merge the new remote branch into your work, making your history look something like this:

Fetch more commits, and merge them into your work
Figure 45. Fetch more commits, and merge them into your work
Next, the person who pushed the merged work decides to go back and rebase their work instead; they do a git push --force to overwrite the history on the server. You then fetch from that server, bringing down the new commits.

Someone pushes rebased commits, abandoning commits you’ve based your work on
Figure 46. Someone pushes rebased commits, abandoning commits you’ve based your work on
Now you’re both in a pickle. If you do a git pull, you’ll create a merge commit which includes both lines of history, and your repository will look like this:

You merge in the same work again into a new merge commit
Figure 47. You merge in the same work again into a new merge commit
If you run a git log when your history looks like this, you’ll see two commits that have the same author, date, and message, which will be confusing. Furthermore, if you push this history back up to the server, you’ll reintroduce all those rebased commits to the central server, which can further confuse people. It’s pretty safe to assume that the other developer doesn’t want C4 and C6 to be in the history; that’s why they rebased in the first place.

Rebase When You Rebase
If you do find yourself in a situation like this, Git has some further magic that might help you out. If someone on your team force pushes changes that overwrite work that you’ve based work on, your challenge is to figure out what is yours and what they’ve rewritten.

It turns out that in addition to the commit SHA-1 checksum, Git also calculates a checksum that is based just on the patch introduced with the commit. This is called a “patch-id”.

If you pull down work that was rewritten and rebase it on top of the new commits from your partner, Git can often successfully figure out what is uniquely yours and apply them back on top of the new branch.

For instance, in the previous scenario, if instead of doing a merge when we’re at Someone pushes rebased commits, abandoning commits you’ve based your work on we run git rebase teamone/master, Git will:

Determine what work is unique to our branch (C2, C3, C4, C6, C7)

Determine which are not merge commits (C2, C3, C4)

Determine which have not been rewritten into the target branch (just C2 and C3, since C4 is the same patch as C4')

Apply those commits to the top of teamone/master

So instead of the result we see in You merge in the same work again into a new merge commit, we would end up with something more like Rebase on top of force-pushed rebase work.

Rebase on top of force-pushed rebase work
Figure 48. Rebase on top of force-pushed rebase work
This only works if C4 and C4' that your partner made are almost exactly the same patch. Otherwise the rebase won’t be able to tell that it’s a duplicate and will add another C4-like patch (which will probably fail to apply cleanly, since the changes would already be at least somewhat there).

You can also simplify this by running a git pull --rebase instead of a normal git pull. Or you could do it manually with a git fetch followed by a git rebase teamone/master in this case.

If you are using git pull and want to make --rebase the default, you can set the pull.rebase config value with something like git config --global pull.rebase true.

If you only ever rebase commits that have never left your own computer, you’ll be just fine. If you rebase commits that have been pushed, but that no one else has based commits from, you’ll also be fine. If you rebase commits that have already been pushed publicly, and people may have based work on those commits, then you may be in for some frustrating trouble, and the scorn of your teammates.

If you or a partner does find it necessary at some point, make sure everyone knows to run git pull --rebase to try to make the pain after it happens a little bit simpler.

Rebase vs. Merge
Now that you’ve seen rebasing and merging in action, you may be wondering which one is better. Before we can answer this, let’s step back a bit and talk about what history means.

One point of view on this is that your repository’s commit history is a record of what actually happened. It’s a historical document, valuable in its own right, and shouldn’t be tampered with. From this angle, changing the commit history is almost blasphemous; you’re lying about what actually transpired. So what if there was a messy series of merge commits? That’s how it happened, and the repository should preserve that for posterity.

The opposing point of view is that the commit history is the story of how your project was made. You wouldn’t publish the first draft of a book, so why show your messy work? When you’re working on a project, you may need a record of all your missteps and dead-end paths, but when it’s time to show your work to the world, you may want to tell a more coherent story of how to get from A to B. People in this camp use tools like rebase and filter-branch to rewrite their commits before they’re merged into the mainline branch. They use tools like rebase and filter-branch, to tell the story in the way that’s best for future readers.

Now, to the question of whether merging or rebasing is better: hopefully you’ll see that it’s not that simple. Git is a powerful tool, and allows you to do many things to and with your history, but every team and every project is different. Now that you know how both of these things work, it’s up to you to decide which one is best for your particular situation.

You can get the best of both worlds: rebase local changes before pushing to clean up your work, but never rebase anything that you’ve pushed somewhere.

prev | next
About this site
Patches, suggestions, and comments are welcome.
Git is a member of Software Freedom Conservancy
scroll-to-top---
title: Configuring issue templates for your repository
intro: You can customize the templates that are available for contributors to use when they open new issues in your repository.
redirect_from:
  - /github/building-a-strong-community/creating-issue-templates-for-your-repository
  - /articles/configuring-issue-templates-for-your-repository
  - /github/building-a-strong-community/configuring-issue-templates-for-your-repository
versions:
  fpt: '*'
  ghec: '*'
  ghes: '*'
topics:
  - Community
shortTitle: Configure
---

{% data reusables.repositories.default-issue-templates %}

## Creating issue templates

{% data reusables.repositories.navigate-to-repo %}
{% data reusables.repositories.sidebar-settings %}
1. In the "Features" section, under **Issues**, click **Set up templates**. You may need to enable **Issues** and refresh the page before you can see the button.

![Screenshot of the "Features" section of a repository's settings, with the "Issues" setting ticked and the green "Set up templates" button visible.](/assets/images/help/repository/set-up-issue-templates-button.png)
1. Use the **Add template** dropdown menu, and click on the type of template you'd like to create.

   ![Screenshot of the "Add template" dropdown menu expanded to show the standard "Bug report" and "Feature request" templates. In addition, the "Custom template" is listed.](/assets/images/help/repository/add-template-drop-down-menu.png)
1. To preview or edit the template before committing it to the repository, next to the template, click **Preview and edit**.
1. To edit the template, click {% octicon "pencil" aria-label="Edit template" %}, and type in the fields to edit their contents.

   ![Screenshot of the preview of an issue template. To the right of the template name, a pencil icon is outlined in dark orange.](/assets/images/help/repository/issue-template-edit-button.png)
1. To automatically set a default issue title, assign the issue to people with read access to the repository, or apply labels to issues raised from the template, use the fields under "Optional additional information." You can also add these details in the issue template with `title`, `labels`, or `assignees` in a YAML frontmatter format.
1. When you're finished editing and previewing your template, click **Propose changes** in the upper right corner of the page.
1. In the "Commit message" field, type a commit message describing your changes.
1. Below the commit message fields, select whether to commit your template directly to the default branch, or to create a new branch and open a pull request. For more information about pull requests, see "[AUTOTITLE](/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)."
1. Click **Commit changes**. Once these changes are merged into the default branch, the template will be available for contributors to use when they open new issues in the repository.

{% ifversion issue-forms %}

## Creating issue forms

{% data reusables.community.issue-forms-beta %}

With issue forms, you can create issue templates that have customizable web form fields. You can encourage contributors to include specific, structured information by using issue forms in your repository. Issue forms are written in YAML using the {% data variables.product.prodname_dotcom %} form schema. For more information, see "[AUTOTITLE](/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-githubs-form-schema)." {% data reusables.actions.learn-more-about-yaml %}

To use an issue form in your repository, you must create a new file and add it to the `.github/ISSUE_TEMPLATE` folder in your repository.

Here is an example of an issue form configuration file.

{% data reusables.community.issue-forms-sample %}

Here is the rendered version of the issue form.

![Screenshot of a rendered issue form, with a mix of text fields and dropdown menus.](/assets/images/help/repository/sample-issue-form.png)

1. Choose a repository where you want to create an issue form. You can use an existing repository that you have write access to, or you can create a new repository. For more information about creating a repository, see "[AUTOTITLE](/repositories/creating-and-managing-repositories/creating-a-new-repository)."
1. In your repository, create a file called `.github/ISSUE_TEMPLATE/FORM-NAME.yml`, replacing `FORM-NAME` with the name for your issue form. For more information about creating new files on GitHub, see "[AUTOTITLE](/repositories/working-with-files/managing-files/creating-new-files)."
1. In the body of the new file, type the contents of your issue form. For more information, see "[AUTOTITLE](/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)."
1. Commit your file to the default branch of your repository. For more information, see "[AUTOTITLE](/repositories/working-with-files/managing-files/creating-new-files)."

{% endif %}

## Configuring the template chooser

{% data reusables.repositories.issue-template-config %}

You can encourage contributors to use issue templates by setting `blank_issues_enabled` to `false`. If you set `blank_issues_enabled` to `true`, people will have the option to open a blank issue.

{% note %}

**Note:** If you used the legacy workflow to manually create an `issue_template.md` file in the `.github` folder and enable blank issues in your _config.yml_ file, the template in `issue_template.md` will be used when people choose to open a blank issue. If you disable blank issues, the template will never be used.

{% endnote %}

If you prefer to receive certain reports outside of {% data variables.product.product_name %}, you can direct people to external sites with `contact_links`.

Here is an example _config.yml_ file.

```yaml copy
blank_issues_enabled: false
contact_links:
  - name: {% data variables.product.prodname_gcf %}
    url: https://github.com/orgs/community/discussions
    about: Please ask and answer questions here.
  - name: {% data variables.product.prodname_dotcom %} Security Bug Bounty
    url: https://bounty.github.com/
    about: Please report security vulnerabilities here.
```

Your configuration file will customize the template chooser when the file is merged into the repository's default branch.

{% data reusables.repositories.navigate-to-repo %}
{% data reusables.files.add-file %}
1. In the file name field, type `.github/ISSUE_TEMPLATE/config.yml`.
1. In the body of the new file, type the contents of your configuration file.
{% data reusables.files.write_commit_message %}
{% data reusables.files.choose_commit_branch %}
{% data reusables.files.propose_new_file %}

## Changing the order of templates

You can set the order in which your issue templates will appear in the template chooser by making changes to the template filenames. The templates in `.github/ISSUE_TEMPLATE` are listed alphanumerically and grouped by filetype, with YAML files appearing before Markdown files.

To control the order of your templates, prefix the filenames with a number. For example: `1-bug.yml`, `2-feature-request.yml`, and `3-epic.yml`.

If you have 10 or more templates, alphanumeric ordering means that `11-bug.yml` will be positioned between `1-feature.yml` and `2-support.yml`. You can keep your intended ordering by prefixing your numeric filenames with an additional `0`. For example: `01-feature.yml`, `02-support.yml`, and `11-bug.yml`.

## Further reading

- "[AUTOTITLE](/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates)"
- "[AUTOTITLE](/communities/using-templates-to-encourage-useful-issues-and-pull-requests/manually-creating-a-single-issue-template-for-your-repository)"
