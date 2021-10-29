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
1. [Provision an OpenShift Cluster](#2-provision-an-openshift-cluster)
1. [Get the source code](#3-get-the-source-code)
1. [Start Kafka and ZooKeeper](#4-start-kafka-and-zookeeper)
1. [Start Quarkus in dev mode](#5-start-quarkus-in-dev-mode)
1. [Exploring the Code](#6-exploring-the-code)
1. [Analyze the results](#7-analyze-the-results)

## 1. Create an account with IBM Cloud

Sign in to IBM [**Cloud**]( https://ibm.biz/BdfJXM). By clicking on create a free account you will get 30 days trial account.

This is essential as it will allow you to create your free cluster in the next step.

## 2. Provision an Openshift Cluster

Provision a free OpenShift Cluster [here](https://reactivejavaos.eohluqb207s.us-south.codeengine.appdomain.cloud).

Enter "oslab" for 'Lab Key' and Enter the email address you used to sign up for IBM Cloud under 'Your IBMId'.

![provision cluster](https://user-images.githubusercontent.com/20628307/139394343-cb958f24-b325-41dc-8902-eb1114236d35.png)

![provision cluster success](https://user-images.githubusercontent.com/20628307/139394399-969a6fc5-4b38-4c4a-bdf3-ccc0b05834c0.png)


## 3. Get the source code

[Clone this repo](https://github.com/1154046/Streaming-Data-with-Apache-Kafka/edit/master/README.md)

Open a new Terminal.

Clone the source code:

`
git clone https://github.com/1154046/Streaming-Data-with-Apache-Kafka/edit/master/README.md
`

Then go into the cloned git directory:

`
cd Streaming-Data-with-Apache-Kafka-Quicklab/
`

## 4. Start Kafka and Zookeeper

From the cloned directory, copy this in a terminal and hit enter:

`
docker-compose up
`

This will start up a minimal Kafka instance (and Zookeeper)

## 5. Start Quarkus in dev mode

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

Windows:

`
Invoke-WebRequest -Uri "http://localhost:8080/windSpeed/stream"
`

You should see some random numbers being generated every second. If you look at the terminal you're running Quarkus in dev mode, you will see the same number being printed out.


## 6. Exploring the Code

### Examine and learn how we use Kafka in Quarkus

Navigate to the following directory using your prefered IDE:
Streaming-Data-with-Apache-Kafka-Quicklab/src/main/java/org/acme/WindGenerator.java

### Sending a periodic message

We're using the Reactive Messaging Flowable to send a random number every second to the windSpeedKph channel. We'll see below how we configure to send this to Kafka.


The @Outgoing annotation is used to mark this method as something that sends to the configured channel. This class is marked with the @ApplicationScoped annotation so that it can be used with Quarkus CDI. This also allows it to be discovered by the plumbing provided by the Quarkus Reactive Messaging Kafka plugin. We define that in the pom.xml like so:

`
  <dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-reactive-messaging-kafka</artifactId>
  </dependency>
`

### Receiving messages

Now open up WindSpeedConverter and have a look at the process method:

`

@Incoming("windSpeed")

@Outgoing("windSpeedMph")

@Broadcast

public double process(int windSpeedinKph) {

  return windSpeedinKph * KphToMph;

}

`




# Next Steps: 

- Try out the lab for 'Creating reactive microservices using MicroProfile reactive messaging' – https://labs.cognitiveclass.ai/tools/theiaopenshift/lab/tree?md_instructions_url=https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/creating-reactive-microservices-using-microprofile-reactive-messaging/instructions.md

- See instructions.md file for additional code content!

Thank you,

Sbusiso Mkhombe

Cloud Engineer, Hybrid Cloud Build Team

IBM Technology Sales

Sbusiso.Mkhombe@ibm.com 
