There is a possibility to access Rapla from another java application through the [Rapla-API](http://rapla.googlecode.com/svn/trunk/javadoc/index.html) and the Rapla facade. You find a sample that connects to a server in the source distribution under `src/org/rapla/examples/RaplaConnectorTest.java`

Make sure you have `rapla-{currentVersion}.jar` in your classpath. You may need to adjust the login and server settings.

Here is the source code from Rapla Connector Test

```
package org.rapla.examples;

import java.util.Locale;

import org.rapla.RaplaMainContainer;
import org.rapla.entities.domain.Allocatable;
import org.rapla.facade.ClientFacade;
import org.rapla.framework.RaplaContext;
import org.rapla.framework.RaplaException;
import org.rapla.framework.StartupEnvironment;
import org.rapla.framework.logger.ConsoleLogger;
/** Simple demonstration for connecting your app and importing some users. See sources*/
public class RaplaConnectorTest
{
    public static void main(String[] args) {
        final ConsoleLogger logger = new ConsoleLogger( ConsoleLogger.LEVEL_INFO);

        try
        {
            // Connects to http://localhost:8051/
            // and calls rapla/rpc/methodNames for interacting 
            StartupEnvironment env = new SimpleConnectorStartupEnvironment( "localhost", 8051,"/", false, logger);
            RaplaMainContainer container = new RaplaMainContainer( env);
            RaplaContext  context = container.getContext();

            // get an interface to the facade and login
            ClientFacade facade = context.lookup(ClientFacade.class);

            if ( !facade.login( "admin", "".toCharArray()) ) {
                throw new RaplaException("Can't login");
            }

            // query resouce
            Allocatable firstResource = facade.getAllocatables() [0] ;
            logger.info( firstResource.getName( Locale.getDefault()));

            // cleanup the Container
            container.dispose();
        }
        catch ( Exception e )
        {
            logger.error("Could not start test ",  e );
        }

    }

}


```