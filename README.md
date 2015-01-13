JOSSO Addon
=======

### Install and configure eXo Platform
- If you want to integrate with JOSSO version 1.8.1. Goto `$PLATFORM_HOME` then run:
```
./addon.sh install exo-josso-181
```
- If you want to integrate with JOSSO version 1.8.2 or above, go to `$PLATFORM_HOME` then run:
```
./addon.sh install exo-josso
```

#### Configure for Platform tomcat
- Edit `$PLATFORM_HOME/gatein/conf/exo.properties` and update these configuration
```
#SSO
gatein.sso.enabled=true
gatein.sso.callback.enabled=${gatein.sso.enabled}
gatein.sso.login.module.enabled=${gatein.sso.enabled}
gatein.sso.login.module.class=org.gatein.sso.agent.login.SSOLoginModule
gatein.sso.josso.agent.config.file=sso/josso/1.8/josso-agent-config.xml
gatein.sso.josso.properties.file=file:${PLATFORM_HOME}/gatein/conf/exo.properties
gatein.sso.josso.host=localhost:8888
gatein.sso.josso.base.url=http://${gatein.sso.josso.host}/josso/signon
gatein.sso.server.url=${gatein.sso.josso.base.url}/login.do
gatein.sso.portal.url=http://localhost:8080
gatein.sso.filter.logout.class=org.gatein.sso.agent.filter.JOSSOLogoutFilter
gatein.sso.filter.logout.url=${gatein.sso.josso.base.url}/logout.do
gatein.sso.filter.login.sso.url=${gatein.sso.server.url}?josso_back_to=${gatein.sso.portal.url}/@@portal.container.name@@/initiatessologin
```
Note: Please replace `${PLATFORM_HOME}` with the real full path of `$PLATFORM_HOME` 

- Add `<Valve className="org.gatein.sso.agent.tomcat.ServletAccessValve" />` into `$PLATFORM_HOME/conf/server.xml`, the content of this file will look like:
```xml
...
    <Engine name="Catalina" defaultHost="localhost">
        <Host name="localhost" appBase="webapps" startStopThreads="-1"
              unpackWARs="${EXO_TOMCAT_UNPACK_WARS}" autoDeploy="true">
            <Valve className="org.gatein.sso.agent.tomcat.ServletAccessValve" />
            ... 
            <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
            ...
            <Listener className="org.exoplatform.platform.server.tomcat.PortalContainersCreator" />
            ...
        </Host>
    </Engine>
...
```

#### Configure for Platform jboss
- Edit `$PLATFORM_HOME/standalone/configuration/gatein/exo.properties` then update following configuration (add new if they do not exist):
```
#SSO
gatein.sso.enabled=true
gatein.sso.callback.enabled=${gatein.sso.enabled}
gatein.sso.login.module.enabled=${gatein.sso.enabled}
gatein.sso.login.module.class=org.gatein.sso.agent.login.SSOLoginModule
gatein.sso.josso.agent.config.file=sso/josso/1.8/josso-agent-config.xml
gatein.sso.josso.properties.file=file:${jboss.home.dir}/standalone/configuration/gatein/exo.properties
gatein.sso.josso.host=localhost:8888
gatein.sso.josso.base.url=http://${gatein.sso.josso.host}/josso/signon
gatein.sso.server.url=${gatein.sso.josso.base.url}/login.do
gatein.sso.portal.url=http://localhost:8080
gatein.sso.filter.logout.class=org.gatein.sso.agent.filter.JOSSOLogoutFilter
gatein.sso.filter.logout.url=${gatein.sso.josso.base.url}/logout.do
gatein.sso.filter.login.sso.url=${gatein.sso.server.url}?josso_back_to=${gatein.sso.portal.url}/@@portal.container.name@@/initiatessologin
```

- Uncomment the below login module in `$PLATFORM_HOME/standalone/configuration/standalone-exo.xml`, then change `${gatein.sso.login.module.enabled}` and `${gatein.sso.login.module.class}` into `#{gatein.sso.login.module.enabled}` and `#{gatein.sso.login.module.class}` respectively.
```xml
<login-module code="org.gatein.sso.integration.SSODelegateLoginModule" flag="required">
    <module-option name="enabled" value="#{gatein.sso.login.module.enabled}"/>
    <module-option name="delegateClassName" value="#{gatein.sso.login.module.class}"/>
    <module-option name="portalContainerName" value="portal"/>
    <module-option name="realmName" value="gatein-domain"/>
    <module-option name="password-stacking" value="useFirstPass"/>
</login-module>
```


### Install and configure JOSSO server
- You can download JOSSO from: http://sourceforge.net/projects/josso/files/

##### Configure for JOSSO 1.8.x
Note: It is recommended that you download the packaging bundled with Tomcat (often started with "apache") , for example http://sourceforge.net/projects/josso/files/JOSSO/JOSSO-1.8.2/apache-tomcat-6.0.29_josso-1.8.2.tar.gz/download. If you want to download a JOSSO standalone package, you should follow this guideline: http://www.josso.org/confluence/display/JOSSO1/Install+JOSSO+Gateway+-+Tomcat+6.0 for setting up josso gateway server.

After download (and setup if need), we will have JOSSO tomcat gateway server at $JOSSO_TOMCAT_HOME 

- If you are using JOSSO 1.8.1: copy all file and folder in `$PLATFORM_HOME/josso181-plugin` into `$JOSSO_TOMCAT_HOME`
- If you are using JOSSO 1.8.2 or later: copy all file and folder in `$PLATFORM_HOME/josso-plugin/josso` into `$JOSSO_TOMCAT_HOME`
- Edit `$JOSSO_TOMCAT_HOME/conf/server.xml` then change ports to avoid conflict when run JOSSO and Platform on the same machine. In this case we will change HTTP port from 8080 to 8888 for using.
- Start JOSSO tomcat with command: `./catalina.sh run`

##### Configure for JOSSO 2.x
Following the this guideline: http://docs.exoplatform.com/public/index.jsp?topic=%2FPLF41%2Fsect-Reference_Guide-Single_Sign_On-JOSSO1.html

Note: If you are using JOSSO 2.2.0: You will need to copy all file in $PLATFORM_HOME/josso-plugin/josso-2.2.0/ into $JOSSO_HOME 

If you are using JOSSO 2.4.x:
- Default username/password is: `admin/atricore`
- You need ignore the `step 6`, do not create `SP1Users` and wire it to `SP1`
- In step 8, You can not wire `SP1` and `SP1EE` directly, you will have to create a `JOSSO 1` resource (in `Resources` >> `JOSSO1`). 
Its name is `JOSSO1-RE` and Partner Application Location is `http://localhost:8080/portal`.
And now, you will wire `SP1` and `JOSSO1-RE` by a `Service connection` and wire `JOSSO1-RE` and `SP1EE` by `activation` connection.

Please read more at: http://sourceforge.net/p/josso/discussion/399715/thread/2590a1bd/

