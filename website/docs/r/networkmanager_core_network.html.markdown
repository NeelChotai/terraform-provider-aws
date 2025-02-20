---
subcategory: "Network Manager"
layout: "aws"
page_title: "AWS: aws_networkmanager_core_network"
description: |-
  Provides a core network resource.
---

# Resource: aws_networkmanager_core_network

Provides a core network resource.

~> **NOTE on Core Networks and Policy Attachments:** For a given core network, this resource's `policy_document` argument is incompatible with using the [`aws_networkmanager_core_network_policy_attachment` resource](/docs/providers/aws/r/networkmanager_core_network_policy_attachment.html). When using this resource's `policy_document` argument and the `aws_networkmanager_core_network_policy_attachment` resource, both will attempt to manage the core network's policy document and Terraform will show a permanent difference.

## Example Usage

### Basic

```terraform
resource "aws_networkmanager_core_network" "example" {
  global_network_id = aws_networkmanager_global_network.example.id
}
```

### With description

```terraform
resource "aws_networkmanager_core_network" "example" {
  global_network_id = aws_networkmanager_global_network.example.id
  description       = "example"
}
```

### With tags

```terraform
resource "aws_networkmanager_core_network" "example" {
  global_network_id = aws_networkmanager_global_network.example.id

  tags = {
    "hello" = "world"
  }
}
```

### With VPC Attachment

The example below illustrates the scenario where your policy document has static routes pointing to VPC attachments and you want to attach your VPCs to the core network before applying the desired policy document. Set the `create_base_policy` argument to `true` if your core network does not currently have any `LIVE` policies (e.g. this is the first `terraform apply` with the core network resource), since a `LIVE` policy is required before VPCs can be attached to the core network. Otherwise, if your core network already has a `LIVE` policy, you may exclude the `create_base_policy` argument.

```terraform
resource "aws_networkmanager_global_network" "example" {}

data "aws_networkmanager_core_network_policy_document" "example" {
  core_network_configuration {
    asn_ranges = ["65022-65534"]

    edge_locations {
      location = "us-west-2"
    }
  }

  segments {
    name = "segment"
  }

  segment_actions {
    action  = "create-route"
    segment = "segment"
    destination_cidr_blocks = [
      "0.0.0.0/0"
    ]
    destinations = [
      aws_networkmanager_vpc_attachment.example.id,
    ]
  }
}

resource "aws_networkmanager_core_network" "example" {
  global_network_id  = aws_networkmanager_global_network.example.id
  create_base_policy = true
}

resource "aws_networkmanager_core_network_policy_attachment" "example" {
  core_network_id = aws_networkmanager_core_network.example.id
  policy_document = data.aws_networkmanager_core_network_policy_document.example.json
}

resource "aws_networkmanager_vpc_attachment" "example" {
  core_network_id = aws_networkmanager_core_network.example.id
  subnet_arns     = aws_subnet.example[*].arn
  vpc_arn         = aws_vpc.example.arn
}
```

## Argument Reference

The following arguments are supported:

* `description` - (Optional) Description of the Core Network.
* `base_policy_region` - (Optional) The base policy created by setting the `create_base_policy` argument to `true` requires a region to be set in the `edge-locations`, `location` key. If `base_policy_region` is not specified, the region used in the base policy defaults to the region specified in the `provider` block.
* `create_base_policy` - (Optional) Specifies whether to create a base policy when a core network is created or updated. A base policy is created and set to `LIVE` to allow attachments to the core network (e.g. VPC Attachments) before applying a policy document provided using the [`aws_networkmanager_core_network_policy_attachment` resource](/docs/providers/aws/r/networkmanager_core_network_policy_attachment.html). This base policy is needed if your core network does not have any `LIVE` policies (e.g. a core network resource created without the `policy_document` argument) and your policy document has static routes pointing to VPC attachments and you want to attach your VPCs to the core network before applying the desired policy document. Valid values are `true` or `false`. Conflicts with `policy_document`. An example of this Terraform snippet can be found [above](#with-vpc-attachment). An example of a base policy created is shown below. The region specified in the `location` key can be configured using the `base_policy_region` argument. If `base_policy_region` is not specified, the region defaults to the region specified in the `provider` block. This base policy is overridden with the policy that you specify in the [`aws_networkmanager_core_network_policy_attachment` resource](/docs/providers/aws/r/networkmanager_core_network_policy_attachment.html).

```json
{
  "version": "2021.12",
  "core-network-configuration": {
    "asn-ranges": [
      "64512-65534"
    ],
    "edge-locations": [
      {
        "location": "us-east-1"
      }
    ]
  },
  "segments": [
    {
      "name": "segment",
      "description": "base-policy"
    }
  ]
}
```

* `global_network_id` - (Required) The ID of the global network that a core network will be a part of.
* `policy_document` - (Optional, **Deprecated** use the [`aws_networkmanager_core_network_policy_attachment`](networkmanager_core_network_policy_attachment.html) resource instead) Policy document for creating a core network. Note that updating this argument will result in the new policy document version being set as the `LATEST` and `LIVE` policy document. Refer to the [Core network policies documentation](https://docs.aws.amazon.com/network-manager/latest/cloudwan/cloudwan-policy-change-sets.html) for more information. Conflicts with `create_base_policy`.
* `tags` - (Optional) Key-value tags for the Core Network. If configured with a provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block) present, tags with matching keys will overwrite those defined at the provider-level.

## Timeouts

[Configuration options](https://developer.hashicorp.com/terraform/language/resources/syntax#operation-timeouts):

* `create` - (Default `30m`)
* `delete` - (Default `30m`)
* `update` - (Default `30m`)

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `arn` - Core Network Amazon Resource Name (ARN).
* `created_at` - Timestamp when a core network was created.
* `edges` - One or more blocks detailing the edges within a core network. [Detailed below](#edges).
* `id` - Core Network ID.
* `segments` - One or more blocks detailing the segments within a core network. [Detailed below](#segments).
* `state` - Current state of a core network.
* `tags_all` - A map of tags assigned to the resource, including those inherited from the provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block).

### `edges`

The `edges` configuration block supports the following arguments:

* `asn` - ASN of a core network edge.
* `edge_location` - Region where a core network edge is located.
* `inside_cidr_blocks` - Inside IP addresses used for core network edges.

### `segments`

The `segments` configuration block supports the following arguments:

* `edge_locations` - Regions where the edges are located.
* `name` - Name of a core network segment.
* `shared_segments` - Shared segments of a core network.

## Import

`aws_networkmanager_core_network` can be imported using the core network ID, e.g.

```
$ terraform import aws_networkmanager_core_network.example core-network-0d47f6t230mz46dy4
```
