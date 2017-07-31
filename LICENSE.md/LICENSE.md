AWS CloudFormation UserData for Ubuntu 16.04 LTS

I use CloudFormation to create EC2 instances on AWS. I use this UserData script to install the CloudFormation scripts on the EC2 instance and then to run the init and signal steps.

       "UserData": {
         "Fn::Base64": {
           "Fn::Join": [
             "“,
             [
               ”#!/bin/bash -x\n",
               "apt-get update\n",
               "apt-get install -y python-pip\n",
               "pip install https://s3.amazonaws.com/cloudformation-examples/aws-cfn-bootstrap-latest.tar.gz\n“,
               "cfn-init -v –resource <RESOURCE NAME CONTAINING INIT CFG>”,
               " –stack “,
               {
                 "Ref”: “AWS::StackName”
               },
               " –region “,
               {
                 "Ref”: “AWS::Region”
               },
               "\n",
               "cfn-signal -e $? –resource <RESOURCE NAME CONTAINING CREATION POLICY>“,
               ” –stack “,
               {
                 "Ref”: “AWS::StackName”
               },
               " –region “,
               {
                 "Ref”: “AWS::Region”
               },
               "\n"
             ]
           ]
         }
       }
     }

You then need to add the CloudFormation init configuration to the resource named above. I put it in the instance resource, but I don’t think it has to be.

For example:

“Metadata”: {
       "AWS::CloudFormation::Init": {
         "config": {
         	“packages”: {
         	“apt”: {
         	“awscli”: []
         	}
         }
       }
     }
   }

You can also add a CreationPolicy onto a resource; usually the instance or the auto-scaling group. Creation policies have replaced WaitConditions.

     "CreationPolicy": {
       "ResourceSignal": {
         "Timeout": “PT5M”
       }
     }

