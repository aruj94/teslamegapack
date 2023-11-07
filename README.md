This is a dummy API for testing GET, POST and DELETE functionalities. This API uses Node js, Express, MongoDb cluster and cloud Redis to carry out its functions. It also uses Jenkins to automate docker image creation, pushing image to google cloud artifact registry and creating a GKE auto-cluster using that image. Kubernetes secrets are used to store .env variables.

Fundamental purpose of this API is to check the post requests for device temperature data. If the data string is incorrectly formatted, it is stored in a mongodb atlas cluster for future retreival. The GET /errors end point is used to retrieve all the errors saved in the mongodb database and displayed in a specific format to the user - {"errors": {error_string_1, error_string2...}}. The DELETE /errors end point is used to delete all the data stored in the mongodb collection.

Middleware is used in the routes to check for the API_KEY in the request headers for authentication purposes. Without the API_KEY, middleware authentication will fail.

To send any request to the API you will need to send in the API key in the headers of your request with a {key, value} pair of {authorization, API_KEY}. Otherwise neither of the post, delete or get functions will work. If you dont format the header with the appropriate key, value pair you will recieve a {"error": "Unauthorized: Invalid API key provided"} with 401 status code.

The API has 3 end points which are POST at '/temp', GET at '/errors' and DELETE at '/errors'. Also one GET at '/' which sends a welcome message.

Here is a summary on endpoints-

POST /temp- The API only supports JSON format requests otherwise you will recieve a { "error": 'Unsupported Request Type' } with 415 status code. Following is some explanation- {“data”: data_string} data_string needs to be of the following format- a: b:'Temperature':c. Here, a and b are int and c is a float. Ideally temperature should be less 90.00. If it is greater than 90.00 you will recieve an overtemp response

example request - {"data":"3659:164099:'Temperature':45.00"} In the above example you will recieve {"overtemp": false} with status 200. If temperatur was above or equal to 90.00 you would've recieved the following response- { "overtemp": true, "device_id": "3659", "formatted_time": "yyyy/mm/dd h:m :s" } In case you have badly formatted data, you will recieve the following responses- {"error": "bad request"} with 400 status code This request's data_string will be written a mongodb database for future retrieval.

GET /errors- This endpoint does not require any request bodu but you will need the {authorization, API_KEY} pair in you request headers for successful retrieval. This endpoint will give you a JSON file with the list of all badly formatted data_string. Like the following- {"errors": ["365951380:1640995229697:'Temperature':45"]} with status code of 200

DELETE /errors- This endpoint does not require any request body but you will need the {authorization, API_KEY} pair in you request headers for successful retrieval. This request will delete all the stored error data from mongodb and once completed will send you a success message in JSON format- {"message": "Error buffer cleared successfully"} with status code of 200

A couple of unit tests have been setup to test routes and middleware operations.

A jenkins pipeline has been created as well which creates a docker image, pushes that image to google cloud artifact registry and then creates a GKE cluster. Kubernestes secrets are used to store .env variables. Jenkins secrets are also used google cloud registry URL, google cloud project Id.