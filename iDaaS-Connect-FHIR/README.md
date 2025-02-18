# iDaaS Connect FHIR
This is the upstream for RedHat Healthcare's <a href="https://github.com/RedHat-Healthcare/iDaaS-Connect/tree/master/iDaaS-Connect-FHIR" target="_blank">iDaaS Connect FHIR</a>. iDAAS has several key components that provide many capabilities. iDAAS Connect is intended ONLY
to enable iDAAS connectivity. iDAAS-Connect-FHIR specifically ONLY deals with enabling 
iDAAS to process the healthcare industry standard FHIR based resources ONLY. Here is the 
<a href="https://www.hl7.org/fhir/resourcelist.html" target="_blank">current FHIR Resource List</a> 
It will process over 80+ of the currently available resources - around 40 clinical FHIR 
resources, all the financial public health and research/evidence based medicine/and quality
reporting and testing resources. You can also find a list of the 
<a href="http://connectedhealth-idaas.io/home/SupportedTransactions" target="_blank">platforms supported transactions</a> 

# General Background
When designing anb building this capability we wanted to ensure that this was an enablement. This means that we focused 
on the ability to enable systems to process FHIR resource data and determine if they wanted to persist the specific FHIR 
resources to a FHIR server. If they did then we made sure the design pattern included supporting multiple FHIR servers. 
So, whatever your choice to process to a FHIR server or not, we enable all these behaviors with simple configuration so
you don't have to do anything but make simple configuration changes in the application.properties.

## FHIR Servers Supported
For this particular repository it has been tested and works with multiple FHIR servers. <br/>
<a href="https://github.com/hapifhir/hapi-fhir-jpaserver-starter" target="_blank">HAPI FHIR JPA Server</a><br/>
<a href="https://github.com/IBM/FHIR" target="_blank">IBM FHIR Server</a><br/>
<a href="https://github.com/microsoft/fhir-server" target="_blank">Microsoft Azure FHIR Server</a><br/>

Which ever FHIR server you implement you will need to follow the specific install instructions from each vendor.
While we have tested with all three of them there could be a need to reconfigure the connectivity details to it.

## Pre-Requisites
For all iDaaS design patterns it should be assumed that you will either install as part of this effort, or have the following:

