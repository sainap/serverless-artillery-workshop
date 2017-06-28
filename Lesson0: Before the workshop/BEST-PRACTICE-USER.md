## Best Practice User
Good for you! You are aware of the consequences of permissions and have decided to simultaneously decrease risk of disaster and enhance your knowledge of AWS permissions. 

First, you will need to create a new policy to specify permissions for this workshop. Picking up where we left off in the README, click `create policy`. 
Name the policy what you wish, though we reccomend 'ServerlessDeployment'. Below you will find a policy script that provides least permissions necessary for a user to complete this workshop. Paste this script into the `Policy Document` section:
```

                 Version: '2012-10-17'
                  Statement:
                    - Effect: Allow
                      Action:
                        - 's3:CreateBucket'
                        - 's3:GetBucketLocation'
                        - 's3:GetObject'
                        - 's3:ListBucket'
                        - 's3:PutObject'
                      Resource: 'arn:aws:s3:::serverless-artillery-*'
                    - Effect: Allow
                      Action:
                        - 'cloudformation:CreateStack'
                        - 'cloudformation:UpdateStack'
                        - 'cloudformation:DeleteStack'
                        - 'cloudformation:DescribeStacks'
                        - 'cloudformation:DescribeStackEvents'
                        - 'cloudformation:DescribeStackResources'
                      Resource: 'arn:aws:cloudformation:${region}:${accountId}:stack/serverless-artillery-*'
```
*NOTE: The last line of the script needs some edits. `region` should be your preferred region, (us-east-1 reccomended) and `accountId` should be the account ID of the user being granted these permissions*
