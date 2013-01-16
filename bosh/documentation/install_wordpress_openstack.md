#Installing Wordpress

###Prerequisites:-


  1. Install the OpenStack. Please 
<a href="https://github.com/ganeshjitta/oss-docs/tree/master/bosh/documentation/install_openstack.md">Click here</a> to know how to install.</li>
  2. Install the Micro BOSH. Please
<a href="https://github.com/ganeshjitta/oss-docs/tree/master/bosh/documentation/install_microbosh_openstack.md">Click here</a> to know how to install.</li>


###Step 1 : Download the latest BOSH stem cell for OpenStack

   root@inception-vm:/home/ubuntu/stemcells/# bosh download public stemcell bosh-stemcell-openstack-0.6.7.tgz


###Step 2 : Upload it to your micro BOSH instance

    root@inception-vm:/home/ubuntu/stemcells/# bosh upload stemcell bosh-stemcell-openstack-0.6.7.tgz

###Step 3 : Git clone the BOSH sample release

    root@inception-vm:/home/ubuntu/releases/# git clone git://github.com/cloudfoundry/bosh-sample-release.git

    root@inception-vm:/home/ubuntu/releases/# cd bosh-sample-release


###Step 4 : Upload the release to micro BOSH

    root@inception-vm:/home/ubuntu/releases/bosh-sample-release#  bosh upload release releases/wordpress-1.yml

###Step 5 : Prepare the deployment manifest file for OpenStack wordpress

Create Manifest File

    root@inception-vm:/home/ubuntu/deployments/# vim wordpress-openstack.yml

Copy the below content and paste it in wordpress-openstack.yml




    ---
    name: wordpress
    director_uuid: f6aa0e16-1165-4193-8653-d2cdb1083a30

    release:
     name: wordpress
     version: latest

    compilation:
     workers: 3
     network: default
     cloud_properties:
        instance_type: m1.small

    # this section describes how updates are handled
    update:
     canaries: 1
     canary_watch_time: 60000
     update_watch_time: 60000
     max_in_flight: 4
     max_errors: 1

    networks:
    - name: default
     type: dynamic
     cloud_properties:
        security_groups:
          - default

    cloud:
     plugin: openstack
     properties:
        openstack:
         auth_url: http://10.0.0.4:5000/v2.0/tokens
         username: admin
         api_key: f00bar
         tenant: admin
         default_key_name: admin-keypair
               default_security_groups: ["default"]
         private_key: /root/.ssh/admin-keypair.pem


    resource_pools:
    - name: medium
     stemcell:
        name: bosh-stemcell
        version: 0.6.7
     network: default
     size: 6
     cloud_properties:
        instance_type: m1.small

    jobs:
     - name: mysql
        template: mysql
        instances: 1
        resource_pool: medium
        persistent_disk: 2048
        networks:
         - name: default
           dynamic_ips:
           - 192.168.22.36
     - name: nfs
        template: debian_nfs_server
        instances: 1
        resource_pool: medium
        persistent_disk: 2048
        networks:
         - name: default
           dynamic_ips:
           - 192.168.22.37
     - name: wordpress
        template: wordpress
        instances: 1
               resource_pool: medium
        persistent_disk: 4096
        networks:
         - name: default
           dynamic_ips:
           - 192.168.22.38
     - name: nginx
        template: nginx
        instances: 1
        resource_pool: medium
        persistent_disk: 2048
        networks:
         - name: default
           dynamic_ips:
           - 192.168.22.39

    properties:
     nginx:
        workers: 1
     wordpress:
        admin: foo@bar.com
        port: 8008
        servers:
         - 192.168.22.38
        servername: 192.168.22.39
        db:
         name: wp
         user: wordpress
         pass: w0rdpr3ss
        auth_key: random key
        secure_auth_key: random key
        logged_in_key: random key
        nonce_key: random key
        auth_salt: random key
        secure_auth_salt: random key
        logged_in_salt: random key
               nonce_salt: random key
     mysql:
        address: 192.168.22.36
        port: 3306
        password: rootpass
     nfs_server:
        address: 192.168.22.37
        #network: 107.21.118.223/255.255.255.255
     debian_nfs_server:
        no_root_squash: true


Note:-

 1. Replace Only the red colored values with actual ones.
 2. Generate hashed password for f00bar.
 3. Replace the password with hashed password.

 
###Step 6 : Deploy wordpress

   root@inception-vm:/home/ubuntu/deployments/# bosh deployment wordpress-openstack.yml

Output will be:

Deployment set to `/home/ubuntu/deployments/wordpress-openstack.yml'

    root@inception-vm:/home/ubuntu/deployments/# bosh deploy


Output will be:

    Getting deployment properties from director...
    Unable to get properties list from director, trying without it...
    Compiling deployment manifest...
    Cannot get current deployment information from director, possibly a new deployment
    Please review all changes carefully
    Deploying `wordpress-openstack.yml' to `microbosh-openstack' (type 'yes' to continue): yes

    Director task 18

    Preparing deployment
     binding deployment (00:00:00)                    
                                                
    binding releases (00:00:00)                                                                       
    binding existing deployment (00:00:00)                                                            
    binding resource pools (00:00:00)                                                                 
    binding stemcells (00:00:00)                                                                      
    binding templates (00:00:00)                                                                      
    binding properties (00:00:00)                                                                     
    binding unallocated VMs (00:00:00)                                                                
    binding instance networks (00:00:00)                                                              
    Done                    9/9 00:00:00                                                                

    Preparing package compilation
     finding packages to compile (00:00:00)                                                            
    Done                    1/1 00:00:00                                                                

    Preparing DNS
     binding DNS (00:00:00)                                                                            
    Done                    1/1 00:00:00                                                                

    Creating bound missing VMs
     medium/0 (00:01:13)                                                                               
     medium/1 (00:01:27)                                                                               
     medium/3 (00:01:43)                                                                               
     medium/2 (00:01:50)                                                                               
    Done                    4/4 00:01:50                                                                

    Binding instance VMs
     nfs/0 (00:00:01)                                                                                 
     wordpress/0 (00:00:01)                                                                            
    nginx/0 (00:00:01)                                                                                
    mysql/0 (00:00:01)                                                                                
    Done                    4/4 00:00:01                                                                

    Preparing configuration
     binding configuration (00:00:00)                                                                  
    Done                    1/1 00:00:00                                                                

    Updating job mysql
     mysql/0 (canary) (00:01:20)                                                                       
    Done                    1/1 00:01:20                                                                

    Updating job nfs
     nfs/0 (canary) (00:01:33)                                                                         
    Done                    1/1 00:01:33                                                                

    Updating job wordpress
     wordpress/0 (canary) (00:01:26)                                                                   
    Done                    1/1 00:01:26                                                                

    Updating job nginx
     nginx/0 (canary) (00:01:25)                                                                       
    Done                    1/1 00:01:25                                                                

    Refilling resource pools
     medium/0 (00:01:13)                                                                               
    medium/1 (00:01:28)                                                                               
    Done                    2/2 00:01:28                                                                

    Task 18 done
    Started        2012-11-20 09:36:26 UTC
    Finished    2012-11-20 09:45:29 UTC
    Duration    00:09:03

    Deployed `wordpress-openstack.yml' to `microbosh-openstack'



