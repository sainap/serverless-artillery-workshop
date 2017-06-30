## Best User - Least Permissions
Good for you! You are aware of the consequences of permissions and have decided to simultaneously decrease risk of disaster and enhance your knowledge of AWS permissions. 

Something to understand about AWS is the differentiation of users, roles, and policies. A user is something that can be verified, it has authentication. Policies and roles, on the other hand, provide authorization of certain activities within the scope of all things AWS. A role can be temporarily assumed by a user, and a policy can be attached to a user. A certain role can also carry several policies, and several policies can be attached to a user as well. 
Right now, you will create a role that is to be assumed by the user you have created. This role will allow the user that assumes it the least permissions needed to complete this workshop without any problems. 
To create a new role, log into the AWS console and navigate to IAM->roles->Create new role->Role for cross-account access. Select `Provide access between AWS accounts you own`. Enter the account ID of the account you are using for this workshop on the next page (Do not require MFA). At step 3, do nothing and click next step. At step 4, name the role something easy to remember and associate with this workshop, perhaps `slsart-workshop-role`. Create the role, then select it from the roles page you will be automatically navigated to.
In the Permissions tab, click `Inline Policies` and create one. The policy should be a custom policy. Name is something easy, like `slsart-workshop-policy` and paste in the script below:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:DeleteBucket",
                "s3:DeleteObject",
                "s3:DeleteObjectVersion",
                "s3:PutLifeCycleConfiguration"
            ],
            "Resource": "arn:aws:s3:::serverless-artillery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:CreateStack",
                "cloudformation:UpdateStack",
                "cloudformation:DeleteStack",
                "cloudformation:DescribeStacks",
                "cloudformation:DescribeStackEvents",
                "cloudformation:DescribeStackResource",
                "cloudformation:DescribeStackResources"
            ],
            "Resource": "arn:aws:cloudformation:<region>:<accountID>:stack/serverless-artillery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:ValidateTemplate"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "iam:CreateRole",
                "iam:DeleteRolePolicy",
                "iam:PutRolePolicy",
                "iam:DeleteRole",
                "iam:CreateFunction",
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::<accountID>:role/serverless-artillery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "lambda:UpdateFunctionCode",
                "lambda:ListVersionsByFunction",
                "lambda:PublishVersion",
                "lambda:InvokeFunction",
                "lambda:GetFunction",
                "lambda:CreateFunction",
                "lambda:DeleteFunction"
            ],
            "Resource": "arn:aws:lambda:<region>:<accountID>:function:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:DescribeLogGroups",
                "logs:CreateLogGroup",
                "logs:DeleteLogGroup"
            ],
            "Resource": "arn:aws:logs:<region>:<accountID>:log-group:*"
        }
    ]
}
```

*NOTE: Some resource lines of the script needs some edits. `<region>` should be your preferred region, (<region> recommended, but a [list](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) of all regions is available) and `<accountID>` should be the account ID of the user being granted these permissions. `accountID` is the 12 digit number in the middle of `User ARN` which can be found on the mainpage of the user (IAM->Users->User).*

For documentation on what a policy is and a general deeper understanding of what this script is doing, checkout Amazon's [Overview of IAM Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).
Validate and save the policy before moving on.
Click the trust relationships tab and then click Edit trust relationship. The line within the `Principal` section should be:
```
"AWS": "arn:aws:iam::<accountID>:<accountName>
```
If the name is correct, Update the Trust Policy and you'll be good to go on to the next part.
Now that you have created a role, return to the command line and access the aws credentials file 
`cd .aws/credentials`
Create new credentials limited only to assuming the role you have created:
``` 
[<accountName>-role]
role_arn = arn:aws:iam::<accountID>:role/slsart-workshop-role
source_profile = <base-profile> 
```
<base_profile> is the profile name associated with the credentials for the user you previously created (in the 3 lines above, for example, the profile name is the first line without the brackets). If you did not name your role `slsart-workshop-role`, that will be different as well.

Finally, export the newly created credentials.
`export AWS_PROFILE=<accountName>-role`
Now, after all that hard work, you are rewarded with 
    a) insight into aws security credentials, very valuable knowledge in this day and age
    b) credentials that come with a very limited blast radius, meaning that if you make a mistake, it won't snowball into anything actually serious. Now, let's continue with the workshop.
