1. Deploy [cf-release and diego-release](https://github.com/cloudfoundry-incubator/diego-release) with a micro bosh on AWS.

1. Open the AWS console, and click on EC2.
![aws](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/aws.png)

1. Click on "Instances" in the EC2 home screen.
![ec2](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/ec2.png)

1. Click on "Launch Instance" in the Instances screen.
![instances](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/instances.png)

1. Select Microsoft Windows Server 2012 R2 Base.
![select ami](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/select_ami.png)

1. Select an instance type. It's not especially important what size we choose. In this example, we will choose
m3.xlarge. Then, click "next".
![instance type](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/instance_type.png)

1. Select a network and subnet. The network should the same VPC we have our micro bosh deployed in. 
![instance details](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/instance_details.png)
The subnet should be be the same mask as the ip address of the job running etcd. For example, if we run 
    
    ```
    bosh vms
    ```
, we get
    
    ```
    VMs total: 30
    Deployment `cf-greenhaus1-diego'
    
    Director task 975
    
    Task 975 done
    
    +--------------------+---------+---------------+------------+
    | Job/index          | State   | Resource Pool | IPs        |
    +--------------------+---------+---------------+------------+
    | brain_z1/0         | running | large_z1      | 10.10.5.72 |
    | cc_bridge_z1/0     | running | bridge_z1     | 10.10.5.76 |
    | cell_windows_z1/0  | running | large_z1      | 10.10.5.73 |
    | cell_z1/0          | running | large_z1      | 10.10.5.74 |
    | consul_z1/0        | running | medium_z1     | 10.10.5.11 |
    | etcd_z1/0          | running | medium_z1     | 10.10.5.10 |
    | route_emitter_z1/0 | running | small_z1      | 10.10.5.77 |
    +--------------------+---------+---------------+------------+
    
    VMs total: 7
    ```
. The etcd_z1/0 job has an ip address of 
    
    ```
    10.10.5.10
    ```
, so our subnet should be
    
    ```
    10.10.5.0/24
    ```
. Then, click "Configure Security Group".

1. Create a new security group that allows traffic from anywhere. This is not recommended for production deployments, but is sufficient for development purposes. Then, click "review and launch".
![instance type](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/security_groups.png)

1. Click "Launch".
![launch](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/launch.png)

1. You can select your existing "bosh" key pair, check the check box to acknowledge you have the private key, and click "Launch Instances".
![key pairs](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/key_pair.png)

1. It will take a minute or two for the instance to launch, but when it does you can right click it in your list of instances and select "Get Windows Password".
You can either upload your private key file or copy its contents into the dialog.
![retrieve password](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/retrieve_password.png)
![retrieved password](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/retrieve_password2.png)
Copy this down somewhere.

1. The easiest way to connect to your new Windows instance is using SSH tunnelling. To do so we will need to get the public IP of your bosh director, which you can find by searching for an
instance named "micro". 
![director ip address](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/director_ip.png)
You will also need the private IP of your Windows instance:
![instance ip address](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/instance_ip.png)

1. At the command line, enter `ssh -L 3389:INSTANCE_PRIVATE_IP:3389 vcap@DIRECTOR_IP`, for example `ssh -L 3389:10.10.5.80:3389 vcap@52.20.21.23`.

1. Open Microsoft Remote Desktop and create a new remote desktop with the same properties shown, with the password you retrieved earlier
![remote desktop](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/remote_desktop.png)

1. Double click the remote desktop you just created to connect to it. You may see a certificate warning which you can ignore by clicking "Continue".
![certificate warning](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/certificate_warning.png)
It may take a minute to connect the first time as Windows sets up your user account.

1. Download [this batch script](https://raw.githubusercontent.com/cloudfoundry-incubator/diego-windows-msi/master/scripts/setup.bat) and run it inside the instance 
to enable the required Windows features and configure the DNS settings that the cell will need.
![enable features script](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/enable_features.png)

1. Either [build the MSI](https://github.com/cloudfoundry-incubator/diego-windows-msi#building-the-msi)
   or download one from [the github release page](https://github.com/pivotal-cf/diego-windows-msi/releases).
   Copy it onto the instance, and follow the instructions to [install the
   MSI](https://github.com/cloudfoundry-incubator/diego-windows-msi#installing-the-msi).
  - The ADMIN_USERNAME is "Administrator"
  - The ADMIN_PASSWORD is the same as the one you copied from Amazon
  - The EXTERNAL_IP is the private IP of the instance (in our case 10.10.5.80)
  - The CONSUL_IPS can be retrieved by running `bosh vms` and copying the `consul_z1/0` IP address (in our case "10.10.5.11")
  - The ETCD_CLUSTER can be retrieved by running `bosh vms` and formatting the `etcd_z1/0` (in the diego deployment) IP address as a URL with port 4001 (in our case "http://10.10.5.10:4001")
  - The CF_ETCD_CLUSTER can be retrieved by running `bosh vms` and formatting the `etcd_z1/0` (in the cf deployment) IP address as a URL with port 4001 (in our case "http://10.244.0.42:4001")
  - The MACHINE_NAME can be retrieved by running `hostname` inside the Windows instance (ie "WIN-3Q38P0J78DF")
  - The STACK will be "windows2012R2"
  - The REDUNDANCY_ZONE will be "z1"
  - The LOGGREGATOR_SHARED_SECRET can be retrieved from the cf deployment manifest
![install MSI](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/install_msi.png)

1. If everything has worked correctly, you should now see the following five services running in the Task Manager
![services](https://github.com/cloudfoundry-incubator/diego-windows-msi/blob/master/README_images/services.png)