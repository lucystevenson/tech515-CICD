# FQG

## Topic - Setting up an auto scaling group with AWS


Introduce topic but don't teach
Talk about what we have done with that topic
What did we learn
How it can be used in the work place

Decide as a group if we're using powerpoint


## My Course Notes

![alt text](image-2.png)

## Auto Scaling
### Types of Scaling
![alt text](image-3.png)

- **Vertical** up and down
  - Step 1 You go on instance settings and select a new size processor for you virtual machine
  - Step 2 Move the workload to new virtual machine
  - Step 3 Delete old instance
- **Horizontal** out and in
  -   Manually add more copies of vm – not necessarily copies
  -   Automatically auto-scaling
Often duplicated instances

Other Cloud Equivalents:
- Azure  - Azure vm scale sets
- GCP - Instance group (Managed Instance Group mig)

Next Steps - horizontal scaling for our Sparta app

To make the horizontal scaling **manually**, we can increase number of instances when we create an instance

To make the horizontal scaling **automatic**, use Aws auto scaling group
When the average across the machines meets certain threshold, it will scale out, otherwise scale in

### Setting up AWS Auto Scaling Group with high availability (HA) and scalability (SC)

- **Launch template** 
    - AMI app data is stored here 
- **Auto Scaling Group** 
  - The auto scaling group will use the launch template to create new vm
  - Decides availability zones and spreads them across them – it names all the instances for us
  - 2 benefits: High Availability (HA)/redundancy and Scalability (Scalability)
  - We have redundancy by splitting vm across availability zones and having a minimum of 2 vm
  - If we stop the 2 instances we create using asg, the auto scaling groups will see instances as unhappy as not getting 200 port, so it will create 2 more as this is the minimum we set
  - If we delete our asg instances, Auto Scaling Group replaces them as the instance **will fail the health check as the health check expects a response code 200 (HTTP OK)**. The instance will also be replaced if it fails the EC2 status checks.

- **Target group**
  - includes all vm available to the load balancer – this is constantly updated
- **Load balancer**
  - distributes traffic to healthy instances – traffic comes in through load balancer!!
  - Helps to make asg highly available
  - >Opening app through Load Balancer: EC2>Aut Scaling Groups>Integrations>Load Balncing> click load balancer link>click on DNS name link

![alt text](image-4.png)
![alt text](image-4.png)

**Step 1** create app vm

**Step 2** create app AMI

**Step 3** create launch template

>AWS>EC2>Launch Instance>Launch Instance>name:tech515-lucy-for-asg-sparta-app-lt>select auto scaling group launch template>set ami to app ami> select t3.micro>add key pair name>add security group>add user data for app to run

![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-8.png)

**Step 4** test launch template

**Step 5** create AWS auto scaling group

>EC2>Auto Scaling Group>create>follow steps 1-7 below
1.	tech515-lucy-app-asg and add launch template
2.	default vpc

![alt text](image-9.png)

3.	
 
![alt text](image-10.png)
 
![alt text](image-11.png)

![alt text](image-12.png)

If app is not running and it does a port check on 80 – it comes up with bad gateway – it will come up with bad health check – grace period needs to be long enough to get the app running

4.	warm up 180 secs

![alt text](image-13.png)
 
5.	no changes
6. add tag key: Name Value:tech515-lucy-app-asg-HA-SC
 
![alt text](image-14.png)

7. review

### Order to delete Auto Scaling Group from a users perspective
1.	Delete the load balancer – first to stop traffic going through the load balancer - load balancer distributes traffic to healthy instances – traffic comes in through load balancer!!
2.	Delete the target group (if the load balancer is gone already it will say None associated in the Load balancer column)
3.	Delete the ASG

What should you have in place?
Have new one up and running and switch over the traffic to the new version before switching off the old vm


## Types of scaling

### **Vertical** up and down
  - Step 1 You go on instance settings and select a new size processor for you virtual machine
  - Step 2 Move the workload to new virtual machine
  - Step 3 Delete old instance
### **Horizontal** out and in
  -   Manually add more duplicates of vm – not necessarily copies

Other Cloud Equivalents:
- Azure  - Azure vm scale sets
- GCP - Instance group (Managed Instance Group mig)

## Horizontal Auto scaling manually

To make the horizontal scaling **manually**, we can increase number of instances when we create an instance

## Horizontal Auto scaling automatically

### How?
- Set up an AWS Auto Scaling Group

### Components


![alt text](image-4.png)

#### **Launch template** 
  - Stores AMI app data
#### **Auto Scaling Group** 
  - Uses the launch template to create new VMs
  - Decides availability zones and spreads VMs across them – it names all the instances for us
  - We have redundancy by splitting vm across availability zones and having a minimum of 2 vm
  - If we delete our asg instances, Auto Scaling Group replaces them as the instance **will fail the health check as the health check expects a response code 200 (HTTP OK)**. The instance will also be replaced if it fails the EC2 status checks.

