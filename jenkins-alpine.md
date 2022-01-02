# Setup Jenkins on Linux VM

## 1. With Debian 11.2

Some inspiration: [How To Automate Jenkins Setup with Docker and Jenkins Configuration as Code](https://www.digitalocean.com/community/tutorials/how-to-automate-jenkins-setup-with-docker-and-jenkins-configuration-as-code)

### 1.1. Debian install

* Unselect Gnome and Desktop environment
* Select SSH server

### 1.2. Debian post-install

* Login as regular user
* `su -` then `apt update`
* `apt install -y curl sudo`

### 1.3. Jenkins install

From official [Debian/Ubuntu installation instructions](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu) (just added `-y` to `apt-get install`)

    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install -y openjdk-11-jdk
    sudo apt-get install -y jenkins

### 1.4. Start Jenkins

    sudo systemctl start jenkins

### 1.5. Configure Jenkins

The default behavior is to protect the Jenkins instance. A password is generated and stored onto the filesystem, and must be pasted in the Jenkins web UI.

To override this behavior, add the following line to `/etc/init.d/jenkins`, under the `JAVA=...` line:

    JAVA_ARGS="-Djenkins.install.runSetupWizard=false"

Note: alternative with [useSecurity](https://stackoverflow.com/questions/15227305/what-is-the-default-jenkins-password) property?

## 2. With Alpine 3.15

> Spoiler alert: it **failed when I ran `java -jar jenkins.war`**.

### 2.1. Create VM

In VirtualBox:

* Create VM with 3G of RAM
* Insert Alpine VM-optimized ISO image
* Configure network interfaces:

    * Keep eth0 as NAT, add port forwarding from 2222 (host) to 22 (guest)
    * Add eth1 as internal network

### 2.2. VM first boot

Once Alpine is booted, run `setup-alpine`. Let it configure `eth0` with `dhcp`, then enter `done` when asked to configure `eth1` and `n` for manual network config (we'll do that later).

The last step is to select a target disk: choose `sda` then `sys`.

Shut down the VM, then **go to VM storage settings and remove Alpine ISO**.

### 2.3. VM post-install config

Edit `/etc/apk/repositories`, uncomment the second line (ending with `/community`).

Run `apk update`.

### 2.4. Install Java

Run:

    apk add openjdk11

### 2.5. Run Jenkins

Run `java -jar jenkins.war`

    localhost:~$ java -jar jenkins.war 
    Running from: /home/johndoe/jenkins.war
    webroot: $user.home/.jenkins
    2022-01-02 07:51:44.928+0000 [id=1]	INFO	org.eclipse.jetty.util.log.Log#initialized: Logging initialized @696ms to org.eclipse.jetty.util.log.JavaUtilLog
    2022-01-02 07:51:45.078+0000 [id=1]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
    2022-01-02 07:51:46.516+0000 [id=1]	WARNING	o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
    2022-01-02 07:51:46.632+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: jetty-9.4.43.v20210629; built: 2021-06-30T11:07:22.254Z; git: 526006ecfa3af7f1a27ef3a288e2bef7ea9dd7e8; jvm 11.0.13+8-alpine-r0
    2022-01-02 07:51:47.084+0000 [id=1]	INFO	o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
    2022-01-02 07:51:47.171+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: DefaultSessionIdManager workerName=node0
    2022-01-02 07:51:47.171+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: No SessionScavenger set, using defaults
    2022-01-02 07:51:47.172+0000 [id=1]	INFO	o.e.j.server.session.HouseKeeper#startScavenging: node0 Scavenging every 660000ms
    2022-01-02 07:51:47.970+0000 [id=1]	INFO	hudson.WebAppMain#contextInitialized: Jenkins home directory: /home/johndoe/.jenkins found at: $user.home/.jenkins
    2022-01-02 07:51:48.126+0000 [id=1]	SEVERE	hudson.util.BootFailure#publish: Failed to initialize Jenkins
    java.lang.NullPointerException
      at java.desktop/sun.awt.FontConfiguration.getVersion(FontConfiguration.java:1262)
      at java.desktop/sun.awt.FontConfiguration.readFontConfigFile(FontConfiguration.java:225)
      at java.desktop/sun.awt.FontConfiguration.init(FontConfiguration.java:107)
      at java.desktop/sun.awt.X11FontManager.createFontConfiguration(X11FontManager.java:719)
      at java.desktop/sun.font.SunFontManager$2.run(SunFontManager.java:379)
      at java.base/java.security.AccessController.doPrivileged(Native Method)
      at java.desktop/sun.font.SunFontManager.<init>(SunFontManager.java:324)
      at java.desktop/sun.awt.FcFontManager.<init>(FcFontManager.java:35)
      at java.desktop/sun.awt.X11FontManager.<init>(X11FontManager.java:56)
    Caused: java.lang.reflect.InvocationTargetException
      at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
      at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
      at java.base/jdk.internal.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
      at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:490)
      at java.desktop/sun.font.FontManagerFactory$1.run(FontManagerFactory.java:84)
    Caused: java.lang.InternalError
      at java.desktop/sun.font.FontManagerFactory$1.run(FontManagerFactory.java:86)
      at java.base/java.security.AccessController.doPrivileged(Native Method)
      at java.desktop/sun.font.FontManagerFactory.getInstance(FontManagerFactory.java:74)
      at java.desktop/java.awt.Font.getFont2D(Font.java:497)
      at java.desktop/java.awt.Font.getFamily(Font.java:1410)
      at java.desktop/java.awt.Font.getFamily_NoClientCode(Font.java:1384)
      at java.desktop/java.awt.Font.getFamily(Font.java:1376)
      at java.desktop/java.awt.Font.toString(Font.java:1869)
      at hudson.util.ChartUtil.<clinit>(ChartUtil.java:269)
      at hudson.WebAppMain.contextInitialized(WebAppMain.java:217)
    Caused: hudson.util.AWTProblem
      at hudson.WebAppMain.contextInitialized(WebAppMain.java:218)
      at org.eclipse.jetty.server.handler.ContextHandler.callContextInitialized(ContextHandler.java:1067)
      at org.eclipse.jetty.servlet.ServletContextHandler.callContextInitialized(ServletContextHandler.java:572)
      at org.eclipse.jetty.server.handler.ContextHandler.contextInitialized(ContextHandler.java:996)
      at org.eclipse.jetty.servlet.ServletHandler.initialize(ServletHandler.java:746)
      at org.eclipse.jetty.servlet.ServletContextHandler.startContext(ServletContextHandler.java:379)
      at org.eclipse.jetty.webapp.WebAppContext.startWebapp(WebAppContext.java:1449)
      at org.eclipse.jetty.webapp.WebAppContext.startContext(WebAppContext.java:1414)
      at org.eclipse.jetty.server.handler.ContextHandler.doStart(ContextHandler.java:910)
      at org.eclipse.jetty.servlet.ServletContextHandler.doStart(ServletContextHandler.java:288)
      at org.eclipse.jetty.webapp.WebAppContext.doStart(WebAppContext.java:524)
      at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:73)
      at org.eclipse.jetty.util.component.ContainerLifeCycle.start(ContainerLifeCycle.java:169)
      at org.eclipse.jetty.server.Server.start(Server.java:423)
      at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart(ContainerLifeCycle.java:110)
      at org.eclipse.jetty.server.handler.AbstractHandler.doStart(AbstractHandler.java:97)
      at org.eclipse.jetty.server.Server.doStart(Server.java:387)
      at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:73)
      at winstone.Launcher.<init>(Launcher.java:192)
      at winstone.Launcher.main(Launcher.java:369)
      at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
      at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
      at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
      at java.base/java.lang.reflect.Method.invoke(Method.java:566)
      at Main._main(Main.java:304)
      at Main.main(Main.java:108)
    WARNING: An illegal reflective access operation has occurred
    WARNING: Illegal reflective access by org.codehaus.groovy.reflection.CachedClass (file:/home/johndoe/.jenkins/war/WEB-INF/lib/groovy-all-2.4.21.jar) to method java.lang.Object.finalize()
    WARNING: Please consider reporting this to the maintainers of org.codehaus.groovy.reflection.CachedClass
    WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
    WARNING: All illegal access operations will be denied in a future release
    2022-01-02 07:51:48.572+0000 [id=1]	INFO	o.e.j.s.handler.ContextHandler#doStart: Started w.@7ff8a9dc{Jenkins v2.327,/,file:///home/johndoe/.jenkins/war/,AVAILABLE}{/home/johndoe/.jenkins/war}
    2022-01-02 07:51:48.615+0000 [id=1]	INFO	o.e.j.server.AbstractConnector#doStart: Started ServerConnector@75b25825{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
    2022-01-02 07:51:48.616+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: Started @4385ms
    2022-01-02 07:51:48.618+0000 [id=22]	INFO	winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
