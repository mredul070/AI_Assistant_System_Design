 # AI Assistant [Chat Bot] Solution Architecture

 ## Assumptions
 - Let's assume this chatbot will be developed by booking.com owners and they want to integrate this chatbot in their website and mobile applications. 
 - Let's assume booking.com uses AWS for their DevOps tasks, so I will try to mention the general services but will provide examples with respect AWS, as I have prior knowlede with AWS. 


 ## Consideration 
 - In a world where most popular choice is microservice and modularty, let's build a solution where this chatbot will a whole different module in the overall system of booking.com
 - As this will be used for booking.com's related services, it is not required to train a Large Language Model(LLM) model this task. Rather we could use paid LLM's API like OpenAI's GPT-3 etc. Also, preparing data for LLM is very difficult and time consuming task. 
 - Hotel booking and recommendations peaks at different times of the year, thus the load in the server will not be constant the whole time, thus we need system architecture automatically scales up and down based on the load. 
- Booking.com must ensure user login before using any feature on the site specially the chatbot to keep track of the user data, also for safety and security of the user data.
****
# The high Level Architecture
Our Solution should consist of the following modules 
 ## Overall system
 ### - User inferface
 - #### Web application 
 - #### Mobile application
 ### - Router/API Gateway
 *This module will decide where to send an API call based on request from frontend applications.*
 ### - Backend Services
 *The back end server. **Let's assume booking.com follows an microservice architecture.***
 - #### Authentication System
 - #### Tour booking and reccomendation Services
 - #### Payment Module
 - #### Other service modules based booking.com's services
 ### - ChatBot Module
 *This is our targeted module for this task.*
 - #### NLU(Natural Language Understanding) Module
 - #### Intent Recognition/Sentiment Analysis
 - #### Knowledge Graph
 - #### NLG(Natural Language Generation) Module
 - #### NER(Named Entity Recogniton) Module
 - #### Query Generator
 - #### Recommendation Module
 - #### Performance and Log Module
 ### - Database/Storage Systems
 *There could be multiple databases based on the overall architecture. For this tasks we will focus on the databases that will require to build the chatbot.*
 ### - Data Extraction Module 
 *This module is reuired for analysis various matrices for business needs.*
 ![System Architecture Image](/images/high_level_system_architecture.png)
****
# The Chat Bot Module

Introduction to Different Modules inside the chat bot

- ## NLU(Natural Language Understanding) Module
*This is actually consists of few other module that I have mentioned in the high level architecture.*

The NLU module processes user queries and extracts relevant information using natural language processing techniques.It identifies the intent behind the query (e.g., find tours, book accommodation) and extracts entities (e.g., location, dates, budget, preferences). The NLU module utilizes a trained model and a rule-based system to understand user input accurately. **So the main task of this module is to decide how to provide answer for a specific message, should it give a result from fixed knowledge or generate answer from trained model or genrate query to call an API.**
So this bigger module consists of the following module:

- ### Intent Recognition/Sentiment Analysis
This module takes an user message as an input and tried to understand what is ther user's intention or user's sentiment in the message. This is very important for the chatbot to unserstand as in how should the chatbot replay to a certain user message based on his intention/emotion.

- #### NER(Named Entity Recogniton) Module
This module extracts named for entity like for this scenerio a hotel name, location name, area name, date etc. which will be required for hotel booking or recommendation from a user message.

- ## Knowledge Graph
There should be fixed answer to fixed answers. This knowledge graphs keeps a record of these questions. This is because if there a fixed to supposed question we do need to turned to deep learning model for each questions. This takes an question as input and provides the answer as the output.

- ## NLG(Natural Language Generation) Module
So this is the conversational AI model which answer questions based on the user question. We need regularly train and update this model for better performance. This model provides an answer based on the whole conversion. We can use the pre-trained gpt-3 model, using our custom data we train this model for this case. Because in this website user will ask question only regarding the services of the booking.com.

