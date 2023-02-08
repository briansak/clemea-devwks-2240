## DEVWKS-2240 <br /> Using SecureX orchestration for Automating Public Cloud Incident Response and Remediation

### Lab Environment: https://expo.ciscodcloud.com/61f3vnmo3a3mzoh0gzq0wfeb6
> **Right-click and open in a new INCOGNITO MODE/PRIVATE WINDOW**

<img width="472" alt="image" src="https://user-images.githubusercontent.com/10421515/216669404-59355e96-5906-4227-bb4e-cf8b32eb1ece.png">

_**IaaS API Documentation**_ <br />
AWS: https://docs.aws.amazon.com <br />
GCP: https://cloud.google.com/compute/docs/reference/rest/v1 <br />
Azure: https://docs.microsoft.com/en-us/rest/api/azure <br />

### Creating Your First AWS Activity

> **SHARED ENVIRONMENT ALERT** <br />
> Make sure you uniquely name your workflow when creating!

API Reference: https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html

Create a new activity that will provide the details of an EC2 instance, following the example presented.

1. Create a **New Workflow**

<img width="1440" alt="newworkflow" src="https://user-images.githubusercontent.com/10421515/217500740-16af2de6-3530-4273-ad4a-fea908aac3a9.png">

2. Drag **AWS Service** --> **Generic AWS API Request** to the canvas

<img width="1441" alt="awsactivity" src="https://user-images.githubusercontent.com/10421515/217501771-e2acfc9d-3e66-4637-89ca-b90b76dceeee.png">

3. Name the activity **Query EC2 Instance** in the activity Display Name
4. Override the workflow target with: **AWS_Target**

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173708851-fde409ab-e0ae-43b1-b36a-f3e0054896ad.png">

5. Specify the URL near the bottom of the activity properties with:

```
https://ec2.us-east-1.amazonaws.com/?Action=DescribeInstances&Filter.1.Name=private-ip-address&Filter.1.Value={Value_Below}&Version=2016-11-15
```
Replacing "{Value_Below}" with the IP address associated with your pod in the table below and set the API Method to **GET**.

Pod 1: 172.31.31.233 <br />
Pod 2: 172.31.22.192 <br />
Pod 3: 172.31.19.34 <br />
Pod 4: 172.31.28.79 <br />
Pod 5: 172.31.27.52 <br />
Pod 6: 172.31.31.163 <br />
Pod 7: 172.31.92.122 <br />
Pod 8: 172.31.85.153 <br />
Pod 9: 172.31.91.202 <br />
Pod 10: 172.31.83.165 <br />
Pod 11: 172.31.88.102 <br />
Pod 12: 172.31.84.240 <br />

<img width="589" alt="image" src="https://user-images.githubusercontent.com/10421515/173710456-a0f3eff7-9b4a-4c71-8db8-505aeb3bb501.png">

6. Click on the "Start" element in the workflow and customize the workflow "Display Name" to **Pod # - Query EC2 Instance**

<img width="1440" alt="workflow name" src="https://user-images.githubusercontent.com/10421515/217502201-fa637a5b-1653-41c9-83ac-ffa29bd894c1.png">

7. Validate and Run this workflow near the top of the canvas.

<img width="1440" alt="validate 1" src="https://user-images.githubusercontent.com/10421515/217502626-bd045e0a-58a5-40b5-a70d-5a75c3f0e940.png">

After running this activity, you should see the details of the instance that you queried in the output body.

<img width="1446" alt="first run" src="https://user-images.githubusercontent.com/10421515/217502274-97df6281-d8aa-49d3-80fe-6a09b24a9ced.png">

Next we will extract some data from the returned XML Response using an XPATH Activity.

8. From the **Core** activites list grab and drag the **XPATH Query** and drag it to the canvas.

<img width="1440" alt="xpath" src="https://user-images.githubusercontent.com/10421515/217502908-aa6377d4-192e-4383-86fb-27d0dd0aa845.png">

9. Rename the activity **Extract EC2 Details**

<img width="1440" alt="name xpath" src="https://user-images.githubusercontent.com/10421515/217505564-bdc29914-b961-4755-b7ea-a1b531494388.png">

10. Scroll down on the properties window and find where you will specify **Source XML to Query**.  Click the puzzle piece in the upper-right corner to select a source.

<img width="1440" alt="xml query" src="https://user-images.githubusercontent.com/10421515/217503035-11e3b1d3-ca8c-418a-af54-4f3fe6a7d5d5.png">

11. Choose **Activities --> Query EC2 Instance --> Body** and choose **Save**

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/216678141-756823d2-5271-478b-9473-74489930d613.png">

12. Click the **"+"** next to XPATH Query.  Fill in the following details:

    Property Name:  **Instance ID** <br />
    XPath Query:  **//reservationSet/item/instancesSet/item/instanceId** <br />
    Property Type: **String** <br />

    Click the **"+"** again to add another variable.
    
    Property Name:  **Security Group** <br />
    XPath Query:  **//reservationSet/item/instancesSet/item/groupSet/item/groupName** <br />
    Property Type: **String** <br />

The properties should look like the graphic below.

<img width="1443" alt="xml query 3" src="https://user-images.githubusercontent.com/10421515/217504755-dcdfa1ce-f7f7-4aaf-86ee-d10882468ce9.png">

Next we'll add a simple conditional block to check if one of our variables matches a specific value.

13. Click on the **Logic** tab of the navigation, expand Logic and drag the **Condition Block** to the canvas.

