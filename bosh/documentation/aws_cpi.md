#AWS CPI Implementation #
AWS CPI is an implementation of BOSH CPI. It allows BOSH to interface with AWS using the AWS Ruby Library. In the sections below we outline the implementation details for each of the method in CPI interface.

##Constants and Assumptions ##

Following constants are used and assumptions made about the parameters used to access AWS Services.
	
	DEFAULT_MAX_RETRIES = 2
    # default availability zone for instances and disks
    DEFAULT_AVAILABILITY_ZONE = "us-east-1a"
    DEFAULT_EC2_ENDPOINT = "ec2.amazonaws.com"
    METADATA_TIMEOUT = 5 # in seconds
    DEVICE_POLL_TIMEOUT = 60 # in seconds
    MAX_TAG_KEY_LENGTH = 127
    MAX_TAG_VALUE_LENGTH = 255
    
##Initialize ##

Implementation of `def initialize(options)` method. In this method the AWS Client and Registry Client are initialized.

1. Validate the `options` passed to this method. Checks for Property `aws` and `registry` in `options` hash set.
2. In the second step the parameters are extracted from `options` object to populate the following properties 
	+ `@aws_params`	
3. Then `AWS::EC2::Client` object is created to interface with AWS EC2 Service Endpoint
4. Instantiate `Registry_Client` using the parameters `registry_endpoint`, `registry_user` and `registry_password`.

Figure below shows the flow of control.

![aws_cpi_initialize](https://raw.github.com/rajdeepd/oss-docs/master/bosh/documentation/aws_images/aws_cpi_initialize.png)

##Create Stemcell ##

Implementation of method `create_stemcell(image_path, cloud_properties)`
Steps outlined below are the flow control implemented to extract and upload kernel image, ramdisk and stem cell image.

1. Create and mount new EBS volume (2GB default)
    * Get the disk size from cloud_properties
    * Call `create_disk(disk_size, current_instance_id)` method to create a disk
       * If size  < 1024 or size > 1024*1000 throw error
       * If `instance_id` is not null call `@ec2.instances[instance_id]` to get `instance` object and the `availability_zone`.
       * Else set the `availability_zone` to default
       * Create `volume_params` object with parameters `:size` and `:availability_zone`
       * Call `@ec2.volumes.create(volume_params)`
2. Get the volume and instance objects from the EC2 client `@ec2`
          
    	  volume = @ec2.volumes[volume_id]
        instance = @ec2.instances[current_instance_id]
3.  Attach ebs volume `volume` to `instance`
4. Copy the StemCell Image to the `ebs_volume`
5. Take a Snapshot of the volume and instantiate the image `@ec2.images.create(params)`
6
Figure below shows the flow control for the method `create_stemcell(image_path, cloud_properties)`

![aws_cpi_createstemcell](https://raw.github.com/rajdeepd/oss-docs/master/bosh/documentation/aws_images/aws_cpi_createstemcell.png)

##Delete Stemcell



##Create VM ##

1. Get the `security_groups`
2. Get images from `@ec2` client for the `stemcell_id`
3. Populate `instance_params`
4. Call `create_instance` on `@ec2` client.
5. Call configure `network_configurator` 
6. Setup initial agent settings
7. Update `@registry` with the agent settings. registry is one of the Bosh Components.
![aws_cpi_create_vm](https://raw.github.com/rajdeepd/oss-docs/master/bosh/documentation/aws_images/aws_cpi_create_vm.png)
    
##Delete VM ##




##Create Disk ##



##Delete Disk##


