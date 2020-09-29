# Url-Shortening

## Start

`Step 1 clone application` ->   `git clone https://github.com/MrGrigoryAlexandrovich/Url-Shortening.git`

`Step 2 init submodule ` ->  `git submodule init` (in application folder)

`Step 3 update submodule ` ->  `git submodule update` (in application folder)

`Step 4 Building containers` ->  `docker-compose build`

`Step 5 Start the server` ->  `docker-compose up`

## Description

Url-Shortening is an application made up of two separate services. The first service is management and the second is redirection. Between services there is a rabbitmq broker message. Management service is the producer, and redirection is the consumer. The message is a array in the format [Created/Deleted, id, realURL, shortURL]. If user enters a real valid URL and call routes create, the management service generates a unique short URL.  After that, the data is stored in the MYSQL database, but also sent to rabbitmq. On the other side, the redirection service receives a message from rabbitmq and saves it in the Redis - in-memory data structure store. If the user wants to delete a short URL, call the delete route with a valid id. As in the case of creating messages, message is sent to rabbitmq from where the consumer receives it. Data is automatically deleted from both - > the MySQL database and the Redis. Redis key is  based on hash part of the short URL. Management service starts on port 8000 and redirection on 8001. Both services have their own config file that can be easily encrypted with the transcrypt.To make it easier to monitor the MYSQL database when starting the server, phpmyadmin is also started on port 8080. Username - root, password - password

## Management service

The management service uses a MYSQL database to store and delete data.
Classic row queries were used to communicate with the database. Queries are stored in one class and are called in routes with certain parameters depending on whether we want to create or delete data. The management service has two routes, one to create a short URL and the other to delete from the database

## Management service routes

`/api/routes`

`/create`  expects an object in the body of the request {"realURL":string} (POST request)

Valid request gets response with status 201 and response data in json (id, realURL, shortURL). Invalid request -> response is status 400

`/delete/:id` expects a valid ID(number) in the parameter returns 200 if exist - 404 not exist (DELETE request)

## Redirection service 

The Redirection service receives messages from rabbitmq and saves or deletes data from Redis depending on the case. The essence of the service itself as the name says is redirect to the realURL using a short URL. We do this by calling the check route with a parameter that is hashed part of shortURL. A rate limiter has been implemented that does not allow more than 10 redirect in a period of 120 seconds to a specific URL. Redis was used for the rate limit system. The redis functions apend, incr, and expire were used. Append to first request, incr from second to 9 and expire for each request with key expiration after 120 seconds. 

## Redirection service route

`/api/routes`

`/check/:shortURL`  expects an string in the parameter of the request (GET request)

## Author

Ahmed Cvrčak (MrGrigoryAlexandrovich)







