# Web Console
Oberkorn has a builtin web console since version 0.3. The very first version just add some 'reading' capabilities, that is, you can view the status or configuration, but you can't do any changes to the controller nor the authorizators.

### How the web console works?
The web console has been designed to work as a standard web application, the front has been developed using ReactTS 18, and the API's are served from the controller and the authorizators from an [**express**](http://expressjs.com) framework.

#### Architecture
WIP include a diagram here and add some details

### Enabling the console
The first step to enable the console is realy simple: in the spec of the deployment of the controller you must add (or change it it already exists) an environment vriable named '**OBKC_CONSOLE**', setting its value to '**true**'. See the sample bellow:

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

For the web console to be accesible from outside the cluster you can just use prot forwarding or enable an ingress and a service to publish access to the controller deployment. Please review [default web console sample](https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/controller-webconsole.yaml).

This sample web console:
  - Creates a service to expose port 3882 of the controller.
  - Creates an ingress to enable access to the console ('/obk-console') and a second path to access the API (see bellow).
  - Creates an Oberkorn Authorizator to protect access to the console.

### Enabling APIs
Enabling the web console is done by configuring the controller, but having the ability to access an Oberkorn Authroizator API is something you must do by modifying Authorizators configuration.

The first step is easy. In the YAML you used to create the Authroizator you need to add a new parameter to the 'config' section, like the sample bellow:

```yaml
spec:
  config:
    replicas: 2
    api: true
    logLevel: 1
```

Parameter **api** set to 'true' will enable the API endpoint in the authorizator (defualt value is 'false').

You have nothing else to do in the authorizator. Now, when accessing the web console you should see your Authorizator in the selector of the console.


### Accessing the console
If everything is ok, the console should be available at http://your.dns.name:youPort/obk-console, I mean, the external DNS name and the external port depend on your cluster and your ingress configuration.

When accessing the console for the first time, the controller will require administrator credentials. In this case, the sample web console YAML uses a Basic Auth (of type 'secret') with one only user: 'admin' (with password 'admin'). The sample web console YAML contains an Oberkorn Authorizator that protects the web console and the APIs using a classic Basic Auth (you can change this validator to use any other validator supported by Oberkorn, and integrate web console securization into your corporate administration securization policies, like using an AD, a KeyCloak or whatever you need).

If you enter the strong admin credentials correctly you'll see the console home page:

![Console homepage](/_media/webconsole/webconsole-welcome.png)

The homepage is easy to understand, it only contains a selector for selecting which authrizator do you want to work with. At least, you should see here one authorizator, which is in fact the authorizator that protects web console access. When you select an authorizator you enter the console main page, which is self-explaining.

![Authorizator Overview](/_media/webconsole/webconsole-overview.png)
