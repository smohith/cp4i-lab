# Cloud Pak for Integration Install Lab

## Familiarization with the Cloud Integration Platform[](#familiarization-with-the-cloud-integration-platform)

The Cloud Integration Platform (CIP) is built on top of IBM Cloud Private. This lab guide is meant to give you an overview of the capabilities of the platform, and get some hands on time with setting up your own environment for the purpose of creating your own demo environment.

This lab guide assumes some familiarity with IBM Cloud Private, Docker & Kubernetes, as well as Helm.

There are two primary components of CIP:

**IBM Cloud Private (ICP)** - An application platform for developing and managing on-premises, containerized applications. It is an integrated environment for managing containers that includes the container orchestrator Kubernetes, a private image repository, a management console, and monitoring frameworks. In this particular boot camp, ICP is deployed on Softlayer, using Skytap as the primary operational interface.

**Platform Navigator** – an application that is built to run on top of IBM Cloud Private that provides a primary user interface where instances of App Connect Enterprise (ACE), API Connect (APIC), Event Streams and MQ Advanced can be managed from a single place. It is automatically deployed with the CIP Bundle.

As a convention for these labs, a <span style="color:red">**red box**</span> will be used to identify a particular area, and when information is to be entered and/or an action is to be taken, it will be in **bold** characters. <span style="color:red">**Red arrows or lines**</span> may be used to indicate specific items that need to be noted as part of the lab

A number of sections will be labeled “Key Idea” or “Key Concept”. Be sure to review these sections when you come across them, as they will explain important items as it relates to the lab content.

## Lab Overview[](#lab-overview)

This lab is broken up into two key parts.

**Part One**: Create and Deploy your CIP Applications

**Part Two**: Deploy Integration Assets

Part one contains the hands on section of the lab where you will get acquainted with the Platform Navigator and be setting up the individual components of the platform.

Part two you will be deploying some basic assets to your newly created environment

## Part One: Create and Deploy your CIP Applications[](#part-one-create-and-deploy-your-cip-applications)

1.  You have been provided a pre-installed environment of the Cloud Integration Platform. It includes a vanilla ICP 3.1.1 install with the CIP Platform Navigator installed and all of the base container images that make up CIP.
2.  The environment you are using consists of 8 different nodes 7 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces. You can access this VM directly using the Skytap interface to use the X-Windows based components.
3.  **VERY IMPORTANT** It is very important you do not suspend your lab environment. We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted. Also it is a good practice that you shut down ICP before powering down your enviroment. A script has been included to handle that for you that will be explained in the coming sections. When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method. So far using power off hasn’t caused any problems that we are aware of.
4.  You may also able to SSH into your environment. You will find out the exact address to SSH to in your Skytap Environment window under `networking` and `published services`. The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node. The published service will look something like `services-uscentral.skytap.com:10000`. Your environment will have a different port on it. You can then SSH to to the machine from your local machine.
5.  Additionally, you can access the machine direct via the Skytap UI. This is functional, but can be cumbersome to work with as its not easy to copy and paste into and out of Skytap, especially for the non X-Windows based environments.
6.  Password-less SSH has also been enabled between the Master node and the other nodes in the environment. **note** a table with the environment configuration is provided below. Credentials for each machine are `skytap`/`A1rb0rn3`.

<table>

<thead>

<tr>

<th>VM</th>

<th>Hostname</th>

<th>IP Address</th>

<th># of CPU</th>

<th>RAM</th>

<th>Disk Space (LOCAL)</th>

<th>Shared Storage</th>

<th>Additional Notes</th>

</tr>

</thead>

<tbody>

<tr>

<td>Master</td>

<td>master/proxy</td>

<td>10.0.0.1</td>

<td>24</td>

<td>32 GB</td>

<td>350 GB</td>

<td>N/A</td>

<td> </td>

</tr>

<tr>

<td>Worker 1</td>

<td>worker-1</td>

<td>10.0.0.2</td>

<td>8</td>

<td>16 GB</td>

<td>370 GB</td>

<td>Monitor</td>

<td>Ceph Master</td>

</tr>

<tr>

