# virtualization
the technology that allows you to run multiple independent computer systems on a single piece of physical hardware.

# Amazon EC2
> a web service that provides resizable compute capacity in the cloud.
>It's essentially a virtual server(instances) in the AWS Cloud.
>EC2 as a virtual computer you can access and control over the internet. Just like a physical computer, it has processing power, memory, storage, and networking capabilities
>EC2 instances run in a virtualized environment within Amazon Web Services (AWS) data centers, securely isolated from other customers. This makes it possible to use enterprise-grade infrastructure without owning physical hardware.
>Amazon EC2 eliminates the challenges associated with traditional on-premises infrastructure, such as high upfront costs, complex setup and maintenance, and limited scalability. By using Amazon EC2, businesses can scale their computing resources up or down efficiently based on their changing needs. This ensures that businesses only pay for the resources they actually use and avoid overprovisioning or underutilizing resources.

# Amazon Machine Image (AMI)
>provides the information required to launch an instance.
>>Instance:a virtual server in the cloud
>An AMI includes:
1. template for the root volume for the instance ,eg :an operating system or an application server with applications
2.Launch permissions :control which AWS accounts can use the AMI to launch instances
3. block device mapping : specifies the volumes to attach to the instance when it is launched

# Instance types
> comprise varying combinations of CPU, memory, storage, and networking capacity.
> give you the flexibility to choose the appropriate mix of resources for your applications
> Each instance type includes one or more instance sizes, allowing you to scale your resources to the requirements of your target workload.
> eg : t3.micro instance type -->2 virtual CPUs and 1 GiB of memory.