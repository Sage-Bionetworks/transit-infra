# Overview
A project to install, configure and manage org-sagebase-transit account.
The purpose of this account is to run the AWS transit gateway as the
"hub" of our hub and spoke architecture.

![alt text][architecture]


## Workflow
AWS enforces a specific workflow when setting up the hub and spoke accounts.
This workflow involves establishing connection between hub and spoke by
sending and accepting invitations.

### Setup a Hub
Setup a transit gateway("hub") and send invitations to share the
transit gateway to spoke accounts defined in the `TgwSharedPrincipals`
parameter.

1. Create a sceptre config template similar to config/prod/sage-tgw.yaml
2. Deploy the hub template:
`
sceptre --var "profile=transit" --var "region=us-east-1" launch --yes prod/new-tgw.yaml
`
3. Login to the AWS transit account then navigate to VPC -> Transit Gateways ->
Select the newly provisioned transit gateway -> Sharing tab ->
Share transit gateway -> Share.

__Note__: This TGW must be shared. Neither cloudformation nor the aws cli support
setting the `sharing` config.

### Setup a Spoke
Setting up the spoke account is a two step process.  First is to accept the
sharing invitation from the hub then create an attachment to the hub.

#### Accept Invitation
The spoke accounts must accept the invitation sent by the hub.
The transit gateway attachments are then setup on the spoke accounts.

1. Login to spoke accounts defined in TgwSharedPrincipals
2. Navigate to Resource Access Manager -> Shared with me -> Resource Shares ->
accept the invitation.
3. Once the invitation is accepted the shared transit gateway (from "hub")
should appear in VPC -> TransitGateway.

#### Setup Attachement
Setting up an attachment adds spoke accounts to the hub account however it does
not setup routes between VPCs.

1. Create an attachment to the hub from the spoke accounts by creating
a sceptre template similar to
`sandbox-infra/sage-tgw-sandcastlevpc-attachment.yaml`
2. Deploy the spoke template:
`
sceptre --var "profile=transit" --var "region=us-east-1" launch --yes prod/sage-tgw-sandcastlevpc-attachment.yaml
`

## Instructions to create or update CF stacks

```
# Update CF stacks with sceptre:
# sceptre launch-stack prod <stack_name>
```

The above should setup resources for the AWS account.  Once the infrastructure
for the account has been setup you can access and view the account using the
[AWS console](https://AWS-account-ID-or-alias.signin.aws.amazon.com/console).

*Note - This project depends on CF templates from other accounts.*

## Contributions
Contributions are welcome.

Requirements:
* Install [pre-commit](https://pre-commit.com/#install) app
* Clone this repo
* Run `pre-commit install` to install the git hook.

## Testing
As a pre-deployment step we syntatically validate our sceptre and
cloudformation yaml files with [pre-commit](https://pre-commit.com).

Please install pre-commit, once installed the file validations will
automatically run on every commit.  Alternatively you can manually
execute the validations by running `pre-commit run --all-files`.
Please install pre-commit, once installed the file validations will
automatically run on every commit.

## Issues
* https://sagebionetworks.jira.com/projects/IT

## Secrets
* We use the [AWS SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html)
to store secrets for this project.  Sceptre retrieves the secrets using
a [sceptre ssm resolver](https://github.com/cloudreach/sceptre/tree/v1/contrib/ssm-resolver)
and passes them to the cloudformation stack on deployment.


[architecture]: transit-gateway-arch.png "hub and spoke architecture"
