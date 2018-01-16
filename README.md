Role Name: tomcat
=================

Deploy Apache Tomcat

Requirements
------------

GNU Tar or unzip, depending on the archive format.

Role Variables
--------------

Default variables in `defaults/main.yml`:

- Tomcat OS user:

	```yaml
	tomcat_os_user:
	  - name: "tomcat"              # Tomcat OS service user
	    group: "tomcat"             # Tomcat OS group
	    shell: "/sbin/nologin"      # Tomcat user shell
	    createhome: false           # Whether to create home dir for OS user or not
	    home: "/home/tomcat"        # Tomcat OS user home
	```

- Variables related to tomcat installation:

	|	Variable name	|	Default value	|	Description	|
	|-----------------------|-----------------------|-----------------------|
	| `tomcat_update` | `false` | Set to `true` when updating tomcat. Ensures tomcat is running tomcat instance is stopped before updating |
	| `tomcat_daemon_name` | `tomcat` | Name of the tomcat OS service/daemon |
	| `tomcat_version` | `8.5.24` | Version of Apache Tomcat to install |
	| `tomcat_archive` | `http://www.apache.org/dist/tomcat/tomcat-{{tomcat_version.split('.')[0]}}/v{{tomcat_version}}/bin/apache-tomcat-{{tomcat_version}}.tar.gz` | Tomcat tarball download url |
	| `tomcat_base_dir` | `/opt` | Base directory where tomcat archive should be extracted |
	| `tomcat_catalina_home` | `/opt/apache-tomcat-{{tomcat_version}}` | Tomcat installation directory |
	| `tomcat_jdbc_url` | `""` | Download url for JDBC driver |
	| `tomcat_jdbc_path` | `[]` | List of paths on the host where the JDBC driver will be downloaded, example: `{{tomcat_catalina_home}}/lib/{{ tomcat_jdbc_driver }}` |
	| `tomcat_systemd:`<br/>&nbsp;&nbsp;`ExecStartPre:`<br/>&nbsp;&nbsp;`ExecStartPost:`<br/>&nbsp;&nbsp;`ExecStopPost:` | <br/>`[]`<br/>`[]`<br/>`[]` | Systemd service pre-start, post-start and post-stop commands. Multiple commands (var list items) only supported when `Type=Oneshot` in the systemd unit service |

- Declare a list of IP addresses to which remote access is allowd to Tomcat administration apps. This variable is used for templating `context.xml` files inside webapps.

	```yaml
	tomcat_context_acl:
		- 192.168.0.5
		- 192.168.1.2
	```

- Tomcat console roles (administration webapps). This variable is used for templating the `tomcat-users.xml` configuration file.

	```yaml
	tomcat_roles:
	  - manager-gui
	  - manager-status
	  - manager-script
	  - manager-jmx
	  - admin-gui
	  - admin-script
	```

- The following variable defines the list of Tomcat console users, containging username, password and Tomcat role. This variable is used for templating the `tomcat-users.xml` configuration file.

	```yaml
	tomcat_console_users:
	  - name: "tomcat"
	    password: "s3cret"
	    roles: "manager-gui,admin-gui,manager-status"
	```

- Define environment variables here, used for templating `${catalina.home}/bin/setenv.sh` file. Most common variables in `setenv.sh` are `CATALINA_OPTS` and `JAVA_OPTS`. Each Java option is declared as a list of items fore ease of configration and they are joined on a single line at the `setenv.sh` template:

	```bash
	CATALINA_OPTS={{ tomcat_catalina_opts | join(' ') }}
	```

	```yaml
	tocmat_catalina_opts:
	  - "-server -Xms4096m -Xmx8192m"

	tocmat_java_opts:
	  - "-server -Xms4096m -Xmx8192m"
	```

	Extra `setenv.sh` items can be declared by the following multi-item varaible list/array:

	```yaml
	tomcat_setenv_extras:
	  - "JAVA_HOME=/usr"
	```

- Tomcat configuration variables, used for templating `${catalina.home}/conf/server.xml` file. Default values from tomcat installation are used here.

	```yaml
	tomcat_conf_listeners: |
	  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
	  <!-- Security listener. Documentation at /docs/config/listeners.html
	  <Listener className="org.apache.catalina.security.SecurityListener" />
	  -->
	  <!--APR library loader. Documentation at /docs/apr.html -->
	  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
	  <!-- Prevent memory leaks due to use of particular java/javax APIs-->
	  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
	  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
	  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
	```
	```yaml
	tomcat_conf_jndi_resources: |
	  <GlobalNamingResources>
	    <!-- Editable user database that can also be used by
	         UserDatabaseRealm to authenticate users
	    -->
	    <Resource name="UserDatabase" auth="Container"
	              type="org.apache.catalina.UserDatabase"
	              description="User database that can be updated and saved"
	              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
	              pathname="conf/tomcat-users.xml" />
	  </GlobalNamingResources>
	```
	```yaml
	tomcat_conf_connector_http: |
	  <Connector port="8080" protocol="HTTP/1.1"
	             connectionTimeout="20000"
	             redirectPort="8443" />
	```
	```yaml
	tomcat_conf_connector_https: |
	  <!--
	  <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
	             maxThreads="150" SSLEnabled="true" >
	      <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
	      <SSLHostConfig>
	          <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
	                       certificateFile="conf/localhost-rsa-cert.pem"
	                       certificateChainFile="conf/localhost-rsa-chain.pem"
	                       type="RSA" />
	      </SSLHostConfig>
	  </Connector>
	  -->
	```
	```yaml
	tomcat_conf_connector_ajp: |
	  <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
	```
	```yaml
	tomcat_conf_connector_extras: |
	```
	```yaml
	tomcat_conf_realm: |
	  <Realm className="org.apache.catalina.realm.LockOutRealm">
	    <!-- This Realm uses the UserDatabase configured in the global JNDI
	         resources under the key "UserDatabase".  Any edits
	         that are performed against this UserDatabase are immediately
	         available for use by the Realm.  -->
	    <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
	           resourceName="UserDatabase"/>
	  </Realm>
	```
	```yaml
	tomcat_conf_host: |
	  <Host name="localhost"  appBase="webapps"
	        unpackWARs="true" autoDeploy="true">

	    <!-- SingleSignOn valve, share authentication between web applications
	         Documentation at: /docs/config/valve.html -->
	    <!--
	    <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
	    -->

	    <!-- Access log processes all example.
	         Documentation at: /docs/config/valve.html
	         Note: The pattern used is equivalent to using pattern="common" -->
	    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
	           prefix="localhost_access_log" suffix=".txt"
	           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
	  </Host>
	```

- Extending configuration files `${catalina.home}/conf/context.xml` and `${catalina.home}/conf/tomcat-users.xml`:

	```yaml
	tomcat_disable_session_persistance: false
	tomcat_conf_context_xml_extras: |

	tomcat_conf_users_xml_extras: |
	```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
    - ichundu.tomcat
```

License
-------

GPLv2

Author Information
------------------

https://github.com/ichundu
