# InstaStore Reverse

In this test you are going to fix some problems happening in this API. You can have a reference of the outputs of the API in https://tranquil-atoll-58683.herokuapp.com/ . Please follow the [instructions](#instructions) below, read the issues that you need to solve, and use the [documentation](#documentation) as a guide.

## Instructions
1. You would need to solve the [issues](#issues) reported by our clients, and write a proper answer to them in a separate mardown file.
2. Please clone/fork this repo privately, and once you finished your work please create a PR and mark @davidcp90 as reviewer.
2. You would need to deploy the service on the internet
3. Please make a small webpage to test the closest store endpoint . Be creative and lean, it doesn't need to be something complex, maybe just something that could help to diagnose the issues in the future with a rapid test.
4. You would have 2 days to solve this challenge. Be aware that you are solving issues happening in production, so having the issues solved as fast as you can would give you extra points.

## Issues

#### Laureano Godinez - Junior Support Analyst HEB Contry reported:
> Los usuarios de HEB reportan que no encuentran
> la tienda de HEB Contry, solicitamos su ayuda
> de manera inmediata para lidiar con este problema.
> El equipo de TI de HEB dice que es culpa de su API.
> Favor arreglar de inmediato.

#### Harshit Patel - Sr Back-end Engineer Falabella India reported:
> Hi Instaleap Team,
> Falabella India team 'd require credentials to
> integrate the closest store endpoint for the project.
> Also, we require to create the following stores on your system
> Tottus Puruchuco
> Tottus Bellavista
> Tottus Cajamarca
> Tottus Quilca
> You can extract their details here https://www.tottus.com.pe/tottus/storelocations

### Samantha Lopez - Account Manager Instaleap reported:
> Hola, me reportan por whatsapp que en HEB no
> pueden obtener tiendas, les da error 500
> It was not possible to place the order, please try again later
> Por favor indicar tiempo de resolución y 
> garantizar que todo funciona. 


## Documentation

The team made this to help you understand the service.

## Requirements
InstaStore is a microservice in charge of selecting the closest "convenience" store to deliver a groceries order to our B2B clients.

### Non-functional
- We expect you to deliver idiomatic code in a way that is easy to read and follows the accepted guidelines in your area of expertise.
- You should write it on Node.js with Express.js. Libraries, transpilers , etc are up to you.
- Endpoints are fast (less than 300ms).
- Endpoints respond error codes that makes sense to the case.
- Please provide documentation for the endpoints you create.

### Functional (user stories)
1. Our B2B clients should be able to consume an endpoint that provides them the following information:
  - storeId
  - storeName
  - isOpen
  - coordinates
  - nextDeliveryTime
2. The endpoint returns the closest store available
3. We need to keep track of each call to the endpoint

## Common questions


- What are the expected request parameters? What is the meaning of each parameter?

  You should expect to receive a json object with the following parameters

  ```json
  "destination": {
    "name": "string”, // Name of the address given by user (required)
    "address": "string”, // Address captured (required)
    "address_two": "string”, //Additional details for the address (line apt, house number, etc)
    "description": "string”, //Instructions for the delivery
    "country": "string”, // Country code according to ISO-3166-1 (required)
    "city": "string", // City name
    "state": "string", // State name
    "zip_code": "string",
    "latitude": 0, //number indicating the latitude of the address provided
    "longitude": 0, //number indicating the longitude of the address provided
  }
  ```

- What is nextDeliveryTime?

  In this case, we are going to consider that each store can deliver 3 orders per hour, and according to that, you need to provide the next time (date+hour) in which the store can deliver an order (nextDeliveryTime).

- In the API response we have 'isOpen', What is the role of this field in determining the closest available store?

  Nothing. I was expecting you to catch things unneeded.

- What are the locations/coordinates of the stores?

  I am attaching a CSV with that information, you can discard any column that you think is not needed.

- What should be the criteria for defining the closest store?

  This has to be one of your implementations decisions, choose the best one according to the time given and the data provided for the challenge. Make sure to justify it, according to point 5 in the repo.

##  Design and Implementation choices


1. Convenience stores can deliver 3 orders per hour,  I'm going to assume that each store can process an order every 20 minutes.
2. The convenience stores are open 24 hours/7 days.
3. You have to be authenticated to consume the endpoint.


### Criteria for defining the closest store available

The closest store would be the one from which the delivery will arrive first, to achieve that, we're are going use the estimated time of arrival (ETA)  defined as the sum of the nextDeliveryTime and the duration of the shipping (shippingTime).

ETA = nextDeliveryTime + shippingTime

We'll select the store with the minimal ETA.

### Architecture

![arch](https://github.com/mercadoni/instastore-reverse/raw/dev/arch.png)

### Database

![db](https://github.com/mercadoni/instastore-reverse/raw/dev/db.png)

### Security

JWT is going to be used to protect the /store/ endpoint from unauthorized use.

The endpoint /user/ will provide a way to login and obtain the token needed to use /store/.

### Tracking

All the calls made to the endpoints would be logged using the library [morgan](https://github.com/expressjs/morgan). In addition to logging, the orders are going to be saved in the database so in further developments we can obtain interesting metrics such as:

- Busiest day periods.
- Zones of most active users.

### Deployment

To deploy locally with env params set(api keys, etc..):

1. Clone this repository
2. npm run seed
3. npm start

To deploy locally with the test env (assumes mongodb is running at localhost, geodecoding will not work if you don't set the api key):

1. Clone this repository
2. npm run seed:local
3. NODE_ENV=test npm start

### Documentation

The API documentation was developed following the OpenAPI Specification v3

### Testing

The API has unit tests, linting and code coverage of at least 93%. 

#### Register

```curl
curl --location --request POST 'https://tranquil-atoll-58683.herokuapp.com/user/register' \
--header 'Content-Type: application/json' \
--data-raw '{
	"username" : "<username>",
	"password": "<password>",
	"password2": "<password>"
}'
```

#### Login

```curl
curl --location --request POST 'https://tranquil-atoll-58683.herokuapp.com/user/login' \
--header 'Content-Type: application/json' \
--data-raw '{
	"username" : "<username>",
	"password": "<password>"
}'
```

#### get closest store available
```curl
curl --location --request POST 'https://tranquil-atoll-58683.herokuapp.com/store/order' \
--header 'Authorization: Bearer <AUTH_TOKEN>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "Casa",
  "address": "Gustavo A. Madero Oriente 223",
  "address_two": "apt 205",
  "description": "knock 3 times",
  "country": "México",
  "city": "Monterrey",
  "state": "N.L.",
  "zip_code": "64570",
  "latitude": 25.691053,
  "longitude": -100.310530
}'
```