<td>Worker 2</td>

<td>worker-2</td>

<td>10.0.0.3</td>

<td>8</td>

<td>16 GB</td>

<td>500 GB</td>

<td> </td>

<td> </td>

</tr>

<tr>

<td>Worker 3</td>

<td>worker-3</td>

<td>10.0.0.4</td>

<td>8</td>

<td>16 GB</td>

<td>500 GB</td>

<td> </td>

<td> </td>

</tr>

<tr>

<td>Worker 4</td>

<td>worker-4</td>

<td>10.0.0.5</td>

<td>8</td>

<td>16 GB</td>

<td>500 GB</td>

<td> </td>

<td> </td>

</tr>

<tr>

<td>Worker 5</td>

<td>worker-5</td>

<td>10.0.0.6</td>

<td>8</td>

<td>16 GB</td>

<td>320 GB</td>

<td>Ceph OSD1</td>

<td> </td>

</tr>

<tr>

<td>Worker 6</td>

<td>worker-6</td>

<td>10.0.0.7</td>

<td>8</td>

<td>16 GB</td>

<td>320 GB</td>

<td>Ceph OSD2</td>

<td> </td>

</tr>

<tr>

<td>Developer</td>

<td>developer</td>

<td>10.0.0.9</td>

<td> </td>

<td> </td>

<td> </td>

<td> </td>

<td> </td>

</tr>

</tbody>

</table>

1.  If it is not done already, power up your Environment. It could take about 5 minutes for all nodes to start up. The master node takes the longest to come up, so if you can see the login prompt from the Skytap UI on the master node, then you are good to go.
2.  It can take up to 30 minutes for the ICP Services to come up completely. You can tell if the start of ICP is complete by checking using one of these two methods:
    *   Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows. Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser. Navigate to the main ICP UI by going to `https://10.0.0.1:8443`. The credentials again are `admin/admin`. If you can log in and see the main ICP Dashboard, you are good to go.
    *   Alternatively, click on the Master node. Open a terminal. Execute an `su -` from the command line and when prompted for a password type in `Passw0rd!`. Execute a `cloudctl login` from the command line. If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
3.  The best place to do your Kubernetes CLI work is from the Master node. Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`.
4.  Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform. **note** at the time of this lab, the ability to create and manage Aspera instances has not yet be added to the Platform Navigator, and will be added at a later date.
5.  You can access the Platform Navigator using the browser on the developer machine. The URL for the navigator was set up in this environment as: `https://mycluster.icp/integration`. You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
6.  The Platform Navigator is designed for you to easily keep track of your integration toolset. Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running. You can also add new instances using the Platform Navigator.
7.  Each application instance requires its own namespace and pull secret for install. To support this lab, namespaces and pull secrets have been provided for you already, with the exception of Event Streams (ES). We’ll talk more about ES later in the lab.

### Key Concepts - Troubleshooting / Recovery[](#key-concepts---troubleshooting--recovery)

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on. If you post queries in Slack about trouble with the lab, we will be asking you to execute a series of these, depending upon what you were doing, and what error occured. A list of useful commands are provided below:

