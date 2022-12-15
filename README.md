# COP5615_Twitter_Project 4.2

## Team Members

- Rishabh Srivastav : UFID 7659-9488
- Ashish Sunny Abraham : UFID 6388-7782

## Outline of the project

- The goal of the project is to implement a JSON based API clone which integrates all design features and functionalities of Twitter using Erlang’s Cowboy web framework. The client should be able to send and receive messages using web sockets and be able to perform various activities like creating an account, sending tweets, subscribe/follow another client, and perform various query activities like finding tweets with a specific hashtag or being able to view mentioned tweets. 

## Commands to run the program

- Compile all the .erl files in the zip folder (engine, user, test, server). 
- Open the erl shell and start the server by typing “server:start()”. This will execute the server instance at specified ipaddress and port.
- Activate the cowboy web server- %{start: {:ranch_listener_sup, :start_link, _opts}} = Plug.Cowboy.child_spec(scheme: :http, plug: MyPlug, options: [port: 8080])
- Server will begin to listen to requests from clients on the localhost:8080 port.
- In order to test each functionality manually, the user can refer to the screenshots attached in the features section to see how to test for each feature.

## Architecture
- The clients communicate with the server using JSON based API calls which have been integrated using web sockets. Cowboy framework was used to design the rest apis wherein upon a user registration, client will populate public-private keys (RSA 2048) for the user and delegate its public key to the server. Server will store public key of the client when it triggers a request for account activation. On a login attempt, cryptographic authentication is in place. The user will not be able to perform any of the functionalities without registering an account. The server then communicates with the worker nodes and assigns them tasks based on requests entered by the client. These workers then query the ets tables and databases and return the response in a JSON based format.
<img width="631" alt="Screenshot 2022-12-15 at 2 50 45 PM" src="https://user-images.githubusercontent.com/59756917/207953671-0990bf0a-7db2-4b1e-876c-4a4755faeb4e.png">


## Implementation Details
- Types of Requests: GET-POST-PUT 
- Rest controller path (separate for each type of API call) : /register,/login,/profile etc. 
- Response format: A JSON object is sent from the server to the client side via API calls which contain responses to the query which is then rendered at the client’s end.
- Example: A request to register an account would have the following JSON response:
{ “account_name”:”ashish”, “password”:”1234” }.
Similarly, the API response would be in the following JSON format for querying hashtags: { “tweetregid”:”1”, “content”:”#COP5615 is fun” }

- The GET calls will be responsible for fetching response from the server in a JSON object, which will be used to view tweets and for querying features (hashtags and mentions)
- The POST calls will be responsible for sending information to the server and interacting with the database (ETS tables) for activities like registering a user, tweeting some content etc. 
- The PUT calls refresh the server for keywords that were entered to query the system. Every time a user enters a new keyword to search for, the PUT helps update the previous state.

## Features implemented
- Register user: Allows the client to create an account on the server and register onto the application. The user won’t be able to access any of the features of the application without first creating an account. The RSA-2048 algorithm has been used for cryptographic encryption on the client side. 
  <img width="1512" alt="Screenshot 2022-12-15 at 12 39 18 PM" src="https://user-images.githubusercontent.com/59756917/207954878-dbf20a8a-18d2-4181-b7f3-047cd55e33d2.png">
  <img width="1512" alt="Screenshot 2022-12-15 at 12 46 37 PM" src="https://user-images.githubusercontent.com/59756917/207954888-9fc9ccbb-b82f-46de-93dd-db751ec3f7b7.png">

  - Creating an account: “Rishabh” and verifying that you can’t re-register if you have already registered
 
- Send tweet: Provides the client with the ability to publish posts by tweeting on their account. The created tweet is displayed in the live view of the user. The server assigns these tasks to worker nodes so that multiple users can tweet at once.

  <img width="1512" alt="Screenshot 2022-12-15 at 12 48 05 PM" src="https://user-images.githubusercontent.com/59756917/207955249-7462063b-8afc-45fb-ab35-a80ae50f8b33.png">
  <img width="1512" alt="Screenshot 2022-12-15 at 12 48 24 PM" src="https://user-images.githubusercontent.com/59756917/207955388-515c4995-5906-4b19-988a-c7243135e95c.png">
  
  - Sending tweets with the account: “Rishabh” user should register first before tweeting.

- Follow and subscribe: Enables clients to follow and subscribe to other clients so that they can view their tweets. These tweets then become part of the subscribed user’s tweet list as well.

  <img width="1512" alt="Screenshot 2022-12-15 at 12 52 09 PM" src="https://user-images.githubusercontent.com/59756917/207955691-a7bad0d2-b6cb-4bd4-9ca2-652b56b800be.png">
  <img width="1512" alt="Screenshot 2022-12-15 at 12 52 21 PM" src="https://user-images.githubusercontent.com/59756917/207955837-b687e896-be15-4ca0-b99a-72d614cc5793.png">
    
    - User “Ashish” is subscribing to User “Rishabh” and is now able to view all tweets of the user that he has subscribed to

  **Querying:** 
- Hashtags: Enables clients to search for particular tweets containing a keyword (hashtag) or a mention (@). The worker nodes scan through the ets tables and cowboy framework return a JSON object of the response that is rendered on the user interface of the application.
  
  <img width="1512" alt="Screenshot 2022-12-15 at 12 55 01 PM" src="https://user-images.githubusercontent.com/59756917/207956916-b684d372-f7fc-4153-a73b-764b98df0fcc.png">
  <img width="1512" alt="Screenshot 2022-12-15 at 12 55 15 PM" src="https://user-images.githubusercontent.com/59756917/207956948-a9239be4-a27a-4587-abfd-8da98eb39f33.png">

-  Mentions: Enables clients to search for particular tweets that contain name of the mentioned clients with the @ keyword. The worker nodes scan through the ets tables and cowboy framework return a JSON object of the response that is rendered on the user interface of the application.

  <img width="1512" alt="Screenshot 2022-12-15 at 12 56 05 PM" src="https://user-images.githubusercontent.com/59756917/207956256-ee9778ae-5061-4e68-a709-4a1dc309651b.png">
  <img width="1512" alt="Screenshot 2022-12-15 at 12 56 15 PM" src="https://user-images.githubusercontent.com/59756917/207956453-236667ed-2dfc-41d6-9d14-bf846156eb73.png">


## Module Specification Details
- engine.erl: This module is responsible for starting the application engine and acts as an interface in setting up connection between clients and the server. It communicates with the server using API calls and receives the JSON response object in return based on which it outputs the results to the client view. It also interacts with the clients registry ETS table, which keeps count of numerous worker nodes within the server, regulating process states.
- user.erl: This module entails information about the clients and their related information. This module sends requests to the server in exchange of awaiting response in the form of JSON object. The client side then filters out relevant information from the query and renders the output on the live view of the user. It also handles the client side cryptographic configuration while registering the user on the server.
- server.erl: This module establishes connection between client nodes and the database via the engine module. All the querying part is handled by the server upon performing lookup operations on the ets tables. It is also responsible for linking the backend code with the front-end part and link it with javascript code to enhance functionality of the application.

## Demo Link
- The link for the demo can be found at: https://youtu.be/4akNdl_FvCY



