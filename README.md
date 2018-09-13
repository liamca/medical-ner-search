# Medical Document Search [Draft]
Leveraging Apache cTAKES and Azure Cognitive Search to Build and Medical Search App

The goal of this project is to show how to leverage [Apache cTAKES](http://ctakes.apache.org/) along with [Azure Cognitive Search](https://azure.microsoft.com/en-us/blog/announcing-cognitive-search-azure-search-cognitive-capabilities/) to build an effective document search application.  

One of the most important things in building an effective search application is to have as much metadata about the content as possible.  Unfortunately with medical documents, this is typically a very unstructured piece of content where you usually only have the block of text from the content.  For that reason, it is very important to "enrich" this content by analyzing it to extract meaningful metadata about the content, such as what diseases were mentioned, or what parts of the anatomy were discussed.  By doing this, you start to put structure to your content which greatly helps what you can do with a search based application.

Using this technique, you can build applications such as this [PubMed Search Demo](http://webmedsearch.azurewebsites.net)

![Medical NER Search Demo of PubMed](https://raw.githubusercontent.com/liamca/medical-ner-search/master/pubmed_search_demo.png)

as well as create graph visualizations that shows correlations such as: 
![Medical NER Search Graph Demo of PubMed](https://raw.githubusercontent.com/liamca/medical-ner-search/master/medical_search_graph.png)

This tutorial assumes that you will be leveraging Windows.

## Overview of What this Tutorial Covers
In this tutorial, we will walk through the following steps:
1. Configure Apache cTAKES as a service to receive text and return medical entities
2. Configure Azure Cognitive Search to ingest unstructured medical content (PDF's) from Azure Blob Storage into a search index that has important metadata as well as extracted text from the content
3. Run an application that processes these PDF's against the Apace cTAKES service to extract medical entities and apply that to the Azure Search index
4. Run an application that allows for searching and exploration of this medical content

## What is Apache cTAKES
cTAKES stands for clinical text analytics and knowledge extraction and is a very effective method of extracting various medical based entities (named entity extraction) from a corpus of text.  For example it can identify disease or anatomical terms mentioned within the text.  It has been trained using dataources such as [UMLS (snomed)](https://www.nlm.nih.gov/healthit/snomedct/).

## What is Azure Cognitive Search
[Azure Cognitive Search](https://azure.microsoft.com/en-us/blog/announcing-cognitive-search-azure-search-cognitive-capabilities/) is a merger of [Azure Search](https://azure.microsoft.com/en-us/services/search/) which is a platform as a service that makes it easy for developers to build great search experiences over their data with [Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/) which has the ability to leverage AI extract meaning from content such as text and images.  By combining these technologies, Cognitive Search has the ability to let developer build effective search applications over unstructured content such as medical documents.

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

If everything runs properly, you should see something like the following: 
> [INFO] Started ServerConnector@3cf55e0c{HTTP/1.1}{0.0.0.0:8080}

> [INFO] Started @38253ms

> [INFO] Started Jetty Server

Open a browser and point it to http://localhost:8080/ and then enter some medical content to test the results such as: 
> Morquio syndrome (referred to as mucopolysaccharidosis IV, MPS IV, Morquio-Brailsford syndrome, or Morquio) is a rare metabolic disorder in which the body cannot process certain types of mucopolysaccharides (long chains of sugar molecules), which the body uses as lubricants and shock absorbers.

The results should look as follows:
![Medical NER Example using cTAKES](https://raw.githubusercontent.com/liamca/medical-ner-search/master/medical_ner_example.png)

## Setting up Azure Search
Now that we have a service that can extract medical entities from text, we need to create an Azure Search service that will make this content searchable to enable the types of applications show at the top.  To learn more about how to do this, please visit [Create an Azure Search service in the portal](https://docs.microsoft.com/en-us/azure/search/search-create-service-portal).  For our demo purposes, you can create a [Free Azure Search Service](https://docs.microsoft.com/en-us/azure/search/search-what-is-azure-search#free-trial).

Once you have created the Azure Search service, you will need to get your Azure Search Service name as you specified in [this step](https://docs.microsoft.com/en-us/azure/search/search-create-service-portal#name-the-service-and-url-endpoint) as well as the [Admin API Key](https://docs.microsoft.com/en-us/azure/search/search-create-index-dotnet#identify-your-azure-search-services-admin-api-key).

## Extracting Medical Entities from Content and Ingesting into Azure Search
The next step will be to take some content and first run it through cTAKES to get the metadata and then load the resulting content into an [Azure Search Index](https://docs.microsoft.com/en-us/azure/search/search-create-index-dotnet).  For this demo, we will keep it simple and use some text files that have medical content in them.  In many cases, the content exists in file formats such as PDF and Office.  This demo does not show how to process this type of content, however [Apache Tika](https://tika.apache.org/) is an excellent tool for extracting text from these file types and you can see some .NET demo code on how to do this [text extraction here](https://github.com/liamca/AzureSearch-AzureFunctions-CognitiveServices/blob/master/ApacheTika/run.csx).

### Running the Demo Code
NOTE: This step requires you to have a valid UMLS account as outlined above.

This demo only extracts a few of the possible medical entity types that cTAKES supports including:
* Disease and Disorders
* Medication Mentions
* Sign and Symptom 
* Anatomical Sites

Other entity types available can be [found here](http://ctakes.apache.org/apidocs/trunk/org/apache/ctakes/typesystem/type/textsem/EventMention.html)

To get started, 
1. Open the MedicalEntityExtraction console application solution in Visual Studio. 
2. Open the MedicalEntityExtraction project and then open the project file Program.cs.
3. Update umlsuser and umlspw values to those obtained from setting up your UMLS account
