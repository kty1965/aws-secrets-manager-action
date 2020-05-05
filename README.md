# AWS Secrets Manager GitHub Action
GitHub Action to fetch secrets from AWS Secrets Manager. 

The fetched secrets are available as environment variables and can be used in subsequent steps in your workflow. 

## Usage
```yaml
steps:
 - name: Read secrets from AWS Secrets Manager into environment variables
   uses: action-factory/aws-secrets-manager-action@0.1
   with:
    aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws_region: ${{ secrets.AWS_REGION }}
    secret_names: |
      my_secret_1_name
      my_secret_2_name
    parse_json: true
```

- `aws_access_key_id`
  - Access Key ID of an IAM user with the required [AWS Secrets Manager permissions](#iam-policy)
- `aws_secret_access_key`
  - Corresponding Secret Access Key of the IAM user
- `aws_region`
  - AWS region code which has your AWS Secrets Manager secrets 
  - Example: `us-east-1`
- `secret_names`: 
  - List of secret names to be retrieved
  - To retrieve a single secret, use `secret_names: my_secret_1_name`
  - To retrieve multiple secrets, use: 
    ```yaml
    secret_names: |
      my_secret_1_name
      my_secret_2_name
    ```
- `parse_json`
  - Secret values can be plan text strings or stringified JSON objects (valid or invalid!).
  - If `parse_json: true` and secret value is a valid stringified JSON object, it will be parsed and flattened.
  - Each of its key value pairs will become individual secrets.
  - Examples: 
    - If `parse_json` is `true`, secret name is `foo` and value is `'{ "bar": "baz" }'` (valid stringified JSON), the parsed secret will be available as `foo.bar` with value `baz`.
    - If `parse_json` is `true`, secret name is `foo` and value is `'{ "bar" = "baz" }'` (invalid JSON string), the secret will not be parsed and will be available as `foo` with value `'{ "bar" = "baz" }'`.
    - If `parse_json` is `false`, secret name is `foo` and value is `'{ "bar": "baz" }'` (or any plain text string), the secret will be available as `foo` with value `'{ "bar": "baz" }'` (as it is). In this case, your application needs to handle parsing of the secret value if applicable.

## IAM Policy
The `aws_access_key_id` and `aws_secret_access_key` used above should belong to an IAM user with the following minimum permissions:
- `secretsmanager:GetSecretValue`
- `kms:Decrypt`
  - required only if you use a customer-managed AWS KMS key to encrypt the secret. You do not need this permission to use your account's default AWS managed encryption key for Secrets Manager.

#### Example 1:
 If your secrets are encrypted using the default AWS managed encryption key, then the IAM user needs to have a policy attached similar to:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

#### Example 2:
 If your secrets are encrypted using a customer managed AWS Key Management Service (KMS) key, then the IAM user needs a policy similar to the one below. We can restrict access to specific secrets (resources) in a specific region or we can use `*` for 'Any'.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "secretsmanager:GetSecretValue",
                "kms:Decrypt"
            ],
            "Effect": "Allow",
            "Resource": [
              "arn:aws:secretsmanager:us-east-1:000000000000:secret:*",
              "arn:aws:secretsmanager:*:000000000000:secret:mySecretID"
            ]
        }
    ]
}
```
Here `us-east-1` is the region code which has the secrets, `000000000000` is your [AWS account ID](https://console.aws.amazon.com/billing/home?#/account) and `mySecretID` is the ID of your secret (usually different from a secret name, refer to AWS Secrets Manager console for the exact ID).

## Contributing
We would love for you to contribute to [`@action-factory/aws-secrets-manager-action`](https://github.com/action-factory/aws-secrets-manager-action). [Issues](https://github.com/action-factory/aws-secrets-manager-action/issues) and [Pull Requests](https://github.com/action-factory/aws-secrets-manager-action/pulls) are welcome!

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/action-factory/aws-secrets-manager-action/blob/master/LICENSE).