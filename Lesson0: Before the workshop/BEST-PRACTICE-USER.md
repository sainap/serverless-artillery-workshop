## Best Practice User
Good for you! You are aware of the consequences of permissions and have decided to simultaneously decrease risk of disaster and enhance your knowledge of AWS permissions. 

First, you will need to create a new policy to specify permissions for this workshop. Picking up where we left off in the README, click `Create Policy`. 
Name the policy what you wish, though we reccomend 'ServerlessDeployment'. Below you will find a policy script that provides least permissions necessary for a user to complete this workshop. Paste this script into the `Policy Document` section:
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
*NOTE: Some resource lines of the script needs some edits. `<region>` should be your preferred region, (<region> reccomended, but a [list](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) of all regions is available) and `<accountID>` should be the account ID of the user being granted these permissions. `accountID` is the 12 digit number in the middle of `User ARN` which can be found on the mainpage of the user (IAM->Users->User).*

For documentation on what a policy is and a general deeper understanding of what this script is doing, checkout Amazon's [Overview of IAM Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).