*   `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space. Here you will see if any pods are up, down, errored or otherwise in transition.
*   `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod. You can use the `describe` command for other objects.
*   `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well. You can also tail the logs by using the `-f` switch at the end.

If you happen to mess up an install of the components, its not difficult to recover. You can use the ICP UI to remove the release or use the CLI by issuing `helm delete --purge <helm-release-name> --tls`. Be careful using this though, as this process is not reversible.

Logging is also done inside of ICP using the ELK stack. You can access the logging inside of ICP and use Elastic Search commands to drill into things. For example, if you were to bring up Kibana and enter in a search like `kubernetes.namespace:<your namespace>`. Using this along with the `kubectl` commands gives you a lot of power to dig into root cause.

There are plenty other commands to use, but these are by far the most common the author of these labs has used while setting these up :)

## App Connect Enterprise[](#app-connect-enterprise)

1.  Before we create a new Application Integration instance, you will need to delete the existing one.
2.  There a couple of ways to remove an installation via the UI or the CLI. Using the ICP UI (`https://10.0.0.1:8443`) go to the top left hamburger config menu select `Workloads` -> `Helm Releases`. Sort by namespace by clicking on the `NAMESPACE` header. You will see two deployments in the `ace` namespace, namely `inventory` and `ace`. For each of them in turn find the `action` icon on the far right (looks like 3 dots). Click that and then select `Delete`. ICP will remove all of the Pods that are part of this install.
3.  Before you can deploy a new Integration Server you need to re-sync your helm repository in ICP by going to the main ICP UI (https://10.0.0.1:8443) then from the top-left hamburger menu `Manage` > `Helm Repositories`. Then Click on the `Sync Repositories`.
4.  Let’s start by creating an App Connect Enterprise Integration Server. From the Platform Navigator, locate the middle panel for `Application Integration` and click on the `add new instance` link.
5.  A pop up window will give you some important information about pre-requisites for creating the instance. Click `continue`.
6.  This will take you to the chart configuration. If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of App Connect Enteprise. Click the `Configure` button to configure your chart. Other than the items listed below, we will be accepting the default values in the chart.
7.  Here you can fill in the information for install of your chart. Give your helm release a `name`. Call the release `ace-icip`.
8.  To the right of the `name` field, Select the `ace` namespace from the `Target Namespace` dropdown.
9.  Tick checkbox under `license`
10.  Expand the `All Parameters` twisty
11.  Uncheck the box entitled `Production usage`
12.  Set the `Replica count` to the value of `1`
13.  Find the `Image Pull Secret` line. Set that to `sa-ace` (This is pre-defined for you, normally, you would need to create this).
14.  For the `Hostname of the ingress proxy to be configured` section. Change the default value provided to `ace.10.0.0.1.nip.io`.
15.  **IMPORTANT** - Find the `Enable Persistant Storage` Section. This box must be unticked (disabled). In order to support persistant storage, Gluster storage must be used (Ceph is not supported with ACE). Implementing/Using Gluster Storage with CIP beyond the scope of this lab.
16.  Similarly uncheck the box for `Use dynamic provisioning`
17.  Leave the other default values in the chart. Scroll down and Click `Install` button on lower right portion of the screen.
18.  Once you see the `Installation Started` pop up. You can validate the progress via the `Helm Releases` in the ICP Portal or via the command line.
19.  Return to your command line and open up a SSH session to your master node on your laptop or desktop. **Note** Each environment has a Skytap Published Service set up for every node on port 22 so you can SSH to the nodes from your machine. If you do a `kubectl get pods -n ace` you should see your pod in a running state.
20.  Returning to the Platform Navigator, you should see your new ACE instance there. Click on it to bring up the ACE Enterprise UI. **Note** if you encounter issues with the self signed certs on the browser. Click on the link where it says `open ace`. This will open a new browser window and should allow you to access the UI properly.
21.  The ACE Dashboard can alternatively be accessed via a web browser and entering the URL for the ACE Dashboard Portal manually. Run the following commands to get the dashboard URL via SSH:

`export ACE_DASHBOARD_URL=$(kubectl get ingress ace-icip-ibm-ace-dashboard-icip-prod-hostname -o jsonpath="{.spec.rules[0].host}{.spec.rules[0].http.paths[0].path}")`

then `echo "Open your web browser to https://${ACE_DASHBOARD_URL}"`

your output will look something like: `Open your web browser to https://ace.10.0.0.1.nip.io/ace-ace-icip`

_Hint_ If the first command fails, you might not be in the `ace` namespace. Quickest way to set your context is just to relogin using `cloudctl login` and then set your namespace to `ace`.

If you can see the App Connect Enterprise portal, then you are are done with this step. We’ll load in a .bar file later.

## MQ[](#mq)

1.  For MQ we will not be using the default chart that is linked to the Platform Navigator. What comes with the CIP is the production version of the chart that requires the installer to configure everything, which is cumbersome if you are just looking to spin up an environment quickly. Fortunately, ICP comes with a chart that is designed specifically for developer types that comes with a pre-configured queue manager, queues, server channels and the like.
2.  Before continuing you will need to delete the existing MQ instance deployed on the platform. Just like in the previous section, using the ICP UI (`https://10.0.0.1:8443`) go to the top left hamburger config menu select `Workloads` -> `Helm Releases`. Search for existing MQ deployments by using the Search widget and the `mq` keyword. Find the `action` icon on the far right (looks like 3 dots) of the returned instance. Click that and then select `Delete`. If you get an `ESOCKETTIMEDOUT` error you can ignore that. Verify that the deployment is gone by refreshing your browser.
3.  Using the top menu, right hand side, click on `Catalog`
4.  Using the context based search filter type in `mq`into the filter bar. Select the `ibm-mqadvanced-server-dev` chart. You can scroll down a bit to peruse the pre-requisites.
5.  This will take you to the chart configuration. If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of MQ. Click the `Configure` button to configure your chart. Other than the items listed below, we will be accepting the default values in the chart.
6.  Here you can fill in the information for install of your chart. Give your helm release a `name`. Call the release `mq-icip`.
7.  To the right of the `name` field, Select the `mq` namespace from the `Target Namespace` dropdown.
8.  Tick checkbox under `license`
9.  Expand the `All Parameters` twisty
10.  Under the `Image Pull Secret` section enter in `sa-mq`.
11.  Under `Persistence` untick both `Enable Persistence` and `Use Dynamic Provisioning`. _Note_ failure to untick this will cause your install to fail.
12.  In the `Service Type` section, select `Node Port`.
13.  Scroll down to the `Queue Manager` settings and enter `admin` for both the `Admin` and `App` passwords.
14.  You can leave the other settings as defaults.
15.  Click `Install` to start the process.
16.  You can monitor the installation from Helm Releases in the ICP UI or via the command line using `kubectl get pods -n mq`. Once you see your pod up and running, you can navigate to the UI via `Workload` ->`Helm releases`. Find your MQ instance and click on it.
17.  Scroll down a bit and Locate the `Service` section. Click on the URL that is called `mq-icip-ibm-mq`
18.  Under `Service Details` locate the URL called `console-https`. This will start up the MQ Console. Login with the credentials of `admin/admin`
19.  You can now review your pre-configured MQ instance.

## Event Streams[](#event-streams)

1.  Before we create a new Event Streams instance, you will need to delete the existing Event Streams instance. However, this time we will also delete the namespace and included secrets just because we can :-)
2.  There a couple of ways to remove an installation. We’ll do this using in two parts, one by UI, one by CLI. Using the ICP UI (`https://10.0.0.1:8443`) go to the top left hamburger config menu select `Workloads` -> `Helm Releases`. Locate the `eventstreams` install. _Hint_ you can filter releases using the search bar at the top. Find the `action` icon on the far right (looks like 3 dots). Click that and then select `Delete`. ICP will remove all of the Pods that are part of this install.
3.  Next we will remove the old namespace. Lets do that via the command line. Connect to your master node via SSH. Login via `cloudctl login` if you have to. Select default namespace.
4.  Delete the namespace as follows `kubectl delete namespace eventstreams`. Deleting the namespace should be fairly quick.
5.  Now lets recreate the namespace. Do this via `kubectl create namespace eventstreams`
6.  Change context to this new namespace by issuing `kubectl config set-context mycluster-context --user=admin --namespace=eventstreams`
7.  re-create your pullsecret then by issuing: `kubectl create secret docker-registry sa-eventstreams --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=admin`
8.  Return back to the Developer Machine via Skytap. Bring up the Platform Navigator again. Locate the panel on the far right for `Messaging` and click on the `add new instance` link and then select `New Event Streams` instance.
9.  Similar to the other instances you have setup, you will have a pop up that gives you some advice on requirements for install. Click `continue`
10.  Like before Click `Configure` in the lower right hand corner. Here we will configure the chart for install. Other than the items listed below, we will be accepting the default values in the chart.
11.  Under `Helm Releases` call it `eventstreams-cip`
12.  Set the `Target Namespace` to `eventstreams`
13.  Tick the checkbox under `License`
14.  Moving down, set the `Image Pull Secret` to `sa-eventstreams`
15.  Click the `All Parameters` Twisty
16.  Under the Global Install Parameters heading, make sure the `Used in Production` box in unchecked.
17.  Modify the `Docker Image Registry` entry by removing the last trailing ‘/’ character. e.g. `mycluster.icp:8500/eventstreams`. This is a design feature of this initial release, and will be fixed later.
18.  Moving down, make sure all of `persistent storage` and `dynamic provisioning` checkboxes are unchecked.
19.  That is the last change. Click the `Install` button on the lower right.
20.  Check the progress of the install using the command line (via SSH) to the master node. `kubectl get pods -n eventstreams`. Use the `watch` prefix to monitor the pods so you don’t have to keep repeating the same command. If you see all the pods up properly then you are good to go. It should take only a few minutes to spin up all of the required pods.
21.  You can access the management interface by returning back to the Platform Navigator UI `https://mycluster.icp/integration` on the Developer Machine. You should see a link to your newly deployed Eventstreams instance. From there you can explore the various configuration options within event streams

