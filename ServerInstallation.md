# Making rapla ready for the multiuser scenario #

This installation is for the 1.8 version of Rapla for older Versions see the README-Server.txt file in the distribution folder of the Rapla application.

## The Client/Server Mode ##

Since version 0.10.2 Rapla server installation is very easy:
Download the binary-distribution or build one with [BuildGuide](BuildGuide.md)

You can start it with:
```
raplaserver.bat (Windows)
raplaserver.sh run (Unix)
```

If you want to install Rapla-Server as a SERVICE on Win NT/2000/XP/Vista/7 or linux
use the scripts in the service/bin folder to test/install/uninstall rapla as a service.

You can also try the script `service/bin/TestWrapper` instead of raplaserver if it fails because java command is not found.

With a running server point your browser to
http://localhost:8051

You should see the rapla welcome screen. Click on start with webstart
and the webstart client should launch. If not launched automatically,
you should choose the webstart program from your
java installation (e.g. C:\Program Files (x86)\Java\jre7\bin\javaws.exe under Windows).

You can login with username "admin" and empty password

Thats it.

## Configuring SSL with jetty ##

Use keytool to generate your ssl keypair. You can find the tool in your java bin directory.
call
```
keytool -alias rapla -genkeypair
```

choose a password. (secret is the default in jetty.xml)

Uncomment the ssl section in jetty/jetty.xml
(etc/jetty.xml since rapla 1.7)

There is a ssl problem with the newest firefox and some jetty ssl libs:

be sure to change

```
<New class="org.eclipse.jetty.http.ssl.SslContextFactory">
  <Set name="keyStore"><SystemProperty name="jetty.home" default="." />/.keystore</Set>
  <Set name="keyStorePassword">secret</Set>
  <Set name="certAlias">rapla</Set>
 </New>
```

to

```
 <New class="org.eclipse.jetty.http.ssl.SslContextFactory">
  <Set name="keyStore"><SystemProperty name="jetty.home" default="." />/.keystore</Set>
  <Set name="keyStorePassword">secret</Set>
  <Set name="certAlias">rapla</Set>
  <Set name="ExcludeCipherSuites">
    <Array type="String">
	     <Item>SSL_RSA_WITH_DES_CBC_SHA</Item>
	     <Item>SSL_DHE_RSA_WITH_DES_CBC_SHA</Item>
	     <Item>SSL_DHE_DSS_WITH_DES_CBC_SHA</Item>
	     <Item>SSL_RSA_EXPORT_WITH_RC4_40_MD5</Item>
	     <Item>SSL_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
	     <Item>SSL_DHE_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
	     <Item>SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA</Item>
	     <Item>SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA</Item>
	     <Item>TLS_DHE_RSA_WITH_AES_256_CBC_SHA256</Item>
	     <Item>TLS_DHE_DSS_WITH_AES_256_CBC_SHA256</Item>
	     <Item>TLS_DHE_RSA_WITH_AES_256_CBC_SHA</Item>
	     <Item>TLS_DHE_DSS_WITH_AES_256_CBC_SHA</Item>
	     <Item>TLS_DHE_RSA_WITH_AES_128_CBC_SHA256</Item>
	     <Item>TLS_DHE_DSS_WITH_AES_128_CBC_SHA256</Item>
	     <Item>TLS_DHE_RSA_WITH_AES_128_CBC_SHA</Item>
	     <Item>TLS_DHE_DSS_WITH_AES_128_CBC_SHA</Item>
    </Array>
   </Set>
   <Set name="ExcludeProtocols">
     <Array type="java.lang.String">
        <Item>SSLv3</Item>
     </Array>
  </Set>
</New>
```

if you get the following error code in firefox: ssl\_error\_weak\_server\_ephemeral\_dh\_key



