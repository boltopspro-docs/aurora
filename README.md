<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/aurora/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# RDS Aurora Database CloudFormation Blueprint

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/blueprints/aurora-mysql.png)](https://youtu.be/DQxhuZ9T7e8)

For videos with the other database engines like PostgreSQL go to the [Videos page](docs/videos.md).

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiUjhkZDZwNEo4Mm9Pdk9xTXZudW5vN3JmYzNxZkltVVZFc1Q1a2Z4NmlIUHBsaVllRWtKNmgyK0Nzdkl5U1NGRzJhZE1uMDhSMVpwa2Z3SWdJckFsZVZZPSIsIml2UGFyYW1ldGVyU3BlYyI6ImR0TlZaajlSM3VZcjhuMGYiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions a RDS Aurora Database Cluster.

* Several [AWS::RDS::DBCluster](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-enablecloudwatchlogsexports) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is extremely flexible and configurable for your needs.
* The default database is aurora-mysql. For a standard RDS Database, refer to the [RDS Blueprint](https://github.com/boltopspro-docs/rds). You might also be interested in the [aurora-serverless](https://github.com/boltopspro-docs/aurora-serverless) blueprint.
* You can use different Aurora databases like MySQL and PostgreSQL.
* Storage is encrypted by default.
* A managed DB Subnet Group can be created by configuring `SubnetIds`. Or you can use an existing `DBSubnetGroupName`.
* The DB Count can be controlled with the `@db_count` variable.
* Or AutoScaling can be enabled to scale DB Instances automatically.  The `PredefinedMetricType` can be controlled as a parameter.
* Can create an optional managed Route53 records that points to the cluster reader and writer endpoint.
* Cluster Parameters can be configured with code.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/aurora values
3. Deploy blueprint

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "aurora", git: "git@github.com:boltopspro/aurora.git"
```

## Configure

Use the [lono seed](https://lono.cloud/reference/lono-seed/) command to generate a starter config params files.

    LONO_ENV=development lono seed aurora
    LONO_ENV=production  lono seed aurora

The files in `config/aurora` folder will look something like this:

    configs/aurora/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Configure the `configs/aurora/params` and `configs/aurora/variables` files.  The parameters required: `MasterUsername`, `MasterUserPassword`, and `VpcId`.  Example:

configs/aurora/params/development.txt:

    DBClusterIdentifier=mydb
    DBInstanceClass=db.t3.small
    DBParameterGroupFamily=aurora-mysql5.7
    Engine=aurora-mysql
    EngineVersion=5.7.12
    MasterUsername=myuser
    MasterUserPassword=mypassword
    VpcId=vpc-111

configs/aurora/variables/development.rb:

```ruby
@db_count = 1 # includes the master. So `@db_count = 3` means 1 master and 2 replicas
```

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy.

    LONO_ENV=development lono cfn deploy aurora --sure --no-wait
    LONO_ENV=production  lono cfn deploy aurora --sure --no-wait

It takes about 15m to deploy the aurora DB instance. Times may vary.

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure: More Details

### Compatible Parameters and Variables

The `Engine`, `EngineVersion`, `DBParameterGroupFamily`, `DBInstanceClass` parameters must be a valid combination.

A managed DB Parameter Group is also created based on the `DBParameterGroupFamily` parameter and `@cluster_parameter_group_parameters` variable. `@cluster_parameter_group_parameters` must contain at least 1 parameter value. Here's a table with tested example values:

Engine | EngineVersion | DBParameterGroupFamily | DBInstanceClass | parameter example
--- | --- | --- | --- | ---
aurora-mysql | 5.7.12 | aurora-mysql5.7 | db.t3.small | time_zone=US/Pacific
aurora-postgresql | 11.6 | aurora-postgresql11 | db.t3.medium | apg\_plan\_mgmt.use\_plan\_baselines=false
aurora | 5.6.10a | aurora5.6 | db.t3.small | time_zone=US/Pacific

Each particular database engine is different and has its own configuration values. Refer to each databases's docs to find out what you can set. Here are some useful docs:

* [aws rds cli tips](https://github.com/boltopspro-docs/rds/blob/master/docs/aws-rds-cli.md)
* [Database Engine Versions for Amazon Aurora PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.20180305.html)
* [How do I change the time zone of my Amazon RDS DB instance?](https://aws.amazon.com/premiumsupport/knowledge-center/rds-change-time-zone/)
* [Choosing the DB Instance Class](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html)

### Specify Number of DB Replicas

You can control the specific number of DB Replicas with the `@db_count` variable. Example:

```ruby
@db_count = 1 # includes the master. So `@db_count = 3` means 1 master and 2 replicas
```

### AutoScaling DB Replicas

You can also enable AutoScaling DB Replicas with parameters. Example:

configs/aurora/params/development.txt:

    EnableAutoscaling=yes
    DisableScaleIn=false
    MaxCapacity=8
    MinCapacity=1
    PredefinedMetricType=RDSReaderAverageCPUUtilization
    ScaleInCooldown=300
    ScaleOutCooldown=300
    TargetValue=50.0

### Using a Custom VPC

You must also provide either `DBSubnetGroupName` or `SubnetIds`. Only provide only one, not both.

By Providing the `SubnetIds` parameter only the blueprint will create a managed [AWS::RDS::DBSubnetGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html) resource in the same custom VPC. Example:

    VpcId=vpc-111
    SubnetIds=subnet-111,subnet-222

### Private Data Subnet

It is recommended to run the database in a private data subnet. The [reference-architecture](https://github.com/boltopspro-docs/reference-architecture) [vpc](https://github.com/boltopspro-docs/vpc) blueprint is has a `PrivateDBSubnetGroup` db subnet group contains the private subnets.  A quick way to get the VPC and subnet values is from the VPC CloudFormation Outputs. Here's an example of the development VPC.

![](https://img.boltops.com/boltopspro/blueprints/vpc/dev-vpc-outputs.png)

If you are not using the reference architecture and you do not have a Db Subnet Group, specifying `SubnetIds` will create a [AWS::RDS::DBSubnetGroup
](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html) for you.

### Security Groups

To assign existing security groups to the RDS database use `VpcSecurityGroupIds`. Example:

    VpcSecurityGroupIds=sg-111,sg-222

If not set, then the blueprint will create and a managed Security Group and assign it to the RDS database.

### Route53 DNS Pretty Host Name

It is recommended to use a Private HostedZone to create a pretty endpoint for the reader and writer DBCluster Endpoints.  Example:

    WriterDnsName=writer.private.
    ReaderDnsName=reader.private.
    HostedZoneName=private.

### Advanced Properties Customizations

Several [AWS::RDS::DBCluster](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Properties that are not configurable with Parameters are configured with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The `@cluster_properties` variable allows you to override any property.  Example:

```ruby
@cluster_properties = {
  port: 3306,
  character_set_name: "utf8_unicode_ci",
}
```

The blueprint is written so that Variables take higher precedence than Parameters.

## Considerations

Using CloudFormation to provision RDS databases has some pros and cons.

* The main pro is that the infrastructure is codified and reproducible.
* The main con is that CloudFormation can **replace** your database entirely, and you'll lose your data!

Depending on the [AWS::RDS::Instance property](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html) property that is changed, CloudFormation **replaces** the database entirely. You have to look for "Update requires: Replacement". This is dangerously easy to forget.

An excellent way to provide a guard rail against accidental replacement is to set the `DBClusterIdentifier` parameter. If you change any property that requires replacement, then the CloudFormation stack update will immediately fail, because RDS won't be able to create another database with the same `DBClusterIdentifier`.

Another technique you can use to prevent an accidental replacement is only to use CloudFormation for the initial provisioning of the database. Afterward, you'll modify the RDS DB with the API or Console only. Yet another way to provide a guard rail is to enable `DeletionProtection=true`. This prevents the current database from being deleted. However, it deletion happens as part of the CloudFormation cleanup step, so it takes longer for the rollback to finish.