## API Connect Install[](#api-connect-install)

1.  To install API Connect, there are a few items that are needed in order to properly configure your ICP environment for install. These are listed below
    *   Persistant Storage (Ceph is required, and has been set up for you already)
    *   7 FQDN’s for the various UIs/Portal of API Connect
2.  Below is a list of the FQDN’s you will be using for this configuration.

### API Connect Management[](#api-connect-management)

**Platform API Endpoint:** `mgmt.10.0.0.1.nip.io`

**Consumer API Endpoint:** `mgmt.10.0.0.1.nip.io`

**Cloud Admin UI Endpoint:** `mgmt.10.0.0.1.nip.io`

**API Manager UI Endpoint:** `mgmt.10.0.0.1.nip.io`

### API Connect Portal[](#api-connect-portal)

**Portal Director API Endpoint:** `pd.10.0.0.1.nip.io`

**Portal Web UI Endpoint:** `pw.10.0.0.1.nip.io`

### API Connect Analytics[](#api-connect-analytics)

**Analytics Ingestion API Endpoint:** `ai.10.0.0.1.nip.io`

**Analytics Client API Endpoint:** `ac.10.0.0.1.nip.io`

### API Connect Gateway[](#api-connect-gateway)

**Gateway Service Endpoint:** aka Management Endpoint `gws.10.0.0.1.nip.io`

