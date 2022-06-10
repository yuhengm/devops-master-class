### Microservices

These simple microservices enable us to **focus on** learning the tools - Docker, Kubernetes, CI, CD and build the infrastructure needed around typical microservices.

### Currency Exchange Service

If you ask it the value of 1 USD in INR, or 1 Australian Dollar in INR, the Currency Exchange Service answers

- 1 USD is 60 INR
- 1 Australian Dollars is 50 INR.

http://localhost:8000/currency-exchange/from/EUR/to/INR

```json
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75.0,
  "exchangeEnvironmentInfo": "37f1ad927c6e v1 27c6e"
}
```

### Currency Conversion

Currency Conversion Service is used to convert a bucket of currencies. If you want to find the value of 10 USD, Currency Conversion Service returns 600.

- **STEP 1** : Currency Conversion Service calls the Currency Exchange Service for the value of 1 USD. It gets a response back saying 60.
- **STEP 2** : The Currency Conversion Service then multiplies 10 by 60, and returns 600 back.

http://localhost:8100/currency-conversion/from/EUR/to/INR/quantity/10

```json
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75.0,
  "quantity": 10,
  "totalCalculatedAmount": 750.0,
  "exchangeEnvironmentInfo": "37f1ad927c6e v1 27c6e",
  "conversionEnvironmentInfo": "fb6316b5713d v1 5713d"
}
```

### How does Currency Conversion know the location of Currency Exchange?

1. Use Docker Link

   - You don't want to HARDCODE
   - Using Docker Link to Connect Microservices.
     ```shell
     % docker run -d -p 8000:8000 --name=currency-exchange in28min/currency-exchange:0.0.1-RELEASE
     % docker run -d -p 8100:8100 --name=currency-conversion --link currency-exchange in28min/currency-conversion:0.0.1-RELEASE
     ```
   - Once the link is established, we have a new url available for currency-conversion service.
   - Configure an Environment Variable - `CURRENCY_EXCHANGE_SERVICE_HOST`
   - --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange

2. Use Custom Networking

   - Create network and launch services as part of the network
     ```shell
     % docker create currency-network
     % docker run -d -p 8000:8000 --name=currency-exchange  --network=currency-network in28min/currency-exchange:0.0.1-RELEASE
     % docker run -d -p 8100:8100 --env CURRENCY_EXCHANGE_SERVICE_HOST=http;//currency-exchange --name=currency-conversion --network currency-network in28min/currency-conversion:0.0.1-RELEASE
     ```

3. Use Docker Compose
   - Create a **`docker-compose.yml`** file to specify images and networks
   - Run with the following script
     ```shell
     cd DIRECTORY_OF_DOCKER_COMPOSE_YML_FILE
     docker-compose up
     ```

