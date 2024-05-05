# Web Console
Oberkorn has a builtin web console since version 0.3. The very first version just added some 'reading' capabilities, that is, you can view the status or configuration of your authrizators, but you can't do any changes to the controller nor the authorizators.d

### How do the web console work?
The web console has been designed to work as a standard web application, the front has been developed using ReactTS 18, and the API's are served from the controller and the authorizators from an [**express**](http://expressjs.com) framework.

#### Architecture
This picture show global web console architecture, including threee main components:

  - The controller, which is in charge of serving the frontapplicationion and the API.
  - A console authorizator (top-right), in charge of securing console access.
  - Ingress, in charge of allowing access to the console from outside the cluster.

![Web console architecture](/_media/webconsole/webconsole-architecture.png)


### Enabling the console
The first step to enable the console is realy simple: in the spec of the deployment of the controller you must add (or change if it already exists) an environment variable named '**OBKC_CONSOLE**', setting its value to '**true**'. See the sample bellow:

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

For the web console to be accesible from outside the cluster you can just use port forwarding, but it is a better idea to enable an ingress and a service in order to enable access to the controller. Please review [default web console sample](https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/controller-webconsole.yaml).

This sample YAML:
  - Creates a service to expose port 3882 of the controller.
  - Creates an ingress to enable access to the console ('/obk-console').
  - Creates an Oberkorn Authorizator to protect access to the console.

### Enabling APIs
Enabling the web console is done by configuring the controller, but having the ability to access an Oberkorn Authorizator API is something you must do by modifying Authorizators configuration.

The first step is easy. In the YAML you used to create the Authorizator you need to add a new parameter to the 'config' section, like the sample bellow. There is no doubt on what parameter we are talking about, it is named '**api**'.

```yaml
spec:
  config:
    replicas: 2
    api: true
    logLevel: 1
```

Parameter **api** set to 'true' will enable the API endpoint of the authorizator (defualt value is 'false').

You have nothing else to do in the authorizator. Now, when you access the web console you should see your Authorizator in the selector of the console.


### Accessing the web console
If everything is ok, the console should be available at http://your.dns.name:youPublicPort/obk-console, I mean, the external DNS name and the external port depend on your cluster and your ingress configuration.

When accessing the console for the first time, the controller will require administrator credentials. In this case, the sample web console YAML uses a Basic Auth (of type 'secret') with one only user: 'admin' (with password 'admin'). The sample web console YAML contains an Oberkorn Authorizator that protects the web console and the APIs using a classic Basic Auth (you can change this validator to use any other validator supported by Oberkorn, and integrate web console securization into your corporate administration securization policies, like using an AD, a KeyCloak or whatever you need).

If you enter the strong admin credentials correctly you'll see the console home page:

![Console homepage](/_media/webconsole/webconsole-welcome.png)

The homepage is easy to understand, it only contains a selector for selecting which authorizator do you want to work with. At least, you should see here one authorizator, which is in fact the authorizator that protects web console access. When you select an authorizator you enter the console main page, which is self-explaining.

![Authorizator Overview](/_media/webconsole/webconsole-overview.png)
