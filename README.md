# CloudFormation Bringup

Most of OpenlyOperated.org's backend can be brought up with CloudFormation. However, because CloudFormation doesn't support specific actions yet, some steps have to be done manually. To see how each piece fits together in the overall infrastructure, see the Architecture diagram in the Shared repository.

# Bringup Instructions

Start with `0-Global.yml` and end in `3-Main.yml`.

Use either the web CloudFormation console to do the bringup, or use the CLI. The console is more versatile because the CLI has a size limit of ~52KB per YAML file.

## Prerequisites

1. __AWS Simple Email Service__ - Configure AWS Simple Email Service and get out of the sandbox so sending email is allowed. Also, verify the `team@[domain]` and `admin@[domain]` addresses. __[Instructions](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/request-production-access.html)__

2. __AWS Route 53__ - Create a Route 53 Public Hosted Zone for the service's domain (e.g, openlyoperated.org) and point the nameservers to it. __[Route 53](https://console.aws.amazon.com/route53/home#hosted-zones:)__

3. __Domain Email Access__ - Must have access to the following email addresses: `hostmaster@[domain]`, `team@[domain]`, and `admin@[domain]`. This is for SSL certificate validation, support emails, and admin/alert emails. 

## 0-Global.yml

Run once per AWS account. This configures:

1. CloudTrail
2. CloudWatch alerts for CloudTrail
3. Audit IAM group and user
4. Open Watch account

Parameter | Description
--- | --- 
`AlertEmail` | Email address to send CloudWatch Alerts to.
`TrailName` | Name of the CloudTrail to create. Defaults to `CloudTrail`.

### After Completion
The Audit User and Open Watch users' access id and key are visible in "Outputs" of the CloudFormation stack.

## 1-Shared.yml

This configures shared roles, the database, Lambda functions, etc that are used by both the `Main` servers and `Admin` server. This also brings up CodeCommit (git) repositories, S3 buckets, and Log Groups.

Parameter | Description
--- | --- 
`Environment` | Name of environment to bring up (e.g, PROD).
`GitBranch` | Which Git branch's code to deploy.
`Domain` | Domain to deploy the site to.

### After Completion

__Configure Git and Push to CodeCommit__

`git clone` each of `Shared`, `Admin`, and `Main` to a local machine. Then, run the following:

```
REGION=us-east-1
ENV=PROD
BRANCH=prod
REPOS=(
  Shared
  Admin
  Main
)
	
for REPO in "${REPOS[@]}"
do
  cd $REPO
  git remote add $ENV https://git-codecommit.$REGION.amazonaws.com/v1/repos/$ENV-$REPO
  git remote set-url --add --push $ENV https://git-codecommit.$REGION.amazonaws.com/v1/repos/$ENV-$REPO
  git push $ENV $BRANCH 
  cd ..
done
```
  
## 2-Admin.yml

This is the Admin server that only a whitelisted IP CIDR range has access to. It initializes the database schema, has a dashboard for Administrators, content management system, newsletter management, amongst other things.

### Initialize Database

Go to `https://admin.[domain]/?initialize=true`, which initializes the database schema and redirects to sign in page.

### Create Admin User

Go to `https://admin.[domain]/signup` to create new admin account. The admin account must have the same domain as the domain of the environment. Confirm the account with email, then sign in.

## 3-Main.yml

Run in `Base` region. This is the Main publicly facing server that hosts the website for the public to view.

## Feedback
If you have any questions, concerns, or other feedback, please let us know any feedback in Github issues or by e-mail.

## License

This project is licensed under the GPL License - see the [LICENSE.md](LICENSE.md) file for details

## Contact

<engineering@openlyoperated.org>