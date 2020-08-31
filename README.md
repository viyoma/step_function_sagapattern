# lambda-saga-pattern

Implements the Saga pattern for Lambda functions using Step Functions.

The example is based on Catie McCaffrey's example in her talk on [Applying the
Saga pattern](https://www.youtube.com/watch?v=xDuwrtwYHu8).

Each request has a compensating request for rollback.

The Saga goes like this:

```
Begin saga
  Start book hotel request
  End book hotel request
  Start book flight request
  End book flight request
  Start book car rental request
  End book car rental request  
End saga
```

## Deployment

#### Steps to set up the Distribution

  a. Build lambda function, and prepare them for subsequent steps in the workflow
  
      Command: sam build -b ./build -s . -t template.yaml -u

  b. Packages the above LambdaFunction. It creates a ZIP file of the code and dependencies, and uploads it to Amazon S3 (please create the S3 bucket and mention the bucket name in the command below). It then returns a copy of AWS SAM template, replacing references to local artifacts with the Amazon S3 location where the command uploaded the artifacts

      Command: sam package \
                --template-file build/template.yaml \
                --s3-bucket viyoma-private-s3 \
                --output-template-file build/packaged.yaml

  c. Deploy Lambda functions through AWS CloudFormation from the S3 bucket created above. AWS SAM CLI now creates and manages this Amazon S3 bucket for you.

      Command:  sam deploy \
                --template-file build/packaged.yaml \
                --stack-name oidc-auth2 \
                --capabilities CAPABILITY_NAMED_IAM 

#### Steps to set up the State Machine

  a. Got to Step Functions and Create a stae Machine using states.json, replace the lambda function Arn's from your ARN's

  b. Execute the state Machine for success by the following input:


          {
              "tripId": "5c12d94a-ee6a-40d9-889b-1d49142248b7",
              "depart": "London",
              "departAt": "2017-07-10T06:00:00.000Z",
              "arrive": "Dublin",
              "arriveAt": "2017-07-12T08:00:00.000Z",
              "hotel": "holiday inn",
              "checkIn": "2017-07-10T12:00:00.000Z",
              "checkOut": "2017-07-12T14:00:00.000Z",
              "car": "Volvo",
              "carFrom": "2017-07-10T00:00:00.000Z",
              "carTo": "2017-07-12T00:00:00.000Z"
              }

  c. Execute the state Machine for Failure by the following input:

        {
              "tripId": "5c12d94a-ee6a-40d9-889b-1d49142248b7",
              "depart": "London",
              "departAt": "2017-07-10T06:00:00.000Z",
              "arrive": "Dublin",
              "arriveAt": "2017-07-12T08:00:00.000Z",
              "hotel": "holiday inn",
              "checkIn": "2017-07-10T12:00:00.000Z",
              "checkOut": "2017-07-12T14:00:00.000Z",
              "car": "Volvo",
              "carFrom": "2017-07-10T00:00:00.000Z",
              "carTo": "2017-07-12T00:00:00.000Z",
              "failBookFlight": true
              }