- ## Recommendation Module
This module generates recommendation based users past booking information and currently provided information. This needs to done using transdormer or attention based model. 

- ## Query Generator 
This module is directly connected with the booking and recommendation module. So when user request for booking of recommendation this module generate an api call based on the information provided to book a hotel or ask for recommendation. This will also able to decide whether it needs more information to make that API call. This could be done using rule based approach.

- ## Performance and Log Monitoring Module
This module will keep track of errors and model performances so that in future the performance of these models can be improved. 

***As our chatbot module is directly connected with booking and recommendation module we should descrive the functionality of this module too.***

- ## Booking and recommendation Module
This module is directly connected with Database which store user's booking and recommendation information. Also, provide API to store booking and recommendation information. 

## How the Chatbot Module works

So when an API gateway detect an API call for the chatbot, A session Manager is called which keeps track of each chat session then sends the request to the chatbot module with a session ID.

Then input message get processed, first the message is sent NLU module to extract the intent, sentiment, names intentities etc. to get the question type. Based on the question type and other information the controller decides from which module the output is requested. 

- If is a known fixed FAQ type question then it sends the message to the knowledge graph module.
- If the message intent is to ask for a recommendation then the controller sends the message to the recommendation module.
- If the NLU message isn't from the abobe two, it sends the message to the NLG module which is the conversational AI. **The controller provides the whole conversation in that session to request a answer for this scenerio.**
- If the message intent is to ask for booking then it sends the message to query selector module. If the query selector finds out there isn't sufficient information then the controller ask generate response to ask the user to provide the additional information.

The controller will save the responses and model performance to required places so that developers can maintain, upgrade and measure performanace of the chatbot. 

There will be a database required to keep the logs of the performance and users preferences.

This is how the chatbot module should work for each individual messages. 

****

# System Architecture

*We have already assumed the developed system will be deplopyed in AWS. The following will be mostly based on AWS's services.*

For API gateway we can use AWS APIgate way and there session manager service by which we can manage session and session ID as well.

We can use  AWS DynamoDB for models internal usecases that we have stated earlier. Also to create the knowledge graph.

Model logs will be monitored in AWS cloudwatch and can be saved in S3 buckets for debugging.

Finally for LLM/gpt-3 we can use AWS's in house service AWS LEX. 

But you need more accuracy and performance in house developed of GPT-3 based model in suggested which in deployed inside a Lambda Function.

## AWS Lambda based Mircroservice Approach 
- As stated in our solution we will be using multiple modules for various tasks. We will be needing various models in the chat bot. Like sentiment analysis and NER are two different models. In this approach we develop each model seperately train in locally and write down inference pipeline in a docker. 
- Then upload the Docker images in the AWS Elastic Container Registry(ECR) from which we will directly create and update the lambda function.
- We will upload the base model in S3 buckets and maintain model thropugh there.
- As shown in diagram there will multiple lambda function for various tasks.

### Benefits
- As module can be maintained and used seperately. Like you only need sentiment analysis you can use that part only. Thus cost reduction can be achieved.
- Using Lambda function ensure high availability of the systems.
- The deployed system will be fast as model inference can be done parallely. Not a single will be depended on the other to be finished. 
- System will **automatically scale up and down** when required.


### Disadvantages
- System looks very complex to maintain and update.
- Multiple places to control a model.
- Maintanence cost will be high.

## ECS Bases Monolithic Approach
- We can package the whole system in flask or django server inside a docker images.
- The upload the image to ECR and deploy the system in **ECS fargate**. 
- Models will be uploaded and maintained through S3.

### Benefits
- Single point of control
- Simplicity in maintanance 
- **Scales up and down when the load varies.**

### Disadvantages
- Different libraries may require various version of the sane library to run, this wil create problems during developed
- Whole system will be running for one model inference thus system runtime will increase
- Can't use a just single module if required as the whole system is interconnected.

*We can deploy the services in other cloud services using the similar types of services.*

*But in house server can also be used, need to use load balancer in that case to cope with the various load in the system.*
