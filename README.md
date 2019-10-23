# Overview
A project to install, configure and manage org-sagebase-transit account.
The purpose of this account is to run the AWS transit gateway as the
"hub" of our hub and spoke architecture.

![alt text][architecture]


## Workflow
AWS enforces a specific workflow when setting up the Transit Gateway in a hub
and spoke architecture.

1. Setup the transit gateway in the central/hub account.
2. Create a transit gateway route table in the hub account.
3. Share the transit gateway in the hub account (VPC settings).
4. Share the transit gateway to spoke accounts with the Resource Access Manager.
5. Accept invitations from spoke account in the Resource Access Manager.
6. Create attachments to the transit gateway hub from spoke accounts.
7. Add ("Associate") spoke TGW attachments to the hub transit gateway route table.

### Setup a Hub
Setup a transit gateway("hub") and send invitations to share the
transit gateway to spoke accounts defined in the `TgwSharedPrincipals`
parameter.

#### Create a transit gateway
First step is to create the transit gateway, create a TGW route table, share the TGW
with within the transit account, then use the AWS resource access manager to share
the TGW to the spoke accounts.

1. Create a sceptre config template similar to config/prod/sage-tgw.yaml
2. Deploy the hub template:
`
sceptre --var "profile=transit" --var "region=us-east-1" launch --yes prod/new-tgw.yaml
`

__Note__: That template takes care of creating the TGW, creating the route table,
and sending an invition to share the TGW to spoke accounts.  However it does not
setup sharing within the same account.

#### Share the transit gateway
__Note__: The TGW must be shared within the transit account. Neither cloudformation nor
the aws cli support configuring the TGW `sharing` setting.

Login to the AWS transit account then navigate to VPC -> Transit Gateways ->
Select the newly provisioned transit gateway -> Sharing tab ->
Share transit gateway -> Share.

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

### Add spoke TGW attachments
__Note__: This step depends on having already completed `Setup a Hub` and `Setup a Spoke`.

1. Create a sceptre config template similar to config/prod/sage-tgw-associations.yaml
2. Deploy the template:
`
sceptre --var "profile=transit" --var "region=us-east-1" launch --yes prod/sage-tgw-association.yaml
`

That template will associate VPCs to your new route table.


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
