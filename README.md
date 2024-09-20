# Op-Scripts

/* show-time.slax */

version 1.0;

ns junos = "http://xml.juniper.net/junos/*/junos";

ns xnm = "http://xml.juniper.net/xnm/1.1/xnm";

ns jcs = "http://xml.juniper.net/junos/commit-scripts/1.0";

import "../import/junos.xsl";

/* This is imported into the Junos CLI help text */

var $arguments = {

<argument> {

<name> "display-format";

<description> "Choose either iso or normal";

}

}

/* Command-line argument */


param $display-format;

match / {

<op-script-results> {

/* Call the display-time template */

call display-time;

}

}

/* Output the localtime to the console in either iso (default) or normal

format */

template display-time {

if( $display-format == "iso" ) {

<output> "The iso time is" _ $localtime_iso;


}

else {

<output> "The time is" _ $localtime;

}

}

user@Junos> op show-time display-format iso

The iso time is 2009-05-12 21:01:10 PDT

user@Junos> op show-time display-format normal

The time is Tue May 12 21:01:13 2009


****************Static route************


/* add-route.slax */

version 1.0;

ns junos = "http://xml.juniper.net/junos/*/junos";

ns xnm = "http://xml.juniper.net/xnm/1.1/xnm";

ns jcs = "http://xml.juniper.net/junos/commit-scripts/1.0";

import "../import/junos.xsl";

match / {

<op-script-results> {


/* Ask for the static route */

var $static-route = jcs:get-input("Enter the static route: ");

/* Ask for the next-hop */

var $next-hop = jcs:get-input("Enter the next-hop: ");

/* Create the configuration change */

var $configuration = <configuration> {

<routing-options> {

<static> {

<route> {

<name> $static-route;

<next-hop> $next-hop;

}

}

}

}

/* Open a connection */

var $connection = jcs:open();

/* Call jcs:load-configuration and provide the connection and

configuration


* change to make.
*/
var $results := { call jcs:load-configuration( $connection,

$configuration ); }

/* Check for errors – report them if they occurred */

if( $results//xnm:error ) {

for-each( $results//xnm:error ) {


<output> message;

}

}

/* If there are no errors then report success */

if( jcs:empty( $results//xnm:error ) ) {

<output> "Committed without errors.";

}


/* Close the connection */

var $close-results = jcs:close($connection);

}

}

Here is an example of the add-route op script in operation:

user@Junos> op add-route

Enter the static route: 10.6.0.0/16

Enter the next-hop: 192.168.1.4

Committed without errors.

And here is the configuration after the scripted change (two routes were already

present):

user@Junos> show configuration routing-options

static {

route 10.1.0.0/16 next-hop 192.168.1.1;

route 10.2.0.0/16 next-hop 192.168.1.1;

route 10.6.0.0/16 next-hop 192.168.1.4;

}
