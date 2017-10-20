# terraform-aws-emr-cluster

A Terraform module to create an Amazon Web Services (AWS) Elastic MapReduce (EMR) cluster.

## Usage

```hcl
data "template_file" "emr_configurations" {
  template = "${file("configurations/default.json")}"
}

module "emr" {
  source = "github.com/azavea/terraform-aws-emr-cluster?ref=0.1.0"

  name          = "DatarpocCluster"
  vpc_id        = "vpc-20f74844"
  release_label = "emr-5.8.0"

  applications = [
    "Hadoop",
    "Ganglia",
    "Spark",
    "Zeppelin",
  ]

  configurations = "${data.template_file.emr_configurations.rendered}"
  key_name       = "hector"
  subnet_id      = "subnet-e3sdf343"

  instance_groups = [
    {
      name           = "MasterInstanceGroup"
      instance_role  = "MASTER"
      instance_type  = "m3.xlarge"
      instance_count = 1
    },
    {
      name           = "CoreInstanceGroup"
      instance_role  = "CORE"
      instance_type  = "m3.xlarge"
      instance_count = "1"
      bid_price      = "0.30"
    },
  ]

  bootstrap_name = "runif"
  bootstrap_uri  = "s3://elasticmapreduce/bootstrap-actions/run-if"
  bootstrap_args = []
  log_uri        = "s3://..."

  project     = "Something"
  environment = "Staging"
}
```

## Variables

- `name` - Name of EMR cluster
- `vpc_id` - ID of VPC meant to house cluster
- `release_label` - EMR release version to use (default: `emr-5.8.0`)
- `applications` - A list of EMR release applications (default: `["Spark"]`)
- `configurations` - JSON array of EMR application configurations
- `key_name` - EC2 Key pair name
- `subnet_id` - Subnet used to house the EMR nodes
- `instance_groups` - List of objects for each desired instance group (see section below)
- `bootstrap_name` - Name for the bootstrap action
- `bootstrap_uri` - S3 URI for the bootstrap action script
- `bootstrap_args` - A list of arguments to the bootstrap action script (default: `[]`)
- `log_uri` - S3 URI of the EMR log destination
- `project` - Name of project this cluster is for (default: `Unknown`)
- `environment` - Name of environment this cluster is targeting (default: `Unknown`)

## Instance Group Example

```hcl
[
    {
      name           = "MasterInstanceGroup"
      instance_role  = "MASTER"
      instance_type  = "m3.xlarge"
      instance_count = 1
    },
    {
      name           = "CoreInstanceGroup"
      instance_role  = "CORE"
      instance_type  = "m3.xlarge"
      instance_count = "1"
      bid_price      = "0.30"
    },
]
```

## Outputs

- `id` - The EMR cluster ID
- `name` - The EMR cluster name
- `master_public_dns` - The EMR master public FQDN
- `master_security_group_id` - Security group ID of the master instance/s
- `slave_security_group_id` - Security group ID of the slave instance/s
