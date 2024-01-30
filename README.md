AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  VpcCIDR:
    Type: String
    Default: '10.0.0.0/16'
    Description: 'CIDR block for the VPC'
  PublicSubnet1CIDR:
    Type: String
    Default: '10.0.1.0/24'
    Description: 'CIDR block for the first public subnet'
  PublicSubnet2CIDR:
    Type: String
    Default: '10.0.2.0/24'
    Description: 'CIDR block for the second public subnet'
  PrivateSubnet1CIDR:
    Type: String
    Default: '10.0.3.0/24'
    Description: 'CIDR block for the first private subnet'
  PrivateSubnet2CIDR:
    Type: String
    Default: '10.0.4.0/24'
    Description: 'CIDR block for the second private subnet'
Resources:
  AssignmentVPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: AssignmentVPC
  PublicSubnet1:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref AssignmentVPC
      CidrBlock: !Ref PublicSubnet1CIDR
      AvailabilityZone: !Select [0, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet1
  PublicSubnet2:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref AssignmentVPC
      CidrBlock: !Ref PublicSubnet2CIDR
      AvailabilityZone: !Select [1, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet2
  PrivateSubnet1:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref AssignmentVPC
      CidrBlock: !Ref PrivateSubnet1CIDR
      AvailabilityZone: !Select [0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: PrivateSubnet1
  PrivateSubnet2:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref AssignmentVPC
      CidrBlock: !Ref PrivateSubnet2CIDR
      AvailabilityZone: !Select [1, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: PrivateSubnet2
  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    Properties:
      Tags:
        - Key: Name
          Value: InternetGateway
  AttachGateway:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref AssignmentVPC
      InternetGatewayId: !Ref InternetGateway
  PublicRouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref AssignmentVPC
      Tags:
        - Key: Name
          Value: PublicRouteTable
  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway
  PublicSubnet1Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable
  PublicSubnet2Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable
  PrivateRouteTable1:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref AssignmentVPC
      Tags:
        - Key: Name
          Value: PrivateRouteTable1
  PrivateRoute1:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NatGateway1
  PrivateRouteTable2:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref AssignmentVPC
      Tags:
        - Key: Name
          Value: PrivateRouteTable2
  PrivateRoute2:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NatGateway2
  NatGateway1:
    Type: 'AWS::EC2::NatGateway'
    Properties:
      AllocationId: !GetAtt ElasticIP1.AllocationId
      SubnetId: !Ref PublicSubnet1
  ElasticIP1:
    Type: 'AWS::EC2::EIP'
  NatGateway2:
    Type: 'AWS::EC2::NatGateway'
    Properties:
      AllocationId: !GetAtt ElasticIP2.AllocationId
      SubnetId: !Ref PublicSubnet2
  ElasticIP2:
    Type: 'AWS::EC2::EIP'
  PrivateSubnet1Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnet1
      RouteTableId: !Ref PrivateRouteTable1
  PrivateSubnet2Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnet2
      RouteTableId: !Ref PrivateRouteTable2
  SecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow all inbound and outbound traffic'
      VpcId: !Ref AssignmentVPC
  PublicSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow inbound traffic to public instances'
      VpcId: !Ref AssignmentVPC
  PrivateSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow inbound traffic from instances in the VPC'
      VpcId: !Ref AssignmentVPC
