# Nostr Serverless Registry

## CloudFormation Template for NIP-05 and NIP-35

This CloudFormation template creates a serverless architecture with an AWS Lambda function, an Amazon API Gateway, and an Amazon DynamoDB table that can be used to implement the functionality specified in [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md) and [NIP-35](https://github.com/nostr-protocol/nips/blob/master/35.md).

## Requirements

- The AWS account must have a configured certificate in AWS Certificate Manager.
- The AWS account must have an Amazon Route 53 hosted zone configured with the certificate.

## Components

The Lambda function is responsible for handling incoming HTTP requests to the API Gateway and querying the DynamoDB table for data. The API Gateway is set up to route requests to the Lambda function, which will handle the request and return a response.

The Lambda function is designed to support requests of the form `https://<domain>/.well-known/nostr.json?name=<local-part>`, where `<domain>` is the domain of the internet identifier and `<local-part>` is the local part of the internet identifier. When a request is made to this endpoint, the Lambda function queries the DynamoDB table for the specified `<name>` and returns a JSON document object with a key "names" that is a mapping of names to public keys. In addition, the returned JSON document may also include an optional "relays" key, which contains an object with public keys as properties and arrays of relays as values.

The DynamoDB table is created with a name that is the same as the CloudFormation stack name and is given a name attribute as its primary key. The table is used to store data about names and public keys, as well as any relays associated with each public key. The table's structure is meant to be:

| key       | value            |
|:----------|:-----------------|
| name      | name             |
| publicKey | nostr public key |
| relays    | right-aligned    |

The template also creates an IAM role for the Lambda function, which grants the function permission to access the DynamoDB table and create logs in Amazon CloudWatch Logs. Additionally, the template creates a log group for the Lambda function in CloudWatch Logs,which can be used to store and view logs generated by the function.

The Parameters section of the template allows you to specify the certificate ARN, domain name, and hosted zone ID that you want to use. The certificate ARN is the Amazon Resource Name (ARN) of the certificate you want to use. The domain name is the domain of the internet identifier, and the hosted zone ID is the ID of the hosted zone you want to use.

In the Resources section, the template creates the resources needed for the serverless architecture, including the Lambda function, IAM role, log group, DynamoDB table, and API Gateway. The Lambda function is given a code block that defines its behavior, and the IAM role is given a policy that grants it the necessary permissions to access the DynamoDB table and create logs in CloudWatch Logs. The log group is created to store logs generated by the Lambda function, and the DynamoDB table is created to store data about names, public keys, and relays. Finally, the API Gateway is created and configured to route requests to the Lambda function.

## Deployment

To deploy this template, simply follow the steps in the AWS CloudFormation documentation to create a new stack and specify this template as the source. During the stack creation process, you will be prompted to enter values for the parameters. Once the stack is deployed, you can use it to implement the functionality specified in [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md) and [NIP-35](https://github.com/nostr-protocol/nips/blob/master/35.md). Users can make requests to the API Gateway endpoint to discover public keys and relays for internet identifiers, and the Lambda function will handle these requests by querying the DynamoDB table and returning the appropriate data in the response.

## Fill DynamoDb

To insert the element into a DynamoDB table using the AWS CLI, you can use the aws dynamodb `put-item` command.

Here's an example of how you can insert the element into a table named `nostr-serverless-registry`:

``` bash
aws dynamodb put-item \
    --table-name nostr-serverless-registry \
    --item '{
        "name": {"S": "Giowe"},
        "publicKey": {"S": "973ed14e7948c6d34ee158bdf1f3ab07dc4fdfcf83f95123e4ddc20bfcb586f5"},
        "relays": {"L": [{"S": "wss://nostr-pub.wellorder.net"}, {"S": "wss://relay.damus.io"}, {"S": "wss://relay.nostr.info"}]}
    }'
```

This command will insert the element into the table with the following attribute values:

`name`: a string with the value "Giowe"

`publicKey`: a string with the value "973ed14e7948c6d34ee158bdf1f3ab07dc4fdfcf83f95123e4ddc20bfcb586f5"

`relays`: a list of strings with the values "wss://nostr-pub.wellorder.net", "wss://relay.damus.io", and "wss://relay.nostr.info"
