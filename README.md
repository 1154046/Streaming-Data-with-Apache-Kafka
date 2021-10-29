# Streaming Data with Apache Kafka Quicklab 

# Workshop Resources
Login/Sign Up for IBM Cloud: https://ibm.biz/BdfJXM

Hands-On Guide: https://github.com/1154046/Streaming-Data-with-Apache-Kafka/

Slides: https://github.com/1154046/Streaming-Data-with-Apache-Kafka/

Workshop Replay: https://www.crowdcast.io/e/reactivejavaopenshift

This Repo is for the upcoming webinar "Build cloud-native event stream apps with Open Liberty and Quarkus" - Register for the live stream and access the replay – https://www.crowdcast.io/e/reactivejavaopenshift

# Prerequisites

## 1. Sign-up/Login to IBM Cloud - https://ibm.biz/BdfJXM

If you are an existing user please login to IBM Cloud

And if you are not, don't worry! We have got you covered! There are 3 steps to create your account on IBM Cloud:

1. Put your email and password.

2. You get a verification link with the registered email to verify your account.

3. Fill the personal information fields.
** Please make sure you select the country you are in when asked at any step of the registration process.

## 2. Docker 

Docker is an open source containerization platform. It enables developers to package applications into containers—standardized executable components combining application source code with the operating system (OS) libraries and dependencies required to run that code in any environment.

Follow this link https://docs.docker.com/get-docker/ to install Docker.

## 3. Maven

Maven is a build automation tool used primarily for Java projects.

Follow this link - https://maven.apache.org/install.html/ to install Maven.

# Workshop Content

