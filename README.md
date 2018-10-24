# fcreds awssecrets

Fcreds awssecrets is a tiny command line utility to pull secrets created with [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/).
[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) is AWS's best practice for storing application secrets.  Used in conjuction with very granular IAM policies (tied to machine roles or keys) means a pretty small surface area for secret exposure. 

awssecrets is written in Go with binaries for eah major platform and was inspired by https://github.com/Versent/unicreds

## Usage

```bash
usage: awssecrets --region=REGION [<flags>] <command> [<args> ...]

A CLI tool to get secrets from AWS secrets manager.

Flags:
      --help             Show context-sensitive help (also try --help-long and --help-man).
  -r, --region=REGION    Configure the AWS region
  -p, --profile=PROFILE  Configure the AWS profile

Commands:
  help [<command>...]
    Show help.

  exec --secret-name=SECRET-NAME <command>...
    Execute a command with all secrets loaded as environment variables.
```

awssecrets supports the AWS_* environment variables, and configuration in `~/.aws/`credentials` and `~/.aws/config`

## Examples

First create a secret in secrets manager using the AWS CLI tools

```bash
aws secretsmanager create-secret --name postgresqlRootPwd --secret-string password123DontHackMe
```

Now execute `env` command, all secrets are loaded as environment variables.

```bash
awssecrets -r us-east-1 exec -n postgresqlRootPwd -n mysqlRootPwd -- env
```

\- or -

```Dockerfile
# how to use within a docker container
RUN curl -sL \
  https://github.com/EconomistDigitalSolutions/fcreds/releases/download/v1.0/awssecrets_linux_amd64.tar.gz \
 | tar zx -C /usr/local/bin \
 && chmod +x /usr/local/bin/fcreds

# our worker code is `node worker.js` simply prefix
# exposing it this way as opposed to using an ENV statement means that only the application
# has the secret not the whole contianer, like a local var as opposed to global scope
CMD /usr/local/bin/fcreds -r eu-west-2 exec -n postgresqlRootPwd -- node worker.js
```

## IAM policy

To access the secret value from your application use [tight and granular IAM policies](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access_identity-based-policies.html#permissions_grant-limited-resources) tied to roles. Any role used to call fcreds will require an IAM policy that looks a bit like this:

```YAML
- PolicyName: SecretsManagerAccess
  PolicyDocument:
    Statement:
    - Effect: Allow
      Action:
      - secretsmanager:GetSecretValue
      Resource:
      - arn:aws:secretsmanager:region:<accountId>:secret:postgresqlRootPwd
```
## Release

To release a new version you'll need Docker running on your machine and the environment variable `GITHUB_TOKEN` set locally. Then we can run

```bash
GITHUB_TOKEN=123456ABCDEF ./release.sh v1.2
```

`release.sh` takes a version number as a parameter (or we'll try to release v1.0 by default)
