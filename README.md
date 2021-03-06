# AutoSpotting

Automatically convert your existing Auto Scaling groups to significantly cheaper spot instances with minimal (often zero) configuration changes.

See [https://github.com/autospotting/autospotting](https://github.com/autospotting/autospotting) for details.

## Using the terraform-aws-autospotting module

* [Sources](#sources)
* [Setting variables](#setting-variables)
* [Using custom binaries](#using-custom-binaries)
  * [From local file](#from-local-file)
  * [From S3](#from-s3)
* [Multiple instances](#multiple-instances)

### Sources

The autospotting module can be used from the Terraform Registry or directly from this repository.

You can use the module with prebuilt binaries from the Terraform Registry:

```hcl
module "autospotting" {
  source  = "cristim/autospotting/aws"
  version = "0.0.9" # this version is subject to change
}
```

You can also use the module from this repository, but you'll also need to provide a reference to the autospotting binaries:

```hcl
module "autospotting" {
  source         = "github.com/autospotting/terraform-aws-autospotting?ref=master"
  lambda_zipname = "lambda.zip"
}
```

### Setting variables

Available variables are defined in the [variables file](variables.tf). To change the defaults, just pass in the relevant variables:

```hcl
module "autospotting" {
  source                                = "github.com/autospotting/terraform-aws-autospotting"
  autospotting_regions_enabled          = "eu*,us*"
  autospotting_min_on_demand_percentage = "33.3"
  lambda_zipname                        = "lambda.zip"
  lambda_memory_size                    = 1024
}
```

Or you can pass them in on the command line:

``` shell
 terraform apply \
   -var autospotting_regions_enabled="eu*,us*" \
   -var autospotting_min_on_demand_percentage="33.3" \
   -var lambda_zipname="lambda.zip" \
   -var lambda_memory_size=1024
```

### Using custom binaries

The `lambda.zip` file can be generated by building it locally. Further instructions are available in the [AutoSpotting repo](https://github.com/AutoSpotting/AutoSpotting/blob/master/CUSTOM_BUILDS.md).

You can also download the latest official nightly build available [here](https://cloudprowess.s3.amazonaws.com/nightly/lambda.zip).

#### From local file

If you store the file locally:

```hcl
module "autospotting" {
  source         = "github.com/autospotting/terraform-aws-autospotting"
  lambda_zipname = "lambda.zip"
}
```

#### From S3

Instead of using a local ZIP file you can refer to the Lambda code in a location in S3:

```hcl
module "autospotting" {
  source           = "github.com/autospotting/terraform-aws-autospotting"
  lambda_s3_bucket = "lambda-releases"
  lambda_s3_key    = "lambda.zip"
}
```

### Multiple instances

You can change the names of the resources terraform will create – or run multiple instances of autospotting that target different ASGs – by using the label variables:

```hcl
module "autospotting_storage" {
  source                              = "github.com/autospotting/terraform-aws-autospotting"
  label_name                          = "autospotting_storage"
  autospotting_allowed_instance_types = "i3.*"
  autospotting_tag_filters            = "spot-enabled=true,storage-optimized=true,"
  lambda_zipname                      = "lambda.zip"
}

module "autospotting_dev_memory" {
  source                              = "github.com/autospotting/terraform-aws-autospotting"
  label_name                          = "autospotting_memory"
  label_environment                   = "dev"
  autospotting_allowed_instance_types = "r5*"
  autospotting_tag_filters            = "spot-enabled=true,memory-optimized=true,environment=dev,"
  lambda_zipname                      = "lambda.zip"
}
```

### Logs or Troubleshooting

To check logs and troubleshoot issues, you can go to the `/aws/lambda/<your_lambda_function_name>` CloudWatch Log group.