**API Gateway Endpoint:** aka API Invocation Endpoint `apigw.10.0.0.1.nip.io`

1.  From the Master, as root issue the command `cloudctl login -u admin -p admin -n apic`
2.  Check the name of the APIC helm release using `helm ls --tls | grep api` (it should be something like `api5`)
3.  Delete the APIC helm release using `helm delete --purge name_of_apic_release_from_above --tls` (this command will take a little while to complete)
4.  Delete all stateful sets, persistent volume claims, and jobs related to the APIC instance using `kubectl delete sts,job,pvc --all -n apic`
5.  Verify all pods and jobs are gone from the namespace using `kubectl get pods -n apic`
6.  Return to the Developer Machine and bring up the Platform Navigator. On the left hand side of the screen locate the `API Lifecycle and Secure Access` section. At the bottom of this section, locate and click on the `Add new instance` link.
7.  A pop up will come up, giving you some information about setting up the instance. Click `Continue`.
8.  Click on the `Configuration` heading. Other than the items listed below, we will be accepting the default values in the chart.
9.  Set the `Helm Release` to `apic-cip`.
10.  Moving to the right, set the `Target Namespace` from the dropdown to `apic`
11.  Tick the checkbox under `License`
12.  Under the `Parameters` heading. Set the `Registry Secret` to `sa-apic`.
13.  Set `Storage Class` to `rbd-storage-class` This is your Ceph Storage class name. **NOTE** it is very important that this setting is correct, otherwise the setup of the shared storage volumes will fail. **Note** you can issue a `kubectl get pvc -n apic` once you kick off the install of your chart to see how the shared space is being used by APIC and if there are any problems e.g. if you see an entry in a `pending` state for an extended period of time.
14.  Set the `Helm TLS Secret` to `helm-tls-secret`
15.  Continuing on, under the `Management` heading. Here you need to fill in the four different management headings as per the chart in step 2 above.
16.  Continue with the same process for the `Portal`, `Analytics` and `Gateway` sections. Follow the chart above and enter in the values for each of the above values in the FQDN list in Step 2.
17.  When done, click the `All Parameters` twisty.
18.  Scroll up a bit and make sure the `Production Deployment` box is not checked
19.  Under the `Global` heading, select the `Mode` drop down, and set the value to `dev`.
20.  Scroll down a bit and located the `Cassandra` heading. Set that value of `Cluster Size` to the value of `1`.
21.  Scroll down some more to the `Gateway` section. Set the `Replica Count` to the value of `1`.
22.  That completes the chart configuration. Click the `Install` button to start the process. The Entire Process to install API Connect can take up to 30 minutes to spin up all of the pods. You can monitor things using the watch command e.g. `watch -n 10 kubectl get pods -n apic`.