<img width="1440" alt="1" src="https://user-images.githubusercontent.com/10421515/217505730-ba49c1ec-191a-49b5-b9a6-1bfeb3104f48.png">

14. Click on each of larger block and rename it **Is Instance Isolated?** and the smaller block on the left and name it **Yes**.

<img width="1440" alt="2" src="https://user-images.githubusercontent.com/10421515/217506053-aa65d08a-1e88-46c2-bbbd-53f3104eeb85.png">

15. Click on the **Activities** tab of the navigation, locate the **Generic AWS API Request** from the AWS Service section and drag it to the canvas into the **Yes** block.

<img width="1443" alt="3" src="https://user-images.githubusercontent.com/10421515/217506693-e9927e8c-2f74-4a30-8796-f8c0f7c8e537.png">

16. Change the activity name to **Tag EC2 Instance**.

<img width="1440" alt="4" src="https://user-images.githubusercontent.com/10421515/217506872-d8c606c8-73a7-4f05-a54d-d3370931a7f8.png">

17. In the properties of the Tag EC2 Instance, scroll down and supply the following details

13. Click **Validate** again at the top of the window and run the workflow again.

When the workflow runs this time you'll be able to see the output of the second activity that parses the XML response from the first activity and extracts the name of the assigend Security Group for your EC2 instance.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/216681662-1b6034bb-043b-4331-be29-2c50cf1c9245.png">

We will revisit this created workflow later after we initiate our completed incident response workflow.

### Importing a workflow from Github

> **SHARED ENVIRONMENT ALERT** <br />
> Make sure you uniquely name your workflow after importing!

1. Click back to Workflows and choose the **Import** button.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173710672-bd66daf1-d85d-4586-880c-3007e300f821.png">

2. Choose, Import from **Git** --> **DEVWKS-2240** for Git Repository, **sxo-aws-ir** for File Name, **Updated Keys** for Git Version, and finally **Import as a New Workflow** and click **Import**.

<img width="608" alt="image" src="https://user-images.githubusercontent.com/10421515/216683454-c86c3b07-d757-43be-a77a-bae18d4bf574.png">

3. After importing, it will show up as **Copy(1)-AWS Incident Response**.  Open this newly created workflow.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173711264-43a14855-cc98-45f9-85c6-b1c559343c77.png">

4. Open your new workflow and make the following changes: <br />

* Name your workflow **Pod X - AWS Incident Response** replacing the pod number with yours.
<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173711715-1849e6ac-150d-4a34-9e1e-dbb02ea800fe.png">

Pod 1: 172.31.31.233 <br />
Pod 2: 172.31.22.192 <br />
Pod 3: 172.31.19.34 <br />
Pod 4: 172.31.28.79 <br />
Pod 5: 172.31.27.52 <br />
Pod 6: 172.31.31.163 <br />
Pod 7: 172.31.92.122 <br />
Pod 8: 172.31.85.153 <br />
Pod 9: 172.31.91.202 <br />
Pod 10: 172.31.83.165 <br />
Pod 11: 172.31.88.102 <br />
Pod 12: 172.31.84.240 <br />

* Replace the variable for 'observable_value' to match your pod above.
<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173711852-8d854b7a-7017-4b46-b7cd-3f9abcdd5baf.png">

Note the activites in this workflow that automate many of the steps outlined in the AWS EC2 Incident Response Guide.

> * Enables Termination Protection on the instance
> * Sets a restricted Security Group limiting access
> * Removes it from any Auto Scaling Groups
> * Removes it from any Elastic Load Balancers
> * Snapshots connected Elastic Block Storage devices
> * Tags the instance with IR details

5. Run your imported workflow.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173712097-05b9094c-e06f-422f-aadf-c88a004e4e52.png">

6. Return to your previously created workflow **Query EC2 Instance** that pulls instance details and run the workflow again.

7. Click on the **Extract Security Group Name** activty and note that after running the incident response workflow, the Security Group has been changed to move the impacted host to an isolated security group.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/216684838-535b37c8-cb18-49f2-9ef0-380f5b9fbce8.png">

### Integration with SecureX threat response

1. Click **Dashboard** at the top of the SecureX Window to exit out of the Orchestration tool.

2. Expand the **Ribbon** at the bottom of the screen to see created casebooks and incidents for this environment.
3. Find the **DEVWKS-2240 Casebook** Casebook created "By Others" by searching for 'DEVWKS' in the casebook pane.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173712877-c8889a14-392b-4454-8012-03f7b4002c5d.png">

4. Click **Investigate in Threat Response** located on the right-hand side of the Casebook drawer.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173712656-c4778e87-3b3d-4962-99b7-701186aef572.png">

This will open up a new browser window that contains the results of our investigation into one domain and the AWS internal IP addresses from our lab.

Once the enrichment of the observables is complete, you should see your IP address denoted as a 'target' in the resulting graph.

<img width="1398" alt="image" src="https://user-images.githubusercontent.com/10421515/167261531-68659cca-c8b5-4f04-b320-221f9afc36e1.png">

5. Use the drop-down to show the SecureX orchestration response actions that can be ran against this host, including the one you imported and uniquely named.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173713168-a4a55394-ef21-4d51-bb2b-b8939234ca58.png">

**THANK YOU** for attending this DevNet Workshop and learning about SecureX orchestration.  I'll be putting some links and code into the Webex space associated with this session following this session!  Enjoy your time at Cisco Live!
