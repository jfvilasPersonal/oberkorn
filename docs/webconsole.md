# Web Console
Oberkorn has a builtin web console since version 0.3. The very first version just added some 'reading' capabilities, that is, you can view the status or configuration of your authrizators, the validators, the rulesets..., but you can't do any changes to the controller nor the authorizators.d

### How do the web console work?
The web console has been designed to work as a standard web application, the front has been developed using ReactTS 18, and the API's are served from the controller and the authorizators using an [**express**](http://expressjs.com) framework.

#### Architecture
This picture shows global web console architecture, including three main components:

  - The controller, which is in charge of serving the frontapplicationion and the API.
  - A console authorizator (top-right), in charge of securing console access.
  - Ingress, in charge of allowing access to the console from outside the cluster.

![Web console architecture](/_media/webconsole/webconsole-architecture.png)


### Enabling the console
The first step to enable the console is realy simple: in the spec of the deployment of the controller you must add (or change if it already exists) an environment variable named '**OBKC_CONSOLE**', setting its value to '**true**'. See the sample below:

```yaml
spec:
  serviceAccount: obk-controller-sa
  containers:
    - name: obk-controller
      image: obk-controller
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 3882
      env:
        - name: OBKC_LOG_LEVEL
          value: '2'
        - name: OBKC_CONSOLE
          value: 'true'
```

For the web console to be accesible from outside the cluster you can just use port forwarding, but it is a better idea to enable an ingress and a service in order to enable access to the controller. Please review [default web console sample](https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/installation/controller-webconsole.yaml).

This sample YAML:
  - Creates a service to expose port 3882 of the controller.
  - Creates an ingress to enable access to the console ('/obk-console').
  - Creates an Oberkorn Authorizator to protect access to the console.

### Enabling APIs
Enabling the web console is done by configuring the controller, but having the ability to access an Oberkorn Authorizator API is something you must do by modifying Authorizators configuration and/or the controller configuration.

The first step is easy. In the YAML you used to create the Authorizator you need to add a new parameter to the 'config' section, like the sample below. There is no doubt on what parameter we are talking about, it is named '**api**'.

```yaml
spec:
  config:
    replicas: 2
    api: true
    logLevel: 1
```

Parameter **api** set to 'true' will enable the API endpoint of the authorizator (default value is 'false').

You have nothing else to do in the authorizators.

By default, the authroizators API is not accesible from outside the cluster, so, for the console to run properly you need to enable a **proxy** feature included in the controller. This feature:

  - Exposes and API endpoint at the controller, the one the web console will use for obtaining data and sending commands to the authorizators.
  - The controller API exposes just 2 endpoints:
    - One endpoint that allows obtaining then list of authorizators that have an API endpoint enabled, that is, the ones you can manage using the web console.
    - A second proxy endpoint is enabled which is responsible of redirecting web console requests to authroizator endpoints using internal kubernetes cluster network. This way, authorizators endpoint do not need to be publicly accesible.

To enable the controller API you just need to add a new environment variable, like the sample below:

```yaml
spec:
  serviceAccount: obk-controller-sa
  containers:
    - name: obk-controller
      image: obk-controller
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 3882
      env:
        - name: OBKC_LOG_LEVEL
          value: '2'
        - name: OBKC_CONSOLE
          value: 'true'
        - name: OBKC_API
          value: 'true'
```

Now, when you access the web console you should see your Authorizators (the ones that have API enabled) in the selector of the Welcome page of the console.


### Accessing the web console
If everything is ok, the console should be available at http://your.dns.name:youPublicPort/obk-console, I mean, the external DNS name and the external port depend on your cluster and your ingress configuration.

When accessing the console for the first time, the controller will require administrator credentials. In this case, the sample web console YAML uses a Basic Auth (of type 'secret') with one only user: 'admin' (with password 'admin'). The sample web console YAML contains an Oberkorn Authorizator that protects the web console and the APIs using a classic Basic Auth (you can change this validator to use any other validator supported by Oberkorn, and integrate web console securization into your corporate administration securization policies, like using an AD, a KeyCloak or whatever you need).

If you enter the strong admin credentials correctly you'll see the console home page:

![Console homepage](/_media/webconsole/webconsole-welcome.png)

The homepage is easy to understand, it only contains a selector for selecting which authorizator do you want to work with. At least, you should see here one authorizator, which is in fact the authorizator that protects web console access. When you select an authorizator you enter the console main page, which is self-explaining.

![Authorizator Overview](/_media/webconsole/webconsole-overview.png)


### Using te web console
These are the main sections of the web console menu:

  - **Overview**: you can review general authoizator configuration and some statistical info about authorizator and validator usage
  - **Validators**: information about the different validators included in the authorizator.
  - **Rulesets**: information about all rulesets defined in the authorizator.
  - **Actions**: you can perform several diagnotics and tracing activities here.

#### Overview
General information about the authroizator (this does not include specific info on the validators).

#### Validators
List of validators and their current configuration.

#### Rulesets
List of rulesets and their current configuration.

![Rulets info](/_media/webconsole/webconsole-rulesets.png)

#### Actions
The actions main menu option leads you to a diagnostics section. First thing to do here is selecting the validator you want to work with. Once selected the tabbed section will enable, allowing you to:

  - Diagnose.
  - Trace (obtaining real-time information on what a user is doing).

##### Invalidate
You can use the web console to invalidate a token by specifying some invalidation criteria. You can set:

  - **User name**, invalidate a token when the token has been issued for a specific user name.
  - **Claim name**, invalidate a token when the token contains a specific claim.
  - **Audience**, invalidate a token when the token has been issued for a specific audience.
  - **Issuer**, invalidate a token when the token has been issued by a specific issuer.

You just need to specify the criteria and click set. Next time a token matching the criteria wold be presented by a client (a user or any application accessing a protected web resource) the access will be rejected, and a 401 will be sent to the requestor.

![Invalidation](/_media/webconsole/webconsole-invalidate.png)


##### Diagnose
Tracing will show real-tiem events related to a specific user you want to trace. Just type-in the user name to trace and click start. As soon as the users performs any secutity-related action, the action descrition will be shown here. You can view here actions like 'Sing in ok', 'Invalid password', 'Vlaid token', etc...

![Tracing](/_media/webconsole/webconsole-diagnostics.png)

You can stop the trace any time.

Please keep in mind all the authorizator replicas will send information to the console about the users you want to trace. This is done by the controller sending tracing configuration to all the replicas, and furthermore, the web console querying the authorizator replicas repeatedly (4 times in a second).