## API Connect Configuration[](#api-connect-configuration)

1.  These next steps are the standard setup steps for intializing the APIC topology and configuring a provider organization.
2.  On the Developer Machine open a browser tab and access `https://mgmt.10.0.0.1.nip.io/admin`. _Note:_ if you get an HSTS error then clear your entire browser history (in particular _Site Preferences_) and restart the browser. Once the Admin Portal comes up, you can log in with `admin/7iron-hide`. You will be logged in and forced to change your password. Set it to whatever you like, but don’t forget it.
3.  You will now be placed into the Cloud Manager Portal. Click the link to `Configure Topology`
4.  Click on `Register Service`
5.  Select `Datapower Gateway v5 Compatible`
6.  Under Gateway details give the gateway a `Title` and then under `Management Endpoint` set that value to `https://gws.10.0.0.1.nip.io`
7.  Set the `API Invocation endpoint` to `https://apigw.10.0.0.1.nip.io`. Click `Save`
8.  Back on the Topology screen, click `Register Service` and then select `Analytics`.
9.  Set the Title to whatever you like and then change the `management endpoint` to `https://ac.10.0.0.1.nip.io`. Click `Save`.
10.  Before registering the Portal service you need to setup Notifications. From the taskbar on the left hand side select `Resources` then `Notifications` followed by `Create`. Give it a `Title` and type in `10.0.0.9` for the `Address` and `2525` for the `port` and leave the remaining fields blank.
11.  At the same time open a terminal, run `java -jar fakeSMTP-2.0.jar` from the ~/fakesmtp directory and click `Start server`. Back in the APIC console, click `Save`
12.  From the taskbar, select `Settings` -> `Notifications`
13.  Under Sender & Email Server click `Edit`.
14.  Enter in a `Name` and `Email` for your Admin user, this can be anything.
15.  Down below, select the email server you configured above and click `Save`
16.  From the Topology screen, click `Register Service` then select `Portal`. _Note_ this doesn’t create a portal yet - it just registers the service.
17.  Set the Title to whatever you like then change the `Management Endpoint` to `https://pd.10.0.0.1.nip.io` and the `Portal Website URL` to `https://pw.10.0.0.1.nip.io`. Click `Save`
18.  You can then associate the analytics service to the gateway service by clicking on the `Associate Analytics` link.
19.  From the top left menu - go to `Provider Organizations`.
20.  Click `Add` -> `Invite Organization Owner`. Enter in a email address, use something different here.
21.  Once you see the activation link, click on it. Copy the link and paste it to a new browser tab window on the developer machine.
22.  No need to fill out any of the information that pops up, API Connect will grab your credentials from ICP, so going forward the API Manager UI will use SSO facilitated via ICP.
23.  You will automatically be logged into the API Manager. From here you can continue with the next section.

## Part Two: Deploy some Integration Assets[](#part-two-deploy-some-integration-assets)

