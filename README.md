# IMPLEMENTING GATEWAY LOAD BALANCER WITH FORTIGATE.

In this project, we implemented gateway load balancer with Fortigate.

Implementing a Gateway Load Balancer (GWLB) with FortiGate involves integrating FortiGate appliances with a load balancing solution to scale network security functions transparently, especially for east-west or north-south traffic inspection.

East west traffic means vpc to vpc traffic ,
North-south traffic means vpc to internet traffic.
However, this project is focused on North-South traffic


The objective of this project is to implement GWLB with fortigate, below are the key aws services and resources to achieve the objective.

1. GWLB : 
Gateway Load Balancer (GWLB) is an AWS service that enables you to deploy, scale, and manage virtual appliances like FortiGate firewalls
2. FortiGate: Acts as a security appliance for traffic inspection (firewall etc.).
3. GWLB Endpoint: Connects VPCs to the GWLB. This is deployed in the workload vpc(main vpc)
4. Security VPC: This VPC Hosts the GWLB and the FortiGate instances.
5. SPoke or Main VPC: This vpc host the workload instances or where the applications seat.

This project documentation shows the step taken to implement GWLB with fortigate and the challenges/ blocker faced in the course of the project.

The Architecture below shows the flow:

![image](https://github.com/user-attachments/assets/1f765652-ac6e-4e62-bf42-f263078b2c43)






Steps
Created a two VPCs. one for the workload or application called the main vpc. The other for hosting the GWLB and virtual appliances(Fortigate). This vpc is called the security vpc, as seen in the screen shot below:


![image](https://github.com/user-attachments/assets/3c3e6c93-0e62-4c5c-bad5-d129b6a201e5)



<u>**Security VPC**</u>
1. In the security vpc, two subnets are created. Private and Public.
   The private subnet is where the fortigate appliances and the GWLB are provisioned. While the public subnet is where we provisioned a Jump server so as to be able to
   manage the Fortigate appliances remotely from the internet.

   ![image](https://github.com/user-attachments/assets/53e9c10e-cd64-4fb6-a50a-2f3a67da6da8)



2. Also the fortigate instances are provisioned by navigating to the aws market.

![image](https://github.com/user-attachments/assets/ddbbe05c-abd7-4416-a5ec-d0b35cde4b35)


See below the fortigate instances for inspection. The essence of the jump server, is to be able to manage the fortigate instances remotely throught the internet.
This is because the fortigate is deployed in the private subnet.


![image](https://github.com/user-attachments/assets/4967c299-4ecb-435b-bdb8-781a265b1cca)


The security group on the Fortigate instance permit the ports below:

![image](https://github.com/user-attachments/assets/754fcbe8-1e68-4179-8612-fe1da742afd3)

GENEVE Preserves original source/destination IPs while routing to appliances. and it works with the GWLB on port 6081.



3. In the security vpc, we deployed the GWLB as seen below:

![image](https://github.com/user-attachments/assets/6cd3fc88-0931-4ac4-8c60-8af9a0fab802)

Also a Target group is created, specifying the IP of the Fortigate appliance.

![image](https://github.com/user-attachments/assets/01b6b5fd-a92d-4eeb-8136-81ea73dc446d)





**MAIN VPC**
1. In the main vpc, two subnets were created. Private and public subnet.
The private subnet hosts the instances that are not allowed to access the internet but can't be accessed from the internet.
The public subnet hosts instances that are allowed to access and be accessed from the internet.
Also an internet gateway is created and attahced to the vpc for internet communication or north -south communication

![image](https://github.com/user-attachments/assets/0606bc3f-c13a-4da1-b7ba-a5949c90dcae)


2. In the public subnet, An endpoint service was created for the GWLB endpoint.

   ![image](https://github.com/user-attachments/assets/6d93d1f3-6854-4235-b794-55f4889b043d)

3. In the public subnet, A GWLB endpoint was created.

![image](https://github.com/user-attachments/assets/5a0b0d8b-1495-49e1-9858-5288e196f0f8)

4. The work load instances were created. One in the private subnet and the other in the public subnet.

![image](https://github.com/user-attachments/assets/9c006cb9-e186-44e8-a096-3cfaac5f0285)


   
Now, all services and resources have been created. But to ensure proper flow of traffic in the right path, the route table was created for each subnet as seen below:

**Main-vpc-Public Subnet route table:**
![image](https://github.com/user-attachments/assets/f0ecf367-f58f-4847-b47b-5adc7193b293)
default traffic pointing towards the internet gateway.



**Main-vpc-Private Subnet route table:**

![image](https://github.com/user-attachments/assets/53f3be1e-1320-4575-9b29-b0d581d3c6f4)
default traffic from the instances in the private subnet is routed towards the GWLBe.


*Summary of the flow for North-South traffic.*
1. Instances in the main vpc private subnet when it tries to reach the internet (e.g 8.8.8.8), the traffic should flow in two phase:
   
   Forward traffic: to the GWLBe >>> GWLB >>> Fortigate instance(for inspection).
   
   Backward traffic: Fortigate instance(After inspection) >>> GWLB >>> back to the GWLBe >>> IGW >>> Internet.




## **CHALLENGES/BLOCKER**

In the course of the project, we realized that the instance in the private subnet, couldn't reach the internet. The traffic is hitting the Fortigate appliance for inspection , but couldnt go to the internet. 


see ping to 8.8.8.8 below, 

![image](https://github.com/user-attachments/assets/d73b32ac-d473-4acf-8d56-def23820ec14)





See below the observation on  the Fortigate appliance.

![image](https://github.com/user-attachments/assets/8f69a4b3-b39e-4799-8630-f6e1418927ba)