1. [Create an account with IBM Cloud](#1-create-an-account-with-ibm-cloud)
1. [Get the source code](#2-get-the-source-code)
1. [Start Kafka and ZooKeeper](#3-start-kafka-and-zookeeper)
1. [Start Quarkus in dev mode](#4-start-quarkus-in-dev-mode)
1. [Exploring the Code](#5-exploring-the-code)
1. [Provision an OpenShift Cluster](#6-provision-an-openshift-cluster)

## 1. Create an account with IBM Cloud

Sign in to IBM [**Cloud**]( https://ibm.biz/BdfJXM). By clicking on create a free account you will get 30 days trial account.

This is essential as it will allow you to create your free cluster in the next step.

## 2. Get the source code

Open a new Terminal and [Clone this repo](https://github.com/1154046/Streaming-Data-with-Apache-Kafka/)


Clone the source code:

`
git clone https://github.com/1154046/Streaming-Data-with-Apache-Kafka/edit/master/README.md
`

Then go into the cloned git directory:

`
cd Streaming-Data-with-Apache-Kafka-Quicklab/
`

## 3. Start Kafka and Zookeeper

From the cloned directory, copy this in a terminal and hit enter:

`
docker-compose up
`

This will start up a minimal Kafka instance (and Zookeeper)

## 4. Start Quarkus in dev mode

Open a second terminal window.

Start Quarkus in dev mode, remember to change into code directory.

`
cd Streaming-Data-with-Apache-Kafka-Quicklab/
`

`
./mvnw quarkus:dev
`


You will need to wait for a few seconds for Quarkus to start and connect to Kafka, look for 'process data:' in the output to know when it's up and running

Execute this command to see the messages streaming from Kafka into the Quarkus application (you will need a third terminal):


Linux/MacOS:

`
curl -N http://localhost:8080/windSpeed/stream
`

The curl command should also work on Powershell if you are using Windows, alternatively:

Windows:

`
Invoke-WebRequest -Uri "http://localhost:8080/windSpeed/stream"
`

You should see some random numbers being generated every second. If you look at the terminal you're running Quarkus in dev mode, you will see the same number being printed out.


## 5. Exploring the Code

### Examine and learn how we use Kafka in Quarkus

Navigate to the following directory using your prefered IDE:
Streaming-Data-with-Apache-Kafka-Quicklab/src/main/java/org/acme/WindGenerator.java

### Sending a periodic message

We're using the Reactive Messaging Flowable to send a random number every second to the windSpeedKph channel. We'll see below how we configure to send this to Kafka.


The @Outgoing annotation is used to mark this method as something that sends to the configured channel. This class is marked with the @ApplicationScoped annotation so that it can be used with Quarkus CDI. This also allows it to be discovered by the plumbing provided by the Quarkus Reactive Messaging Kafka plugin. We define that in the pom.xml like so:

```
  <dependency>
    <groupId>io.quarkus</groupId>
    
    <artifactId>quarkus-smallrye-reactive-messaging-kafka</artifactId>
  </dependency>
```

### Receiving messages

Now open up WindSpeedConverter and have a look at the process method:

```
  @Incoming("windSpeed")

  @Outgoing("windSpeedMph")

  @Broadcast

  public double process(int windSpeedinKph) {

    return windSpeedinKph * KphToMph;
  }
```

You can see we're using two annotations, @Incoming and @Outgoing with corresponding channel names. This method takes inputs from windSpeed and outputs to windSpeedMph, hence it is tranforming the data and re-broadcasting it. 

We setup periodic sending to the windSpeedKph channel but this receiver is listening to windSpeed … how are messages making it from one to the other? The next section discusses using Kafka for messaging.



### Setting up Kafka as the messaging engine
Open up this file to see how we've configured Kafka to be used in our Quarkus project:
Streaming-Data-with-Apache-Kafka-Quicklab/src/main/resources/application.properties

```
  # Outgoing
  mp.messaging.outgoing.windSpeedKph.connector=smallrye-kafka
  mp.messaging.outgoing.windSpeedKph.topic=windSpeed
  mp.messaging.outgoing.windSpeedKph.value.serializer=org.apache.kafka.common.serialization.IntegerSerializer
  # Incoming
  mp.messaging.incoming.windSpeed.connector=smallrye-kafka
  mp.messaging.incoming.windSpeed.value.deserializer=org.apache.kafka.common.serialization.IntegerDeserializer
```

Now you should be able to connect the dots from the windSpeedKph channel to the windSpeed channel (and intermediate windspeed Kafka topic). Instead of using an in-memory channel, this configuration sets up Kafka as the messaging provider (along with the inclusion of the Quarkus-Kafka messaging dependency/JAR file we mentioned above)

What about the @Outgoing("windSpeedMph") channel?

This is an in-memory channel, that lives in the Quarkus server instance.


Using an in-memory channel to send/receive messages, and streaming data over HTTP.

We are converting the incoming data and broadcasting it to the windSpeedMph channel. Let's see where we consume those
messages; open this file:
Streaming-Data-with-Apache-Kafka-Quicklab/src/main/java/org/acme/WindSpeedResource.java

And look at the stream method:

```
  @Inject
  @Channel("windSpeedMph") Publisher<Double> windSpeed;
  @GET
  @Path("/stream")
  @Produces(MediaType.SERVER_SENT_EVENTS)
  @SseElementType("text/plain")

  public Publisher<Double> stream() {
    return windSpeed;
  }
```

We @Inject the @Channel called windSpeedMph into this method in the variable windSpeed. The remainder of the annotations setups a HTTP endpoint with @Path("/stream") that @Produces(MediaType.SERVER_SENT_EVENTS) and responds to a @GET request. This class has a @Path("/windSpeed")


When you ran this from the commandline in an earlier step of this tutorial:

`
curl -N http://localhost:8080/windSpeed/stream
`

You were hitting this endpoint!

### Sending and process a message from an HTTP request

Now that we've connected all the dots for the sample application, let's write some code to manually send a wind speed measurement and convert/display it.

### Send a message into Kafka

We're going to create a new in-memory channel windSpeedManual to send a message from an HTTP request. We'll @Inject a @Channel("windSpeedManual") and set it up as an Emitter. 

The @Path("/generate/{speed}") is where we can invoke this routine. Add this code to the appropriate places in the WindSpeedResource.java file.

```
  import org.eclipse.microprofile.reactive.messaging.Channel;
  import org.eclipse.microprofile.reactive.messaging.Emitter;
  import org.jboss.resteasy.annotations.SseElementType;
  import org.reactivestreams.Publisher;
  import javax.inject.Inject;
  import javax.ws.rs.*;
  import javax.ws.rs.core.MediaType;
  import javax.ws.rs.core.Response;
  ....
  @Inject @Channel("windSpeedManual") Emitter<Integer> windSpeedEmitter;
  @POST
  @Path("/generate/{speed}")

  public Response generate(@PathParam("speed") Integer speed) {
    windSpeedEmitter.send(speed);
    return Response.status(Response.Status.CREATED).entity(speed).build();
  }
```

### Consume a message from Kafka

Once we have the above setup, we can receive the in-memory message from the @Incoming("windSpeedManual") channel and send it out over the @Outgoing("windSpeedKph") so it can be processed with the other messages already moving through the system (that we setup in the periodic message sender previously).

Add this code to WindGenerator.java

```
  import org.eclipse.microprofile.reactive.messaging.Incoming;
  import org.eclipse.microprofile.reactive.messaging.Outgoing;
  ....
  @Incoming("windSpeedManual")
  @Outgoing("windSpeedKph")
  public Integer generateManual(int windSpeedManual) {
  return windSpeedManual;
  }
```

### Send a message using HTTP
Everything is setup and ready to go! Send a message by invoking the HTTP endpoint using a curl command line so:

`
curl -v -X POST http://localhost:8080/windSpeed/generate/100
`

Of course, make sure you're still running Quarkus in dev mode:

`
./mvnw quarkus:dev
`

The value we're sending in is 100, but remember that we're converting from KPH to MPH, so you won't see 100, but you're see: 62.137100000000004!


## 6. Provision an Openshift Cluster and Deploy to OpenShift

We can run our project on OpenShift and hit the endpoints from our OpenShift Route(Our app URL) instead of running everything locally.

Provision a free OpenShift Cluster [here](https://reactivejavaos.eohluqb207s.us-south.codeengine.appdomain.cloud).

Enter "oslab" for 'Lab Key' and Enter the email address you used to sign up for IBM Cloud under 'Your IBMId'.

![provision cluster](https://user-images.githubusercontent.com/20628307/139394343-cb958f24-b325-41dc-8902-eb1114236d35.png)

![provision cluster success](https://user-images.githubusercontent.com/20628307/139394399-969a6fc5-4b38-4c4a-bdf3-ccc0b05834c0.png)

You will receive an email invitation to join the IBM Cloud Organization in which your cluster is provisioned. You will need to accept the email before you proceed.


Navigate to your cluster https://cloud.ibm.com/resources. Make sure you selected the right organization on the drop down menu next to 'Manage' in the navigation bar.

The cluster should be visible under 'Clusters'. Click on your Cluster to view the Cluster Dashboard.

We want to deploy our App to OpenShift. 

Click on "Open OpenShift Console" and the copy your login command.


Run the login command:
`
oc login token="..."
`

Next we want to create a new project and deploy our application:

`
oc new-app https://github.com/1154046/Streaming-Data-with-Apache-Kafka/
`

Navigate back to your dashboard, here you can see the newly deployed app and can copy the app route which you can use in your curl commands.



# Next Steps: 

- Try out the lab for 'Creating reactive microservices using MicroProfile reactive messaging' – https://labs.cognitiveclass.ai/tools/theiaopenshift/lab/tree?md_instructions_url=https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/creating-reactive-microservices-using-microprofile-reactive-messaging/instructions.md

- See instructions.md file for additional code content!

Thank you,

Sbusiso Mkhombe

Cloud Engineer, Hybrid Cloud Build Team

IBM Technology Sales

Sbusiso.Mkhombe@ibm.com 
