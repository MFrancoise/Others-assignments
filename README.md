{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Deploy 3 EC2 instances, each with its own OS selection and personalized Apache/IIS message",

  "Parameters": {
    "VpcId": { "Type": "AWS::EC2::VPC::Id" },
    "SubnetId": { "Type": "AWS::EC2::Subnet::Id" },
    "KeyName": { "Type": "AWS::EC2::KeyPair::KeyName" },

    "CustomHtml1": { "Type": "String" },
    "CustomHtml2": { "Type": "String" },
    "CustomHtml3": { "Type": "String" },

    "InstanceName": { "Type": "String" },

    "SelectedRegion": {
      "Type": "String",
      "AllowedValues": ["us-east-1","us-east-2","us-west-1","us-west-2"]
    },

    "InstanceType": {
      "Type": "String",
      "Default": "t2.micro",
      "AllowedValues": ["t2.micro","t3.micro","t3.small"]
    },

    "OS1": {
      "Type": "String",
      "AllowedValues": ["Ubuntu", "AmazonLinux", "Windows"]
    },
    "OS2": {
      "Type": "String",
      "AllowedValues": ["Ubuntu", "AmazonLinux", "Windows"]
    },
    "OS3": {
      "Type": "String",
      "AllowedValues": ["Ubuntu", "AmazonLinux", "Windows"]
    }
  },

  "Mappings": {
    "RegionMap": {
      "us-east-1": {
        "Ubuntu": "ami-0b6d9d3d33ba97d99",
        "AmazonLinux": "ami-0c02fb55956c7d316",
        "Windows": "ami-09639480113b0df96"
      },
      "us-east-2": {
        "Ubuntu": "ami-0f2b4fc905b0bd1f1",
        "AmazonLinux": "ami-00399ec92321828f5",
        "Windows": "ami-0ae452a0c234d1923"
      },
      "us-west-1": {
        "Ubuntu": "ami-07d9b9ddc6cd8dd30",
        "AmazonLinux": "ami-0b2f6494ff0b07a0e",
        "Windows": "ami-06d68239bc44b141b"
      },
      "us-west-2": {
        "Ubuntu": "ami-0d527b8c289b4af7f",
        "AmazonLinux": "ami-0892d3c7ee57c1fce",
        "Windows": "ami-02df557e4e9e6c1ea"
      }
    }
  },

  "Conditions": {
    "IsUbuntu1": { "Fn::Equals": [{ "Ref": "OS1" }, "Ubuntu"] },
    "IsAmazonLinux1": { "Fn::Equals": [{ "Ref": "OS1" }, "AmazonLinux"] },
    "IsWindows1": { "Fn::Equals": [{ "Ref": "OS1" }, "Windows"] },

    "IsUbuntu2": { "Fn::Equals": [{ "Ref": "OS2" }, "Ubuntu"] },
    "IsAmazonLinux2": { "Fn::Equals": [{ "Ref": "OS2" }, "AmazonLinux"] },
    "IsWindows2": { "Fn::Equals": [{ "Ref": "OS2" }, "Windows"] },

    "IsUbuntu3": { "Fn::Equals": [{ "Ref": "OS3" }, "Ubuntu"] },
    "IsAmazonLinux3": { "Fn::Equals": [{ "Ref": "OS3" }, "AmazonLinux"] },
    "IsWindows3": { "Fn::Equals": [{ "Ref": "OS3" }, "Windows"] }
  },

  "Resources": {
    "WebSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "Enable HTTP only",
        "VpcId": { "Ref": "VpcId" },
        "SecurityGroupIngress": [
          { "IpProtocol": "tcp", "FromPort": 80, "ToPort": 80, "CidrIp": "0.0.0.0/0" }
        ]
      }
    },

    "WebServer1": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId": {
          "Fn::FindInMap": ["RegionMap", { "Ref": "SelectedRegion" }, { "Ref": "OS1" }]
        },
        "InstanceType": { "Ref": "InstanceType" },
        "KeyName": { "Ref": "KeyName" },
        "SubnetId": { "Ref": "SubnetId" },
        "SecurityGroupIds": [{ "Ref": "WebSecurityGroup" }],
        "Tags": [
          { "Key": "Name", "Value": { "Fn::Sub": "${InstanceName}-1" } },
          { "Key": "OS", "Value": { "Ref": "OS1" } }
        ],
        "UserData": {
          "Fn::Base64": {
            "Fn::If": [
              "IsWindows1",
              { "Fn::Sub": "<powershell>\nInstall-WindowsFeature Web-Server\nSet-Content -Path \"C:\\inetpub\\wwwroot\\index.html\" -Value \"${CustomHtml1}\"\n</powershell>" },
              {
                "Fn::If": [
                  "IsAmazonLinux1",
                  { "Fn::Sub": "#!/bin/bash\nyum update -y\nyum install -y httpd\nsystemctl enable httpd\nsystemctl start httpd\necho \"${CustomHtml1}\" > /var/www/html/index.html\n" },
                  { "Fn::Sub": "#!/bin/bash\napt update -y\napt install -y apache2\necho \"${CustomHtml1}\" > /var/www/html/index.html\nsystemctl restart apache2\n" }
                ]
              }
            ]
          }
        }
      }
    },

    "WebServer2": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId": {
          "Fn::FindInMap": ["RegionMap", { "Ref": "SelectedRegion" }, { "Ref": "OS2" }]
        },
        "InstanceType": { "Ref": "InstanceType" },
        "KeyName": { "Ref": "KeyName" },
        "SubnetId": { "Ref": "SubnetId" },
        "SecurityGroupIds": [{ "Ref": "WebSecurityGroup" }],
        "Tags": [
          { "Key": "Name", "Value": { "Fn::Sub": "${InstanceName}-2" } },
          { "Key": "OS", "Value": { "Ref": "OS2" } }
        ],
        "UserData": {
          "Fn::Base64": {
            "Fn::If": [
              "IsWindows2",
              { "Fn::Sub": "<powershell>\nInstall-WindowsFeature Web-Server\nSet-Content -Path \"C:\\inetpub\\wwwroot\\index.html\" -Value \"${CustomHtml2}\"\n</powershell>" },
              {
                "Fn::If": [
                  "IsAmazonLinux2",
                  { "Fn::Sub": "#!/bin/bash\nyum update -y\nyum install -y httpd\nsystemctl enable httpd\nsystemctl start httpd\necho \"${CustomHtml2}\" > /var/www/html/index.html\n" },
                  { "Fn::Sub": "#!/bin/bash\napt update -y\napt install -y apache2\necho \"${CustomHtml2}\" > /var/www/html/index.html\nsystemctl restart apache2\n" }
                ]
              }
            ]
          }
        }
      }
    },

    "WebServer3": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId": {
          "Fn::FindInMap": ["RegionMap", { "Ref": "SelectedRegion" }, { "Ref": "OS3" }]
        },
        "InstanceType": { "Ref": "InstanceType" },
        "KeyName": { "Ref": "KeyName" },
        "SubnetId": { "Ref": "SubnetId" },
        "SecurityGroupIds": [{ "Ref": "WebSecurityGroup" }],
        "Tags": [
          { "Key": "Name", "Value": { "Fn::Sub": "${InstanceName}-3" } },
          { "Key": "OS", "Value": { "Ref": "OS3" } }
        ],
        "UserData": {
          "Fn::Base64": {
            "Fn::If": [
              "IsWindows3",
              { "Fn::Sub": "<powershell>\nInstall-WindowsFeature Web-Server\nSet-Content -Path \"C:\\inetpub\\wwwroot\\index.html\" -Value \"${CustomHtml3}\"\n</powershell>" },
              {
                "Fn::If": [
                  "IsAmazonLinux3",
                  { "Fn::Sub": "#!/bin/bash\nyum update -y\nyum install -y httpd\nsystemctl enable httpd\nsystemctl start httpd\necho \"${CustomHtml3}\" > /var/www/html/index.html\n" },
                  { "Fn::Sub": "#!/bin/bash\napt update -y\napt install -y apache2\necho \"${CustomHtml3}\" > /var/www/html/index.html\nsystemctl restart apache2\n" }
                ]
              }
            ]
          }
        }
      }
    }
  },

  "Outputs": {
    "Instance1PublicIP": { "Value": { "Fn::GetAtt": ["WebServer1", "PublicIp"] } },
    "Instance2PublicIP": { "Value": { "Fn::GetAtt": ["WebServer2", "PublicIp"] } },
    "Instance3PublicIP": { "Value": { "Fn::GetAtt": ["WebServer3", "PublicIp"] } }
  }
}
