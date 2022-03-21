# Install MaPS-Tool

# pre-requirements
## PAT
Before using maps-tools you need to create a PAT. \
If you need help creating a PAT click [here.](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry)

## Docker Deskop
Before using maps-tools you'll also need to install Docker Desktop.
If you need help installing Docker Desktop click [here.](https://docs.docker.com/desktop/mac/install/)


## install the MaPS-Tool
```bash
bash <(curl -s https://raw.githubusercontent.com/spring-media/maps-tools-install/main/maps-tools)
```


# How To Use

## Login to the Container

```bash
maps-tools
```

## Exit the Container

```
exit
```

## Login using MFA (EXPERIMENTAL)
```bash
maps-tools -l
```

## Options

```bash
-e: Environment example (AWS_PROFILE NAME: as-spring-mediasites-stage|as-spring-mediasites-prod)
-v: DO NOT USE a Docker volume mount for local .aws, .ssh, .helm and .kube configs inside the container.
-h: Print this help text
-i: Ask for a local Docker image to use
-d: create daemon container that runs in the background, so you can use docker exec to connect to
-a: attach to daemon container that previously created with -d option
-l: (EXPERIMENTAL) option to perform AWS login with AZURE AD, default role is Developer,
    you can also specify default assume role: Admin, Developer, OnlyViewer after parameter, e.g. -l Admin
    NOT supported on arm64
-o: add aditional cmdline parameter to docker run/create command
    IMPORTANT: use double quotes! you can use this option more than one time to add as many parameters you want
    e.g. -o "-p 2222:22" -o "-e DISPLAY=host.docker.internal:0"
-u: update maps-tools and docker image (force pull)
```

# Installes aliases and bash functions

| command | description |
|---|---|
| `awsctx` | switch between various AWS profiles, `awsctx -l <profilename>` also performs a login, `awsctx -l <profilename> Admin` assumes role Admin instead default Developer role |
| `eks-update-kubeconfig` | add all existing EKS cluster in `$AWS_ACCOUNT` into your kubeconfig (`$HOME/.kube/config-$AWS_ACCOUNT`) |
| `eks-token` | for your current Kubernetes context display the token you need for accessing the Kubernetes dashboard |
| `helm-toggle` | helm client and Tiller (server side) versions always must match. Simply toggle between different Helm versions installed by brew. |
| `kubectl login` | performs an aws login for your current Kubernetes cluster context. So when working with multiple clusters in different accounts you do not have to remember where to login. |
| `k` | `kubectl` |
| `kctx` | `kubectx` |
| `kns` | `kubens` |
| `kgpo` | `kubectl get pods`, `kgsvc` for `kubectl get service`,  [full list here](https://raw.githubusercontent.com/ahmetb/kubectl-aliases/master/.kubectl_aliases)|
| `stern` | get logs from more than one pod, e.g. `stern pod-name` to get logs of all pods that name starts with pod-name |
| `kds` | get base64 decrypted kubernetes secrets |

You can finde more here: https://maps.as-infra.de/user-manual/cheatsheet/

# FAQ

## com.docker.backend Process uses 100% CPU time

### Problem

Process com.docker.backend uses 100% of CPU time. This mostly happend when you try to mount your $HOME directory.
Also see docker issue https://github.com/docker/for-mac/issues/5164

### Solution

There is a workaround: in docker desktop settings deactivate the option under general "Use gRPC FUSE for file sharing" https://github.com/docker/for-mac/issues/5164#issuecomment-901072179

## You cannot login with Azure AD option `maps-tools -l`

### Problem

You propably get an error message like that:
```
[Error: EACCES: permission denied, open 'aws-azure-login-unrecognized-state.png'] {
  errno: -13,
  code: 'EACCES',
  syscall: 'open',
  path: 'aws-azure-login-unrecognized-state.png'
}

The source profile "takeoff" must have credentials.
```

This mainly happens if you try to login the first time with the setup or if something happens in the AD or your account, e.g. if you change your AS-IT AD password (thanks god this isn't anymore ones in a quarter ;-)
### Solution

This error comes from Azure AD Authorization opens web dialogs that aws-azure-login application do not handle correctly. Most times you must choose your account or enter a basic auth to go further in the Azure AD login prozess.

Solution is based on that medium article: https://medium.com/@mreichelt/how-to-show-x11-windows-within-docker-on-mac-50759f4b65cb

You just need a valid X11 display to display the dialog boxes Azure AD is sending.

* Install the latest [XQuartz](https://www.xquartz.org/) X11 server and run it
* Activate the option ‘Allow connections from network clients’ in XQuartz settings
* Quit & restart XQuartz (to activate the setting)
* Run in terminal following commands
```
xhost + 127.0.0.1
```
```
maps-tools -l -o "-e DISPLAY=host.docker.internal:0"
```
ignore possible login errors that may ocure
```
aws-azure-login -p takeoff --no-sandbox -m gui
```
Now aws-azure-login can start chromium browser windows and you can do the login proccess as usal.

#### TIP: If it's possible don't do this in the campus wlan or VPN connection. It works, but maybe you cannot set the option to ask less times for login credentials.

## Login error `The requested DurationSeconds exceeds the MaxSessionDuration set for this role.`

### Problem

MaxSessionDuration error message when trying `aws-adfs` login

`botocore.exceptions.ClientError: An error occurred (ValidationError) when calling the AssumeRoleWithSAML operation: The requested DurationSeconds exceeds the MaxSessionDuration set for this role.`

### Solution

The session duration can be configured for every single role through https://console.aws.amazon.com/iam/home?region=eu-central-1#/roles by adjusting the Maximum CLI/API session duration.

## `aws eks describe-cluster` expecting wrong parameters

### Problem

`aws eks describe-cluster --name mycluster` fails saying `error: argument --cluster-name` is required, although `--name` is correct.

### Solution

`rm -rf ~/.aws/models/eks` - see also [here](https://docs.aws.amazon.com/de_de/eks/latest/userguide/create-cluster.html), search for "argument".

## `pip install` fails with `SysCallError(-1, 'Unexpected EOF'` messages

### Problem

Tools like `aws-adfs` or `awslimitchecker` are deployed using `pip` (the Python package installer) but the installation or update suddenly fails with the following message:

`Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError("bad handshake: SysCallError(-1, 'Unexpected EOF')",),)': /simple/aws-adfs/`

### Solution

Open the `Programs/AS Tools/Self Service` app and launch `Fix my Mac`. This app is repairing broken links and permissions which might happen during OS Updates.

## `terraform` 0.12 issues

### Problem

The `terraform` and `terragrunt` clients previously installed with `brew` are autoupdating occasionally. As the [infra-k8s-template](https://github.com/spring-media/infra-k8s-template) is `terraform` 0.11 code, newer `terraform/terragrunt` versions are failing.

### Solution

Use [tfswitch](https://github.com/warrensbox/terraform-switcher) (for `terraform-switch`) and [tgswitch](https://github.com/warrensbox/tgswitch) (for `terragrunt-switch`) to individually select the right `terraform`/`terragrunt` version.

Both cmdline clients are being preinstalled with this setup now.

As `tfswitch`/`tgswitch` can cause conflicts with vanilla `terraform`/`terragrunt` clients they should be uninstalled.

```bash
brew uninstall terragrunt
brew uninstall terraform
brew uninstall terraform@0.11
```^^

### Problem

If you're having trouble with your login or you're having trouble executing `terraform`/`terragrunt` commands.

### Solution

If this happens to you all you gotta do is remove your ~/.aws/config.  
`rm -rf ~/.aws/config ~/.aws/config_azure ~/.aws/config_adfs`  
After this just pull the newest maps-tools image.  
`maps-tools -u`  
And run maps-tools without any parameter.   
`maps-tools`  
This should fix the problem.  

