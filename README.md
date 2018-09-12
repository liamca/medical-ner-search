# Medical Document Search [Draft]
Leveraging Apache cTAKES and Azure Search to Build and Medical Search App

The goal of this project is to show how to leverage [Apache cTAKES](http://ctakes.apache.org/) along with [Azure Search](https://azure.microsoft.com/en-us/services/search/) to build an effective document search application.  

This tutorial assumes that you will be leveraging Windows.

## What is Apache CTakes
cTAKES stands for clinical text analytics and knowledge extraction and is a very effective method of extracting various medical based entities (named entity extraction) from a corpus of text.  For example it can identify disease or anatomical terms mentioned within the text.  It has been trained using dataources such as [UMLS (snomed)](https://www.nlm.nih.gov/healthit/snomedct/).

## What is Azure Search
Azure Search is a platform as a service that makes it easy for developers to build great search experiences over their data.  

## Getting Started with cTAKES
In this section we will learn how to download and run a JAVA servlet based implimentation of Apache cTAKES that has the ability to receive through an API call a set of text that is processed and returns a set of medical named entities found in the content.

### Setup UMLS Account
Since cTAKES leverages UMLS, we will need to sign up for a UMLS account.  To do this visit the [UMSL](https://www.nlm.nih.gov/research/umls/) page and choose Sign Up.  Make note of the license restrictions of using this and the fact that the approval is not instantaneous.

### Download and Configure cTAKES Web App
We will be using healthnlp's implementation which can be found [here](https://github.com/healthnlp/examples).  Specifically, we will be using the [ctakes-web-client](https://github.com/healthnlp/examples/tree/master/ctakes-web-client).  You can see an example of what the running web app will look like in [this demo](http://54.68.117.30:8080/index.jsp).

To get this running, you will not only need to download this project, but also install Maven and JAVA.  In my case, I have:
* Java JDK 1.8.0
* Maven 3.5.3

After installing this, from my machine I configured JAVA and Maven as follows:

> set JAVA_HOME=d:\ctakes\jdk1.8.0_152
> set MAVEN_HOME=d:\ctakes\apache-maven-3.5.3
> set M2_HOME=d:\ctakes\apache-maven-3.5.3
> set path=%path%;d:\ctakes\apache-maven-3.5.3\bin;d:\ctakes\jdk1.8.0_152\bin

After this point I could change to the \examples-master\ctakes-web-client directory and run:
> mvn jetty:run

If everything runs properly, you should see the following: 
>