Change the password (if you didn't choose secret).
Either copy .keystore form your home folder to the rapla directory or enter the path to .keystore in jetty.xml

Start raplaserver and go to https://localhost:8443/rapla

If the 8443 port is already used on your server choose another one and restart rapla.

## Rapla in the cloud ##

There is a deployment guide for cloud bees. [DeployingOnCloudBees](DeployingOnCloudBees.md)

Its also tested with openshift [DeployingOnOpenshift](DeployingOnOpenshift.md)


## Installing into jboss ##

For more information look  into [DeployingOnJBoss](DeployingOnJBoss.md)

## Installing into tomcat ##

For more information look  into [DeployingOnTomcat](DeployingOnTomcat.md)

## Installing into another servlet container ##

If you want to use another servlet container (e.g. tomcat) you can
copy the rapla.war in the webapps folder to the webapps folder of
your servlet container.

You need to set a jndi entry jdbc/rapladb in the servlet container if you are using a database or resource/raplafile if you are using the file, see contexts/rapla.xml for the setting in jetty. You also have to set a second environment variable rapladatasource to either raplafile (for file storage) or rapladb (for database storage).

See the documentation of your servelt-container for more details about how to set both entries from contexts/rapla.xml
in the container of your choice.

## Installing in Apache or IIS ##

You can install Jetty with apache or IIS by adding
the AJP13 listener in etc/jetty.xml.

See http://www.eclipse.org/jetty/ for more information

## Troubleshooting: ##

1. The most commen error will be long stacktrace with
> "Caused by: java.net.BindException: Could not bind to port 8051"
> somewhere hidden between the lines. This means that either another
> rapla server is running (so stop or kill this process) or that this
> port is blocked by another application.  You need to configure
> another the port in etc/jetty.xml

```
<Call name="addConnector">
      <Arg>
         <New class="org.eclipse.jetty.server.nio.SelectChannelConnector">
            <Set name="port"><SystemProperty name="jetty.port" default="8051"/></Set>
```

2. look in the troubleshooting faq under [FAQ](FAQ.md)

3. Post a question to rapla-developers@lists.sourceforge.net


## Running Multiple Rapla servers: ##

There is no Problem running multiple rapla servers on one computer. You just need to copy the contexts/rapla entry folder and rename it to another name and edit the entries

Considering  the original rapla.xml
```
  <Set name="contextPath">/</Set>
  <New id="raplafile" class="org.eclipse.jetty.plus.jndi.EnvEntry">
    <Arg></Arg>
	<Arg>raplafile</Arg>
	<Arg type="java.lang.String"><SystemProperty name="jetty.home" default="." />/data/data.xml</Arg>
	<Arg type="boolean">true</Arg>
  </New>
  <New id="rapladb" class="org.eclipse.jetty.plus.jndi.Resource">
	<Arg>jdbc/rapladb</Arg>
	<Arg>
	   <New class="org.hsqldb.jdbc.JDBCDataSource">
         <Set name="Url">jdbc:hsqldb:data/rapla-hsqldb</Set>
         <Set name="User">db_user</Set>
         <Set name="Password">your_pwd</Set>
	   </New>
	</Arg>
 </New>
 <New id="rapladatasource" class="org.eclipse.jetty.plus.jndi.EnvEntry">
    <Arg></Arg>
	<Arg>rapladatasource</Arg>
	<Arg type="java.lang.String">raplafile</Arg>
	<!-- 
	<Arg type="java.lang.String">rapladb</Arg>
	 -->
	<Arg type="boolean">true</Arg>
 </New>
```
> If you want a second instance running on a different context. e.g. secondRapla.xml
```
  <Set name="contextPath">/secondRapla</Set>
  <New id="raplafile" class="org.eclipse.jetty.plus.jndi.EnvEntry">
    <Arg></Arg>
	<Arg>raplafile</Arg>
	<Arg type="java.lang.String"><SystemProperty name="jetty.home" default="." />/data/secondRapla_data.xml</Arg>
	<Arg type="boolean">true</Arg>
  </New>
  
  <New id="rapladb" class="org.eclipse.jetty.plus.jndi.Resource">
	<Arg>jdbc/rapladb</Arg>
	<Arg>
	   <New class="org.hsqldb.jdbc.JDBCDataSource">
         <Set name="Url">jdbc:hsqldb:data/secondRapla-hsqldb</Set>
         <Set name="User">db_user</Set>
         <Set name="Password">your_pwd</Set>
	   </New>
	</Arg>
 </New>
 <New id="rapladatasource" class="org.eclipse.jetty.plus.jndi.EnvEntry">
    <Arg></Arg>
	<Arg>rapladatasource</Arg>
	<Arg type="java.lang.String">raplafile</Arg>
	<!-- 
	<Arg type="java.lang.String">rapladb</Arg>
	 -->
	<Arg type="boolean">true</Arg>
 </New>
```



Now you should have two applications running

http://localhost:8051/

http://localhost:8051/secondRapla/