#### **Target group**
  - includes all vm available to the load balancer – this is constantly updated
#### **Load balancer**
  - distributes traffic to healthy instances – traffic comes in through load balancer!!
  - Helps to make asg highly available
DNS name link

### ASG Key Notes
- Uses launch template, which stores AMI app data, to make VMs and then creates availability zones to spread VMs across
- The load balancer then distributes the traffic to healthy instances

## Auto Scaling Group Benefits

- **high availability (HA)**
  - if the instance fails the health check it will be replaced by the ASG
  - if Availability Zone fails the vm will keep working as vm are split across 3 different availability zones a b c
- **scalability (SC)**
  - makes it easy to scale an application up or down as needed by automatically adding more resources when demand increases
  - when the average across the machines meets certain threshold, it will scale out, otherwise scale in'


talk about app and front page

test environment so populated with dummy data

## Presentation Notes

split into high availabilit and high scalability and use a diagram
do not have anything on diagram that you haven't explained

2 topics
How to deploy the Sparta app on AWS with high availability?

- **high availability (HA)**
  - if the instance fails the health check it will be replaced by the ASG
  - if Availability Zone fails the vm will keep working as vm are split across 3 different availability zones a b c

- **scalability (SC)**
  - makes it easy to scale an application up or down as needed by automatically adding more resources when demand increases
  - when the average across the machines meets certain threshold, it will scale out, otherwise scale in'

Introduce topic but don't teach
Talk about what we have done with that topic
What did we learn
How it can be used in the work place

Topic: How to deploy the Sparta app on AWS with high scalability?

Presentation:
v1

I'm going to talk about how we used auto scaling groups on AWS to make our app highly scalable

To help you understand the benefit of an auto scaling group, I'm going to show you what happens when we don't have one

Let's say we have one virtual machine that runs the Sparta app and two people are currently accessing the app on their laptops - the traffic from these two people is distributed to this one virtual machine

The Sparta app runs on virtual machines in AWS, and users access it through their web browsers. Their laptops send requests to the app, and the virtual machines respond

if we suddenly had an increase in the number of people wanting to use our website, we would have loads of traffic distributed to this one virtual machine and it might become overloaded, so the app users may find the website loads really slowly

now I'm going to show you what would happen when we have an auto scaling group

if we had an increase in the number of people wanting to go on our app like before, an auto scaling group would detect this increase in traffic to our app and automatically scale out by adding how ever many virtual machines are needed to handle the traffic

the traffic is now split between these virtual machines and no user will see a loading screen

the same goes if the traffic drops to the two users. the auto scaling group will detect this and scale in by reducing the number of virtual machines, as the extra one is no longer needed

It’s also important to note that virtual machines cost money to run. By automatically removing virtual machines when they’re no longer needed, this helps reduce unnecessary costs.

In conclusion, an Auto Scaling Group handles traffic changes for us by automatically adding however many virtual machines are needed. This improves our app performance and optimises costs without manual intervention

And now Desi is going to talk about VPCs in AWS

feedback
- Use I not we
- Past tense titles
- Phrase it more like what I did and how it helped our app
- As you can see on the rhs the asg reacts to this increase in traffic by...
- Add circles to diagrams when talking about certain bits

The Sparta app runs on virtual machines in AWS, and users access it through their web browsers. Their laptops send requests to the app, and the virtual machines respond

v2
I'm going to talk about how I used an auto scaling groups on AWS to make my Sparta app highly scalable

To help you understand the benefit of an auto scaling group, I'm going to show you what happened when I didn't have one

I had 1 virtual machine running my app and let's say I have two people
accessing the app on their laptops through their web browsers - the traffic from these two people is distributed to this one virtual machine

if I suddenly had an increase in the number of people wanting to use my app, loads of traffic would be distributed to this one virtual machine and it may become overloaded. as a result the app would load really slowly

now I'm going to show you what happened when I had an auto scaling group

We set up our auto scaling group to start with 2 virtual machines

if we had an increase in the number of people using our app like before, my auto scaling group would detect this increase in traffic and handle it for me by automatically scaling out and adding how ever many virtual machines were needed to handle this traffic

so traffic is now split between these virtual machine s and no user will see a loading screen

the same goes if the traffic drops to the two users. my auto scaling group would detect this and scale in by reducing the number of virtual machines, as the extra one is no longer needed

It’s also important to note that virtual machines cost money to run. By automatically removing virtual machines when they’re no longer needed, this helps reduce unnecessary costs.

In conclusion, my Auto Scaling Group handled traffic changes for me by automatically adding and removing virtual machines. This improved my app performance and by removing unused virtual machines money is being saved as it costs to run each virtual machine. 

And now Desi is going to talk about VPCs in AWS


🍽️