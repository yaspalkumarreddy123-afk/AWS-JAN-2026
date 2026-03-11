# AWS VPC Cleanup Guide

After practicing, it is important to **delete the AWS resources** to avoid unnecessary charges. Follow these steps in the correct order to clean up all created components.

---

## **1. Delete EC2 Instances**
Before deleting networking components, **terminate EC2 instances** running in the VPC:

1. Go to **AWS Console** → **EC2 Dashboard**.
2. Select the **instances** running in the **Public and Private Subnets**.
3. Click **Actions** → **Instance State** → **Terminate**.
4. Wait until the instances are **terminated**.

---

## **2. Delete NAT Gateway**
1. Navigate to **VPC Dashboard** → **NAT Gateways**.
2. Select the **NAT Gateway** associated with the Public Subnet.
3. Click **Actions** → **Delete NAT Gateway**.
4. Wait for the status to change to **deleted**.

---

## **3. Release the Elastic IP (EIP)**
1. Go to **VPC Dashboard** → **Elastic IPs**.
2. Select the **EIP** that was assigned to the NAT Gateway.
3. Click **Actions** → **Release Elastic IP**.
4. Confirm the release.

---

## **4. Detach and Delete Internet Gateway (IGW)**
1. Navigate to **VPC Dashboard** → **Internet Gateways**.
2. Select the **IGW** attached to your VPC.
3. Click **Actions** → **Detach from VPC**.
4. After detaching, click **Actions** → **Delete Internet Gateway**.

---

## **5. Delete Route Tables**
**Public and private subnets use custom route tables**. Remove them before deleting the subnets.

1. Go to **VPC Dashboard** → **Route Tables**.
2. Select the **custom route tables** (Public-RT and Private-RT).
3. Click **Actions** → **Delete Route Table**.
4. Confirm the deletion.

**Note:** Do **not** delete the **main route table** until the subnets are removed.

---

## **6. Delete Subnets**
1. Navigate to **VPC Dashboard** → **Subnets**.
2. Select **Public-Subnet** and **Private-Subnet**.
3. Click **Actions** → **Delete Subnet**.
4. Confirm the deletion.

---

## **7. Delete the VPC**
1. Navigate to **VPC Dashboard** → **Your VPCs**.
2. Select `MyVPC`.
3. Click **Actions** → **Delete VPC**.
4. Confirm the deletion.

---

## **8. Verify Cleanup**
To ensure **all resources** are deleted, check:
- **EC2 Dashboard** (No running instances)
- **VPC Dashboard** (No unused VPCs)
- **Elastic IPs** (No allocated IPs)
- **NAT Gateways** (None listed)

✅ **Cleanup is complete!** Your AWS account is now free of unnecessary charges.

---

## **Additional Resources**
- [AWS VPC Cleanup Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html)
- [EC2 Instance Lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [AWS Elastic IP Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-eips.html)
