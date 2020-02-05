# Terraform 0.12 Examples
This repository contains some Terraform 0.12 examples that demonstrate new HCL features and other Terraform enhancements that are being added to Terraform 0.12. Each sub-directory contains a separate example that can be run separately from the others by running `terraform init` followed by `terraform apply`.

These examples have been tested with terraform 0.12 beta1.

The examples are:
1. [First Class Expressions](./first-class-expressions)
1. [For Expressions](./for-expressions)
1. [Dynamic Blocks and Splat Expresions](./dynamic-blocks-and-splat-expressions)
1. [Rich Value Types](./rich-value-types)
1. [New Template Syntax](./new-template-syntax)
1. [Reliable JSON Syntax](./reliable-json-syntax)

## Installing Terraform 0.12 Beta1
1. Determine the location of the Terraform binary in your path. On a Mac of Linux machine, run `which terraform`. On a Windows machine, run `where terraform`.
1. Move your current copy of the Terraform binary to a different location outside your path and remember where so you can restore it after using the Terraform 0.12 beta. Also note the old location.
1. On a Mac or Linux machine, rename the `~/.terraform.d` directory to something like `.terraformd`; on a Windows machine, rename `%USERPROFILE%\terraform.d` to `%USERPROFILE%\terraformd`. This way, you can restore the directory (if anything was in it) after the class.
1. Download the Terraform 0.12 beta1 zip file for your OS from https://releases.hashicorp.com/terraform/0.12.0-beta1.
1. Unzip the file and copy the terraform or terraform.exe binary to the location where your original terraform binary was. If you did not previously have the terraform binary deployed, copy it to a location within your path or edit your PATH environment variable to include the directory you put it in.
1. Download the AWS and template providers for your OS from the [AWS provider](http://terraform-0.12.0-dev-snapshots.s3-website-us-west-2.amazonaws.com/terraform-provider-aws/1.60.0-dev20190216H00-dev/) and the [Template provider](http://terraform-0.12.0-dev-snapshots.s3-website-us-west-2.amazonaws.com/terraform-provider-template/2.1.0-dev20190216H00-dev/) directories.
1. Unzip the provider binaries from the zip files.
1. Create a directory to contain the providers:
  1. On a Mac, run `mkdir -p ~/.terraform.d/plugins/darwin_amd64`.
  1. On a Linux machine, run `mkdir -p ~/.terraform.d/plugins/linux_amd64`.
  1. On a Windows laptop, run `mkdir %USERPROFILE%\terraform.d\plugins\windows_amd64`.
1. Copy the AWS and template provider binaries to the directory you created.
1. Clone this repository to your laptop with the command `git clone https://github.com/hashicorp/terraform-guides.git`.
1. Use `cd terraform-guides/infrastructure-as-code/terraform-0.12-examples` to change into the directory containing the Terraform 0.12 examples.

## Exporting AWS Environment Variables
Several of the examples provision some simple infrastructure into AWS.  You will therefore need to export your AWS keys. On Mac or Linux, do this with these commands:
```
export AWS_ACCESS_KEY_ID=<access_key>
export AWS_SECRET_ACCESS_KEY=<secret_key>
```
On Windows, use `set` instead of `export`.

Some examples use the AWS provider and have the region argument set for it.  You can change th region in the Terraform code if desired.

## Running the Examples
To run the example, do the following unless otherwise instructed:
1. Navigate to the example's directory.
1. Run `terraform init`.
1. Run `terraform apply`.

## First Class Expressions Example
The [First Class Expressions](./first-class-expressions) example creates an AWS VPC, a subnet, a network interface, and an EC2 instance. It illustrates the following new features:
1. Referencing of Terraform variables and resource arguments without interpolation using [First Class Expressions](https://www.hashicorp.com/blog/terraform-0-12-preview-first-class-expressions). (Note that this blog post refers to "attributes" instead of to "arguments".)
1. The need to include `=` when setting the value for arguments of type map or list.

In particular, the Terraform code that creates the VPC refers to the variable called vpc_name directly (`Name = var.vpc_name`) without using interpolation which would have used `${var.vpc_name}`. Other code in this example also directly refers to the id of the VPC (`vpc_id = aws_vpc.my_vpc.id`) in the subnet resource, to the id of the subnet (`subnet_id = aws_subnet.my_subnet.id`) in the network interface resource, and to the id of the network interface (`network_interface_id = aws_network_interface.foo.id`) in the EC2 instance. In a similar fashion, the output refers to the private_dns attribute (`value = aws_instance.foo.private_dns`) of the EC2 instance.

Additionally, the code uses `=` when setting the tags arguments of all the resources to the maps that include the Name key/value pairs.  For example the tags for the subnet are added with:
```
tags = {
  Name = "tf-0.12-example"
}
```
This is required in Terraform 0.12 since tags is an argument rather than a block which would not use `=`. In contrast, we do not include `=` when specifying the network_interface block of the EC2 instance since this is a block.

It is not easy to distinguish blocks from arguments of type map when looking at pre-0.12 Terraform code. But if you look at the documentation for a resource, all blocks have their own sub-topic describing the block. So, there is a [Network Interfaces](https://www.terraform.io/docs/providers/aws/r/instance.html#network-interfaces) sub-topic for the network_interface block of the aws_instance resource, but there is no sub-topic for the tags argument of the same resource.

For more on the difference between arguments and blocks, see [Arguments and Blocks](https://www.terraform.io/docs/configuration/syntax.html#arguments-and-blocks).

For more on expressions in general, see [Expressions](https://www.terraform.io/docs/configuration/expressions.html).

## For Expressions Examples
The [For Expressions](./for-expressions) example illustrates how the new [For Expression](https://www.terraform.io/docs/configuration/expressions.html#for-expressions) can be used to iterate across multiple items in lists. It does this for several outputs, illustrating the usefulness and power of the **for** expression in several ways.  We use two tf files in this example:
1. main.tf creates a VPC, subnet, and 3 EC2 instances and then generates outputs related to the DNS and IP addresses of the EC2 instances.
1. lists-and-maps-with-for.tf shows how the **for** expression can be used inside lists and maps.

We first generate outputs that give the list of private DNS addresses for the 3 EC2 instances in several ways, first using the **old splat syntax**:
```
output "private_addresses_old" {
  value = aws_instance.ubuntu.*.private_dns
}
```

We also use the new [Splat Espression](https://www.hashicorp.com/blog/terraform-0-12-generalized-splat-operator) (`[*]`):
```
output "private_addresses_full_splat" {
  value = [ aws_instance.ubuntu[*].private_dns ]
}
```

For more on splat expressions, see [Splat Expressions](https://www.terraform.io/docs/configuration/expressions.html#splat-expressions).

We next use the new **for** expression:
```
output "private_addresses_new" {
  value = [
    for instance in aws_instance.ubuntu:
    instance.private_dns
  ]
}
```
All three of these give an output like this (with different output names):
```
private_addresses_new = [
  "ec2-54-159-217-16.compute-1.amazonaws.com",
  "ec2-35-170-33-78.compute-1.amazonaws.com",
  "ec2-18-233-162-38.compute-1.amazonaws.com",
]
```

When creating the EC2 instances, we only assign a public IP to one of them by using the conditional operator like this: `associate_public_ip_address = ( count.index == 1 ? true : false)`

We then use the conditional operator with lists inside the **for** expression in an output to show all the private and public IPs of the 3 instances.  We do this in two ways, using the list() interpolation and using brackets.

This is the version with the list() interpolation:
```
output "ips_with_list_interpolation" {
  value = [
    for instance in aws_instance.ubuntu:
    (instance.public_ip != "" ? list(instance.private_ip, instance.public_ip) : list(instance.private_ip))
  ]
}
```

This is the version with brackets:
```
output "ips_with_list_in_brackets" {
  value = [
    for instance in aws_instance.ubuntu:
    (instance.public_ip != "" ? [instance.private_ip, instance.public_ip] : [instance.private_ip])
  ]
}
```

Both of these give outputs like:
```
ips = [
  [
    "172.16.10.218",
  ],
  [
    "172.16.10.199",
    "34.201.169.46",
  ],
  [
    "172.16.10.250",
  ],
]
```
Note that ips consists of 3 lists, of which only the second has two items because only the second EC2 instance has a public IP.

In the lists-and-maps-with-for.tf code, we demonstrate the use of the **for** expression to convert a list of lower-case letters to upper case. We first do this in a list:
```
output "upper-case-list" {
  value = [for l in var.letters: upper(l)]
}
```
and then in a map:
```
output "upper-case-map" {
  value = {for l in var.letters: l => upper(l)}
}
```
The first gives the output
which gives the output:
```
upper-case-list = [
  "A",
  "B",
  "C",
]
```
while the second gives:
```
upper-case-map = {
  "a" = "A"
  "b" = "B"
  "c" = "C"
}
```

## Dynamic Blocks and Splat Expressions
The [Dynamic Blocks and Splat Expresions](./dynamic-blocks-and-splat-expressions) example shows how the [dynamic blocks](https://www.terraform.io/docs/configuration/expressions.html#dynamic-blocks) can be used to dynamically create multiple instances of a block within a resource and how [splat expressions](https://www.terraform.io/docs/configuration/expressions.html#splat-expressions) (`[*]`) can now be used to iterate across those blocks. Recall that the old splat expression (`.*`) could only iterate across top-level attributes of a resource.

In this example, we create an AWS security group with 2 dynamically generated ingress blocks and then create an output that iterates across the ingress blocks to give us both ports.

Here is the variable with the list of ports for the ingress rules:
```
variable "ingress_ports" {
  type        = list(number)
  description = "list of ingress ports"
  default     = [8200, 8201]
}
```

Note that we have explicitly declared the variable to have type `list(number)`. This was not necessary, but it illustrates the expanded types available within Terraform 0.12.

Here is the aws_security_group resource:
```
resource "aws_security_group" "vault" {
  name        = "vault"
  description = "Ingress for Vault"
  vpc_id      = aws_vpc.my_vpc.id

  dynamic "ingress" {
    iterator = port
    for_each = var.ingress_ports
    content {
      from_port   = port.value
      to_port     = port.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

Instead of declaring two separate ingress blocks, we declared a single dynamic ingress block. We also specified the iterator argument as port. (Don't use quotes.) If we had not done that, then we would have used `ingress.value` instead of `port.value` in the code since the default iterator is the label of the dynamic block.

The repetition of dynamic blocks is generated from the `for_each` argument which say what variable to iterate over. The referenced variable should be a list or a map. For lists, you can refer to the "value" of each member. For maps, you can refer to the "key" and "value" of each member. The actual repeated block is generated from the nested `content` block.

Here is the output which uses the new splat expression:
```
output "ports" {
  value = aws_security_group.vault.ingress[*].from_port
  #value = aws_security_group.vault.ingress.*.from_port
}
```

Note that the splat expression occurs after "ingress" and that we used the new version `[*]` instead of the old version `.*` which we show in a comment. You can use either version in Terraform 0.12, but neither would have worked in older versions for nested blocks.

The actual output generated by the apply is:
```
from_ports = [
  8201,
  8200,
]
```

For more on splat expressions, see [Splat Expressions](https://www.terraform.io/docs/configuration/expressions.html#splat-expressions).

## Rich Value Types
The [Rich Value Types](./rich-value-types) example illustrates how the new [Rich Value Types](https://www.hashicorp.com/blog/terraform-0-12-rich-value-types) can be passed into and out of a module. It also shows that entire resources can be returned as outputs of a module.

The top-level main.tf file passes a single map with 4 strings into a module after defining the map as a local value:
```
module "network" {
  source = "./network"
  network_config = local.network_config
}
```
This works because the variable for the module is defined as a map with 4 strings too:
```
variable "network_config" {
  type = object({
    vpc_name = string
    vpc_cidr = string
    subnet_name = string
    subnet_cidr = string
  })
}
```
Inside the module, we refer to the strings with expressions like `var.network_config.vpc_name`.

The module creates an AWS VPC and subnet and then passes those resources back to the root module as outputs:
```
output "vpc" {
  value = aws_vpc.my_vpc
}
output "subnet" {
  value = aws_subnet.my_subnet
}
```
These outputs are then in turn exported by the root module as outputs.

This example also illustrates that we can define a variable as an explicit list with a default value (interface_ips) and assign that to a resource.  We define the variable with:
```
variable "interface_ips" {
  type = list
  description = "IP for network interface"
  default = ["172.16.10.100"]
}
```
Note that we don't use quotes around "list" because types are now first-class values.

We pass the variable into the aws_network_interface.rvt resource with `private_ips = var.interface_ips`. In the past, we would probably have set some string variable like interface_ip to "172.16.10.100" and then used `private_ips = ["${var.interface_ip}"]`. To some extent, we have just shifted the list brackets and quotes to the definition of the variable, but this does allow the specification of the resource to be clearer.

We also create an EC2 instance.

## New Template Syntax
The [New Template Syntax](./new-template-syntax) example illustrates how the new [Template Syntax](https://www.hashicorp.com/blog/terraform-0-12-template-syntax) can be used to support **if** conditionals and **for** expressions inside `%{}` template strings which are also referred to as directives.

The new template syntax can be used inside Terraform code just like the older `${}` interpolations. It can also be used inside template files loaded with the template_file data source provided that you use version 2.0 or higher of the Template Provider.

The code in main.tf creates a variable called names with a list of 3 names and uses the code below to show all of them on their own rows in an output called all_names:
```
output all_names {
  value = <<EOT

%{ for name in var.names ~}
${name}
%{ endfor ~}
EOT
}
```

Note that use of `%{ for name in var.names ~}` to iterated through the names in the names variable, the injection of each name with `${name}`, and the end of the for loop with `%{ endfor ~}`.

The strip markers (`~`) in this example prevent excessive newlines and other whitespaces from being output. We include a blank line before the new template to make sure the first name appears on a new line.

Note that we don't do either of the following which you might have expected:
```
%{ for name in var.names
name
endfor ~}
EOT
}
```
or
```
%{ for name in var.names ~}
%{ name }
%{ endfor ~}
```

Here is a second example, in which we just output one of the 3 names:
```
output "just_mary" {
  value = <<EOT
%{ for name in var.names ~}
%{ if name == "Mary" }${name}%{ endif ~}
%{ endfor ~}
EOT
}
```

As mentioned above, you can also use the new template syntax inside template files if you use version 2.0 or newer of the Template Provider.

We include two templates here:
1. actual_vote.txt that uses the old template syntax with expressions like `${voter}` and `${candidate}`
1. rigged_vote.txt that uses the new template syntax `%{ if candidate == "the Democrate" }the Republican%{ else }${candidate}%{ endif }`.

In the latter, a vote for "the Democrat" is awarded to "the Republican". Other votes are processed as voted.

For more on the new string templates, see [String Templates](https://www.terraform.io/docs/configuration/expressions.html#string-templates).

## Reliable JSON Syntax
The [Reliable JSON Syntax](./reliable-json-syntax) example illustrates how the new [Reliable JSON Syntax](https://www.hashicorp.com/blog/terraform-0-12-reliable-json-syntax) makes life easier for customers using Terraform JSON files instead of HCL files.

As you work through this example, you will need to change the extensions of the files so that only one has the `tf.json` extension at any time. Be sure to change the extension to "tf.json" rather than to "tf".

Also, if you run `terraform init`, you will be prompted to run the `terraform 0.12upgrade` or `terraform validate` command.  So, for this example, just run `terraform validate` for each file.

Let's start by comparing the errors given for this JSON by Terraform 0.11.10 and 0.12:

variable1.tf.json
```
{
  "variable": {
    "example": "foo"
  }
}
```
Running `terraform validate` with Terraform 0.11.10 gives:
```
Error: Error loading /home/ubuntu/test_json/variable1.tf.json: -: "variable" must be followed by a name
```

Terraform 0.12 gives:
```
Error: Incorrect JSON value type

  on variable1.tf.json line 3, in variable:
   3:     "example": "foo"

Either a JSON object or a JSON array is required, representing the contents of
one or more "variable" blocks.
```

The latter is better for two reasons:
1. It gives us the line number for which the error occurred.
1. It tells us that Terraform knew we were defining a Terraform variable and that a variable needs certain things to be legitimate.

While the Terraform 0.11.10 error is telling us we need a name, that is not really true and it is not clear where we would add a name. We could try this:

variable2.tf.json
```
{
  "variable": "name" {
    "example": "foo"
  }
}
```
But this gives us other errors.

We could also try this:

variable3.tf.json
```
{
  "variable": {
    "name": "foo",
    "example": "foo"
  }
}
```
But we again get errors.

Now, let's follow the advice that the first Terraform 0.12 error gave us which was to add a JSON object or array:

variable4.tf.json
```
{
  "variable": {
    "example": {
      "label": "foo"
    }
  }
}
```

Running `terraform validate` with Terraform 0.11.10 gives:
```
Error: Error loading /home/ubuntu/test_json/variable4.tf.json: 1 error(s) occurred:

* variable[example]: invalid key: label
```

Terraform 0.12 gives:
```
Error: Extraneous JSON object property

on variable4.tf.json line 4, in variable.example:
 4:       "label": "foo"

No argument or block type is named "label".
```

Both of these errors are telling us that "label" is not a valid atribute for a variable. If we look at the [variables](https://www.terraform.io/docs/configuration/variables.html) documentation, we see that the valid arguments for a Terraform variable are type, default, and description.

So, let's try changing "label" to "default" so that we have this:

variable-correct.tf.json
```
{
  "variable": {
    "example": {
      "default": "foo"
    }
  }
}
```

Now running `terraform validate` runs without error for both Terraform 0.11.10 and 0.12.

Additionally, we can even now include comments in our JSON code:

variable-with-comment.tf.json
```
{
  "variable": {
    "example": {
      "//": "This property is a comment and ignored",
      "default": "foo"
    }
  }
}
```
While Terraform 0.11.10 complains about this, saying that "//" is an invalid key, Terraform 0.12 accepts and ignores the comment.

In summary, the error messages for parsing Terraform JSON configuraitons are much improved over those that were given in earlier versions of Terraform.


## Cleanup
When done with the Terraform 0.12 beta, delete the terraform.d or .terraform.d directory under your home directory and rename the original version of the directory to what it was before. Replace the Terraform 0.12 binary with the Terraform binary you were previously using in your path.
