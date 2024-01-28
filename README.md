### Task Description:
`restaurants.json` in the repository contains one hundred restaurants from the Helsinki area.

Task is to create an API endpoint /discovery that takes coordinates of the customer as an input and then returns a page (JSON response) containing most popular, newest and nearby restaurants (based on given coordinates).

Location of a customer needs to be provided as request parameters lat (latitude) and lon (longitude), e.g. /discovery?lat=60.1709&lon=24.941. Both parameters accept float values.

An JSON object returned by the /discovery -endpoint has the following structure:

```
{
   "sections": [
      {
           "title": "Popular Restaurants",
           "restaurants": [.. add max 10 restaurant objects..]
      },
      {
           "title": "New Restaurants",
           "restaurants": [..add max 10 restaurant objects..]
      },
 	{
           "title": "Nearby Restaurants",
           "restaurants": [.. add max 10 restaurant objects..]
      }

   ]
}
```

For each restaurants-list, there are maximum 10 restaurant objects returned. A list can also contain fewer restaurants (or even be empty) if there are not enough objects matching given conditions. A section with an empty restaurants-list is removed from the response.

There are two main rules followed:

All restaurants returned by the endpoint are closer than 1.5 kilometers from given coordinates, measured as a straight line between coordinates and the location of the restaurant.
Open restaurants (online=true) are more important than closed ones. Every list is first populated with open restaurants, and closed ones are added only if there is still capacity left.
In addition each list has a specific sorting rule:

“Popular Restaurants”: highest popularity value first (descending order)
“New Restaurants”: Newest launch_date first (descending). This list has also a special rule: launch_date must be no older than 4 months.
“Nearby Restaurants”: Closest to the given location first (ascending).

The same restaurant can obviously be in multiple lists (if it matches given criteria).

----------

This file is divided into 2 main sections:  
The first section deals with the details of my work, whereas the second section deals with the 'HOW' to get the code up and running.

**Important Note**: Since the earth is (almost a) sphere, the distances I have computed (or rather had MongoDB compute them) are in a manner of how an actual GPS would compute the straight line distance, and not with the simple 2-D distance formula. To implement this feature, I have used a mongoengine functionality which finds documents with _location_ (PointField) values within a certain distance and sorts them in ascending order. You can verify that the distances computed are correctly sorted using [this website](https://gps-coordinates.org/distance-between-coordinates.php).

### Section 1 - The Work's Details

In an overview:
* I have utilized the Flask framework for the assignment.
* Instead of directly working with the _restaurants.json_ file, I have used MongoDB, whose collection is populated with the data from _restaurants.json_ file when the application starts.
* The Flask app and MongoDB are implemented as microservices (namely _backend_ and _mongo_ respectively), with _backend_ being dependent on _mongo_.
* MongoEngine is the Document-Object Mapper I have used for my purpose.
* Based on the problem statement and sorting rules, appropriate logic has been implemented, and a JSON output, with the same format given in _discovery_page.json_, is computed.  

##### 1. _db.py_ contents
This file contains creation of a MongoEngine object and a function definition for initializing the database.

##### 2. _model.py_ contents
The document's schema is defined in this file. _launch_date_ and _location_ are defined as DateTimeField and PointField formats respectively for easy manipulation.

##### 3. _app.py_ contents
This is the main application code file, wherein the following tasks are carried out:
1. Flask application instance is created.
2. Database is configured and initialized.
3. Collection is populated using the _restaurants.json_ file.
4. API endpoint implementing the required logic and output is written.

To understand the details, please refer to the code which is commented enough to understand why and what is being done.

##### 4. _Dockerfile_ contents
The following is being done:
1. A Python image is pulled.
2. The Working directory is set to `/usr/app`
3. All the contents from the project folder are copied to the working directory.
4. Dependencies in the `requirements.txt` are installed.
5. The port number 5000 is exposed.
6. Environment variable `FLASK_APP` is set to `app.py`.
7. The command `flask run --host=0.0.0.0` is implemented to start the flask app.

##### 5. _docker-compose.yaml_ contents
There are two services configured here - _backend_ and _mongo_.

A container `flaskbackend` is to be created using `backend`'s configuration. This container has `HOST_PORT` as 5000 and `CONTAINER_PORT` as 5000. The `backend` service, however depends on `mongo` service. The `mongo` service is configured with the container name as `mongo`, which is also to join the default network. This container has `HOST_PORT` as 1048 and `CONTAINER_PORT` as 27017.  

##### 6. _requirements.txt_ contents
This files includes all the modules and the fixed version dependencies

### Section 2 - How to get the code up and running?

The four components for this part are:
1. The Submitted (Compressed) Folder
2. Docker
3. Docker Compose
4. Postman  

In case the device on which my code is being tested on does not have any/all of the above mentioned components, the following hyperlinks can be utilized:
* Windows: (Docker Compose ships with Docker Desktop)
  * [Install Docker Desktop on Windows](https://docs.docker.com/docker-for-windows/install/)
  * [Install Postman on Windows](https://www.postman.com/downloads/)
* Mac: (Docker Compose ships with Docker Desktop)
  * [Install Docker Desktop on Mac](https://docs.docker.com/docker-for-mac/install/)
  * [Install Postman on Mac](https://www.postman.com/downloads/)
* Linux:
  * [Install Docker Engine](https://docs.docker.com/engine/install/)
  * [Install Docker Compose](https://docs.docker.com/compose/install/)
  * [Install Postman on Linux(x64)](https://www.postman.com/downloads/)

Once the required software tools are present, the following needs to be carried out:
1. Extract the compressed folder submitted by me to a location of your choice.
2. Depending on the OS you are working on, open the respective command line interface.
3. On the command line interface, navigate to the extracted folder.
4. Run the command `docker-compose up`. After a while, the services get started. Something similar to `flaskbackend |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)` will be prompted on the command line interface.
5. Open the Postman app. With request type set to **GET**, type the URL in the format `http://localhost:5000/discovery?lat=xxxx&lon=xxxx`, where `xxxx` denotes any floating point number.  Hit **Send**, and you can view the output below. Make sure to select the _Pretty_ and _JSON_ options to view the output clearly. If the request is proper, the response status will be 200 and you can view the results. But if the URL is a bad request i.e. if _lat_ and/or _lon_ are mispelled/non-existent, and/or the `xxxx` value(s) are not floating point numbers, and/or `xxxx` is out of bound value(s) for latitude and/or longitude, a JSON error message with response code 400 will be displayed.
