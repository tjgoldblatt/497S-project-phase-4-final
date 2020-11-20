## CS 497S Phase 4 - LinkUp
Overview: 
The goal of our project is to develop a website that enables users to make plans with their friend group by utilizing the Yelp API to give locations in a specific area. Our goal is to have a group of friends stored in a database, and when that group wants to meet up somewhere, they will use the Yelp API microservice by searching for an activity in a location, which will create an event in an event database that stores the name, group that created the event, date/time, location, and the type of activity the group wants to do. This will allow the friend groups to agree on a location, time, and place in one central location.

## Team Members:
Ron Arbo\
TJ Goldblatt\
Joe Pasquale\
Juelin Lui\
Nikil Thurai

## System Design:
Note: In our project repo's README, we have links to each of the READMEs for our microservices, with details such as service author, description of the service, and how to properly use the service functionality. So for service-specific descriptions, those are the best place to look.

Our overall system can be broken down into a few major components, namely our nginx router, flask services, and mongoDB databases.
Nginx serves on port 80, and takes in any requests that the outside world has for our application. It will then identify which service this request is intended for (by looking at the URL endpoint received), and send the request to that specific service. Our nginx.conf file specifies where each request type will be going, and specifies whether we serve our UI or send to a flask service.
Our flask services are designed to receive HTTP POST requests and respond with some sort of data relating to the application. Some services will simply respond with the results of a DB operation for that service’s object type, whereas other services (like the Yelp API service or Google Calendar service), will respond with other data to be used by the application. Other microservices like the Google Calendar API require user authentication in order to execute functions, so each call is individualized to each user, and scales with the amount of requests.
The Flask services responsible for managing objects like Groups and Events for our application will each have their own connection to a mongoDB database. These services will access the database for their own object through pymongo and a mongoDB Docker container. This way, we can access our DB through POST requests to the Flask application, while still keeping these services separate.
The JWT service generates an encrypted token for user authentication. When a user logs in successfully, a token that contains the user's relevant information will be sent back to the user in JSON format. The front-end on the user side will keep the token and put it in the headers. All subsequent requests from the front-end will carry this token. The other microservices can extract the token from the header of user requests and decrypt it. Note that the valid token can be generated only with a valid secret key(that is shared by different microservices and invisible to the public). In this way, different microservices can use the token to authenticate the user. This is a typical authentication method for microservices to maintain the stateless property of requests.
The frontend of our web application runs on port 3000 and solely communicates with the other microservices through NginX, this prevents the product from over complicating the overall system by only needing to communicate with one other microservice. It only worries about sending proper HTTP requests to the router which will handle the rest of the requests needed to complete that initial request from the frontend.
For scheduling we are running a python script which will take in a json object representation of a group and the group’s availability and from there will look for available times among the group. We also included multiple different formats to make it easier during testing for the group to use the scheduler. For scalability concerns this script is not resource intensive and can be used to fit any number of individuals or requests. The internal functionality takes in the json object and iterates through to find the top times that best work. This function can also be used in the testing phase. 
We connect all of our services through a common docker-compose file, which manages the common network on which all of our containerized services run on. This way, our Flask applications can recognize the DB instance they must use, Nginx knows which service to route URLs to, etc.

## Addressed Scalability:
Our team has addressed scalability issues by ensuring that all of our microservices are wrapped in docker containers. This allows us to treat each microservice as a blackbox, knowing that for example, when we pass in a request to the Yelp API, it “magically” returns the correct response, there's no need to worry about how it does it just that it does. By being able to treat each microservice as a blackbox, we’re preventing the need for future additions to our product to rely on understanding every microservice and how it functions. One microservice is its own world, with no ties to another microservice, thus allowing for maximum scalability.
Another benefit of using Docker containers is the existing software for their management. Applications like Kubernetes and DockerSwarm offer great ways for you to balance load and scale containers to more effectively address the load of your application. These applications make this process very convenient, requiring only an initial configuration from the user. After this, the application can monitor and manage your containers in a way that optimizes performance. 
Nginx simplifies the way we route and distribute requests received by the application. It provides a common entrypoint for any requests and will appropriately distribute them throughout the application. While doing this, Nginx also acts as a load balancer, making sure to not overflow any one service with too much traffic. This makes it ideal for the microservice architecture, as well as for scalability.
After research, we found mongoDB to be the best DB available for scalability. It also has a convenient image for Docker containers, making it easy to integrate into our project.
We found a software called SoapUI that we have been using as a load testing interface for our project. In this software, you can specify an endpoint that you’d like to call, and SoapUI will call for a set amount of times and give you statistics about response time like min, max, average, etc. This software has much more functionality than what we used it for, but it was perfect as a simple load tester that allowed us to check if our scalable systems were actually as such. 
uWSGI is our service for connecting Python to Nginx and it actually has great scalability functionality. In the uWSGI configuration file, you’re able to specify the amount of threads to be allocated to a particular service. We  conducted some tests, and found that increasing this number of threads actually helped the performance of our application. We tested our /group/read endpoint with our SoapUI load testing interface. We first tested with 2 threads for the groups service, and found an average time per request of 25.06 milliseconds. We tested the endpoint over the course of 1000 requests, and we did this 3 times. Then, we tested again except we used 8 threads. This time, we found an average time per request of 16.85 milliseconds. We tested three trials here as well, 2 of which were 1000 runs and one of 500 runs. So, we can conclude that this functionality of uWSGI can be used to scale larger systems for improved performance. Obviously it isn’t feasible to just increase the number of threads every time you want to make your system faster, but it’s safe to assume that, through testing, one can optimize their number of threads to yield both a fast and efficient final product.

## Personal Video Demonstrations:
TJ - https://www.youtube.com/watch?v=ubq48EbuDOo&feature=youtu.be \
Joe - https://youtu.be/I6b81Kp82wM \
Ron - https://youtu.be/KuTlRGPYYqA \
Juelin \
Nick

## Project Demonstration:
https://youtu.be/x7r8-5Y_M94 

## Link to Application:
http://ec2-54-86-209-172.compute-1.amazonaws.com/ 