1.  Load up the management page for your ACE install. You can access it from the Platform Navigator by clicking on your `ace` install or just direct your browser to `https://mycluster.icp/ace-ace-icip/`
2.  Click on the `+Server` button and then on `Add a BAR file`. On the local file system, navigate to the `/home/student/techguides` directory and find the lone bar file which should be `RESTRequest_Everything.bar`. A pop up with a content URL will come up. Copy this to your clipboard. You will need it for the next step.
3.  You will now deploy your bar file using the ACE chart, so you will be taken to the chart configuration screen in ICP. Click the `Configure` button or the `Configuration` tab at the top to continue
4.  Set the `Helm Release` to `lab`
5.  Change the `Target Namespace` to `ace`
6.  Tick the `License` checkbox
7.  Click on the `All Parameters` twisty to open it up
8.  Paste in the URL given to you above `Content Server URL`
9.  Untick the `Production Usage` checkbox
10.  Set the `Replica Count` to `1`
11.  Set the `Image Pull Secret` to `sa-ace`
12.  Untick the `Enable Persistence` and `Use Dynamic Provisioning` checkboxes.
13.  Under the `Service` settings, find the `NodePort IP` change this to `lab.10.0.0.1.nip.io`. This will be the endpoint for your service running on ACE.
14.  Scroll down to the `Integration Server` section and make sure all the names, passwords, and aliases fields are blank
15.  Leave the remaining settings as defaults and then click `Install` at the bottom. Your chart will now install. You can view the progress of the install via Helm Releases as prompted or use `kubectl`.
16.  Return back to your ACE Management UI. (Click the `done` on the window from the previous step). You should now see one service running on your integration server. Click on the 3 dot hamburger button on the `lab` icon. Click Open.
17.  You will now see 3 icons, one for the REST Request API, Application and Shared Library. Click on the hamburger icon for REST Request_API and then click open
18.  You will then see two URL’s that look something (but not exactly) like this:

    *   REST API Base URL `http://lab.10.0.0.1.nip.io:30098/restrequest_api/v1`
    *   OpenAPI document `http://lab.10.0.0.1.nip.io:30098/restrequest_api/v1/Tutorial_swagger.json`

Copy and paste the link for the OpenAPI document into a browser window and then save the json file on the local file system. We will use this inside of API Connect

1.  Open up your API Manager Window inside the developer machine - `https://mgmt.10.0.0.1.nip.io/manager`. The login should happen automatically as it will use your ICP credentials for login.
2.  Click on the `Manage Catalogs`
3.  Click on `Sandbox`
4.  Click on `Settings` from left menu
5.  Click on `Gateway Services`
6.  Click `edit` and choose `Gateway`.
7.  From the API Manager homescreen, use the menu on the left to navigate to `Develop`.
8.  On the APIs and Products screen click `add` -> `Api`
9.  Select the `From Existing API Service` radio button option. Click `Next`.
10.  Click the `browse` button to navigate to the `Tutorial_swagger.json` file on your file system you had saved previously.
11.  Click `Next`
12.  Review the settings then click `Next` twice (do not activate the API yet).
13.  Review the summary to ensure the API was created properly, and then click the `Edit API` button.
14.  Click the `Assemble` tab to bring up the Assembly editor.
15.  Click the Play button to start the Test tool.
16.  Click the blue `Activate API` button to deploy.
17.  Select the `get /UserNumber` operation.
18.  Scroll down to the `Parameters` section and find the `UserNumber` - replace what is there with the number `1`.
19.  Scroll down and click the `Invoke` button. The first time you invoke this, it will bring up a CORS warning. Copy and paste the link for the API into a new browser tab. It will return a 401 error for authorization, but this will load up the cert.
20.  Run the invoke again, and you should see the output properly.

### Conclusion[](#conclusion)

You have completed the initial CIP Labs by taking a base ICP install and then imported some basic integration assets and exposed that as an API running on the configuration. The key value is having the single platform that runs all of the capability that can be managed easily, user the power of Kubernetes.

**End of Lab**

</div>

* * *

<footer>

<div class="row">

<div class="col-lg-6 footer-left">©2019 IBM. All rights reserved.  
Site last generated: May 16, 2019  
</div>

<div class="col-lg-6 footer-right">

![IBM](cip_helm_install_files/ibm-logo-small.png)

</div>

</div>

</footer>

</div>

</div>

</div>
