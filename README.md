# CloudFormation
#Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.

  #InstType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
    - t1.micro
    - t2.nano
    - t2.micro
    - t2.small
    - t2.medium

  #SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

#Mappings:
  AWSInstanceType2Arch:
    t1.micro:
      Arch: PV64
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64
  #AWSRegionArch2AMI:
    us-east-1:
      PV64: ami-2a69aa47
      HVM64: ami-6869aa05
      HVMG2: ami-61e27177
    us-west-2:
      PV64: ami-7f77b31f
      HVM64: ami-7172b611
      HVMG2: ami-60aa3700

#Resources:
  InstSecGrp:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: cf-sg-cb-test
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp:
          Ref: SSHLocation
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0

  #EC2Webserver:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType:
        Ref: InstType
      SecurityGroups:
      - Ref: InstSecGrp
      KeyName:
        Ref: KeyName
      ImageId:
        Fn::FindInMap:
        - AWSRegionArch2AMI
        - Ref: AWS::Region
        - Fn::FindInMap:
          - AWSInstanceType2Arch
          - Ref: InstType
          - Arch
      Tags:
        - Key: "Name"
          Value: "cf-ss-ws"
        - Key: "Type"
          Value: "Webserver"

#Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value:
      Ref: EC2Webserver
  #AZ:
    Description: Availability Zone of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Webserver
      - AvailabilityZone
  #PublicDNS:
    Description: Public DNSName of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Webserver
      - PublicDnsName
  #PublicIP:
    Description: Public IP address of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Webserver
      - PublicIp