1. An existing Kafka (or some flavor of it) up and running. Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/Project-Herophilus/Project-Herophilus-Assets/blob/main/Kafka.md) <br/>
[KafkaWindows](https://github.com/Project-Herophilus/Project-Herophilus-Assets/blob/main/KafkaWindows.md) <br/>
No matter the platform chosen it is important to know that the Kafka out of the box implementation might require some changes depending upon your implementation needs. Here are a few we have made to ensure: <br/>
In /config/consumer.properties file we will be enhancing the property of auto.offset.reset to earliest. This is intended to enable any new system entering the group to read ALL the messages from the start. <br/>
auto.offset.reset=earliest <br/>
2. Some understanding of building, deploying Java artifacts and the commands associated. If using Maven commands then Maven would need to be intalled and runing for the environment you are using. More details about Maven can be found [here](https://maven.apache.org/install.html)<br/>
3. An internet connection with active internet connectivity, this is to ensure that if any Maven commands are
run and any libraries need to be pulled down they can.<br/>

## Scenario: Integration 
This repository follows a very common general facility based implementation. The implementation
is of a facility, we have named MCTN for an application we have named MMS. This implementation 
specifically defines one FHIR endpoint per FHIR resource.

### Integration Data Flow Steps
 
1. This respository acts as an HTTP/HTTP(s) secure endpoint for processing FHIR Data. Each FHIR Resource has a 
specifically defined URL endpoint. It posts the transactions and gets a response back.
2. iDAAS Connect FHIR will do the following actions:<br/>
    a. Receive the FHIR message. Internally, it will audit the data it received to 
    a specifically defined topic.<br/>
    b. The FHIR message will then be processed to a specifically defined topic for this implementation. There is a 
    specific topic pattern -  for the facility and application each data type has a specific topic define for it.
    For example: FHIRSvr_AdverseEvent, FHIRSvr_Consent, etc. <br/>
    c. If the code is enabled then the FHIR resource data can be sent to an external FHIR server. If and external 
    FHIR server is configured the respomse from it will then be sent back to the FHIR client.<br/>
    d. The response is also sent to the auditing topic location.<br/>
    
# Start The Engine!!!
This section covers the running of the solution. There are several options to start the Engine Up!!!

## Step 1: Kafka Server To Connect To
In order for ANY processing to occur you must have a Kafka server running that this accelerator is configured to connect to.
Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/Project-Herophilus/Project-Herophilus-Assets/blob/main/Kafka.md) <br/>
[KafkaWindows](https://github.com/Project-Herophilus/Project-Herophilus-Assets/blob/main/KafkaWindows.md) <br/>

## Step 2: Running the App: Maven or Code Editor
This section covers how to get the application started.

### Design Pattern/Accelerator Configuration
All iDaaS Design Pattern/Accelelrators have application.properties files to enable some level of reusability of code and simplfying configurational enhancements.<br/>
In order to run multiple iDaaS integration applications we had to ensure the internal http ports that
the application uses. In order to do this we MUST set the server.port property otherwise it defaults to port 8080 and ANY additional
components will fail to start. iDaaS Connect HL7 uses 9980. You can change this, but you will have to ensure other applications are not
using the port you specify.

Alternatively, if you have a running instance of Kafka, you can start a solution with:
`./platform-scripts/start-solution-with-kafka-brokers.sh --idaas.kafkaBrokers=host1:port1,host2:port2`.
The script will startup iDAAS server.

It is possible to overwrite configuration by:
1. Providing parameters via command line e.g.
2. Leverage the respective application.properties file in the correct location to ensure the properties are properly set
To use with a custom location `./start-solution.sh --spring.config.location=file:./config/application.properties`. However,
if you run from a Java IDE or from any command line that just invokes the jar it will automatically pull the application.properties
file in the resources directory closest to the jar file.

As you look at the properties the idaas.fhirVendor on startup determines the FHIR Server that would be
leveraged:
Supported properties include:
```
idaas.fhirVendor=ibm
idaas.fhirVendor=hapi
idaas.fhirVendor=microsoft


idaas.kafkaBrokers=localhost:9092 #a comma separated list of kafka brokers e.g. host1:port1,host2:port2
idaas.fhirVendor=ibm
idaas.ibmURI=http://localhost:8090/fhir-server/api/v4/
idaas.hapiURI=http://localhost:8080/hapi/api/v4/
idaas.msoftURI=http://localhost:9999/microsoftapi/api/v4/
```

# Testing
In order to assist anyone implementing or testing this specific reference architecture we have in the put a series of 
assets in place. 

## Test Data
To ensure resources have a variety of data to leverage for complete testing into our main assets [area](https://github.com/Project-Herophilus/Project-Herophilus-Assets/tree/main/Testing/TestData).
We have put testing data into specifically defined directories to ensure it is simply to find and leverage.

## API - Testing
When it comes to any solution APIs can be tested by leveraging our published Postman collections.
### API - Published Postman Content
Here is the general invite to the [PostmanCollection](https://app.getpostman.com/join-team?invite_code=2ad1e1b6b06ad4f377a54466d8136417&ws=7d70ed7c-dd18-48d6-95ec-f325d13e67f3). Or,
you can leverage the published [PostmanCollections](https://universal-capsule-967150.postman.co/users/3200250),
it is important to use either ProjectHerophilus-FHIR or iDaaS-FHIR shared collections.

# Defects/Bugs
All defects or bugs should be submitted through the GitHub Portal under the 
<a href="https://github.com/Project-Herophilus/iDaaS-Connect/issues" target="_blank">issues tab</a>

# Chat and Collaboration
You can always leverage the discussion boards in Github to post any feedback and file issues.

If you would like to contribute feel free to, contributions are always welcome!!!! 

Happy using and coding....

