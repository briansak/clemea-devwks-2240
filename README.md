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

2. From the activities tab, drag the **AWS Service** --> **Generic AWS API Request** actiivity to the canvas.

<img width="1441" alt="awsactivity" src="https://user-images.githubusercontent.com/10421515/217501771-e2acfc9d-3e66-4637-89ca-b90b76dceeee.png">

3. Name the activity **Query EC2 Instance** in the activity Display Name and Override the workflow target with: **AWS_Target** in the activity properties.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173708851-fde409ab-e0ae-43b1-b36a-f3e0054896ad.png">

4. Specify the URL near the bottom of the activity properties with:

```
https://ec2.us-east-1.amazonaws.com/?Action=DescribeInstances&Filter.1.Name=private-ip-address&Filter.1.Value={Value_Below}&Version=2016-11-15
```
Replacing "{Value_Below}" with the IP address associated with your pod in the table below and set the API Method to **GET**.

|     |     |
| --- | --- |
| Pod 1: 172.31.22.192 | Pod 11: 172.31.4.177 |
| Pod 2: 172.31.19.34 | Pod 12: 172.31.9.192 |
| Pod 3: 172.31.28.79 | Pod 13: 172.31.2.170 |
| Pod 4: 172.31.7.175 | Pod 14: 172.31.6.32 |
| Pod 5: 172.31.11.48 | Pod 15: 172.31.11.139 |
| Pod 6: 172.31.14.141 | Pod 16: 172.31.5.13 |
| Pod 7: 172.31.9.110 | Pod 17: 172.31.2.231 |
| Pod 8: 172.31.5.98 | Pod 18: 172.31.4.105 |
| Pod 9: 172.31.6.146 | Pod 19: 172.31.0.150 |
| Pod 10: 172.31.13.208 | Pod 20: 172.31.7.250 |

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

17. In the properties of the Tag EC2 Instance, scroll down and supply the following details:

Override Workflow Target to **AWS_Target**.

Set the **AWS API Request** URL to the one below and the API Method to **GET**
``` https://ec2.amazonaws.com/?Action=CreateTags&ResourceId.1={Your Instance ID}&Tag.1.Key=Isolated&Tag.1.Value=&Version=2016-11-15 ```

<img width="1440" alt="5" src="https://user-images.githubusercontent.com/10421515/217512813-b0e3f9fb-b1c6-439a-96f3-c1c7fbbcdc6e.png">

18. Highlight the **{Your Instance ID}** in the URL string and click the puzzle piece in the upper-right of that field.

<img width="1441" alt="tag1" src="https://user-images.githubusercontent.com/10421515/217563952-29bb7057-b6ec-48e3-9733-c208b9777c26.png">

Choose **Activities --> Extract EC2 Details --> XPath Queries --> Instance ID**

<img width="618" alt="instanceid" src="https://user-images.githubusercontent.com/10421515/217561549-889d4f84-e1bc-4d09-b115-b91b1de7993e.png">

The resulting URL should show the variable name set as one of the parameters.

<img width="914" alt="awsapi1" src="https://user-images.githubusercontent.com/10421515/217562214-1225cb3f-cee0-426b-a1ec-152011028c85.png">

19. Since we only have one condition, you can delete the other conditional branch.  By clicking on it, choosing the three dots, and choosing **Delete**.

<img width="1440" alt="delete" src="https://user-images.githubusercontent.com/10421515/217564957-25552a79-ca77-4ae1-bb58-819bfca78a70.png">

20. Click into the **Yes** conditional branch to set the test condition.  Click on the puzzle piece in the **Left Opperand** field.

<img width="1440" alt="Condition" src="https://user-images.githubusercontent.com/10421515/217514255-71b49128-a605-4413-9049-dbef399387db.png">

Choose **Activities --> Extract EC2 Details --> XPath Queries --> Security Group**

<img width="626" alt="rightoperand" src="https://user-images.githubusercontent.com/10421515/217514576-105996a9-d205-4b2f-9f03-896b5ef2c864.png">

Then specify **Isolated_SG** as the **Right Opperand**.  When completed, it should look like the graphic below.

<img width="1440" alt="conditional 2" src="https://user-images.githubusercontent.com/10421515/217514942-195e73fb-eec6-4039-a0a7-8fb67a1d338c.png">

21. Click **Validate** again at the top of the window and run the workflow again.

<img width="1440" alt="validate 2" src="https://user-images.githubusercontent.com/10421515/217512954-05529e9c-46f7-4cc4-b286-e3f708d35a7c.png">

When the workflow runs this time you'll be able to see the output of the second activity that parses the XML response from the first activity and extracts the name of the Instance and assigend Security Group for your EC2 instance.  You'll notice that the conditional block doesn't trigger since the condition we specified doesn't match.

<img width="1440" alt="second run" src="https://user-images.githubusercontent.com/10421515/217569404-44666d5d-c6cc-479e-b24b-7b64f3e059d5.png">

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

4. Name your workflow **Pod X - AWS Incident Response** replacing the pod number with yours.
<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173711715-1849e6ac-150d-4a34-9e1e-dbb02ea800fe.png">

5. Replace the variable for **'observable_value'** with the IP address for your pod from the table below.

<img width="1440" alt="observable" src="https://user-images.githubusercontent.com/10421515/217571825-0aa28061-3290-4bfd-a3e5-79fb2094439d.png">

|     |     |
| --- | --- |
| Pod 1: 172.31.22.192 | Pod 11: 172.31.4.177 |
| Pod 2: 172.31.19.34 | Pod 12: 172.31.9.192 |
| Pod 3: 172.31.28.79 | Pod 13: 172.31.2.170 |
| Pod 4: 172.31.7.175 | Pod 14: 172.31.6.32 |
| Pod 5: 172.31.11.48 | Pod 15: 172.31.11.139 |
| Pod 6: 172.31.14.141 | Pod 16: 172.31.5.13 |
| Pod 7: 172.31.9.110 | Pod 17: 172.31.2.231 |
| Pod 8: 172.31.5.98 | Pod 18: 172.31.4.105 |
| Pod 9: 172.31.6.146 | Pod 19: 172.31.0.150 |
| Pod 10: 172.31.13.208 | Pod 20: 172.31.7.250 |

Replace the value for your pod and click save.

<img width="602" alt="observable2" src="https://user-images.githubusercontent.com/10421515/217571749-e5cdd295-e06f-47c6-bdce-2fbf20664553.png">

Note the activites in this workflow that automate many of the steps outlined in the AWS EC2 Incident Response Guide.

> * Enables Termination Protection on the instance
> * Sets a restricted Security Group limiting access
> * Removes it from any Auto Scaling Groups
> * Removes it from any Elastic Load Balancers
> * Snapshots connected Elastic Block Storage devices
> * Tags the instance with IR details

6. Run your imported workflow.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173712097-05b9094c-e06f-422f-aadf-c88a004e4e52.png">

7. Return to your previously created workflow **Pod X - AWS Workflow** that pulls instance details and run the workflow again.

9. Click on the **Extract EC2 Details** activty and note that after running the incident response workflow, the Security Group has been changed to move the impacted host to an isolated security group (Isolate_SG) and our condition now matches and the additional tagging activity has been run to identify the host as isolated.

<img width="1440" alt="third run" src="https://user-images.githubusercontent.com/10421515/217574158-8c1c3073-d447-45ac-bd5d-e1ff5611a231.png">

### Integration with SecureX threat response

1. Click **Dashboard** at the top of the SecureX Window to exit out of the Orchestration tool.

2. Expand the **Ribbon** by clicking the SecureX icon at the bottom-right of the screen to see created casebooks and incidents for this environment.

<img width="1440" alt="securex1" src="https://user-images.githubusercontent.com/10421515/217579250-7ad04e22-a54b-4a09-9fe9-82e1572553c4.png">

4. Click on the casebooks icon on the top of the window and find the **DEVWKS-2240 Casebook** Casebook created "By Others" by searching for 'DEVWKS' in the casebook pane.

<img width="1445" alt="casebook1" src="https://user-images.githubusercontent.com/10421515/217579421-c186b01a-a67f-4698-85a8-e15567bf3d5b.png">

4. Click **Investigate in Threat Response** located on the right-hand side of the Casebook drawer or on the upper-right of the ribbon.

<img width="1440" alt="casebook2" src="https://user-images.githubusercontent.com/10421515/217579607-b0c7779b-b05f-4725-a87d-79a8cc4e6d65.png">

This will open up a new browser window that contains the results of our investigation into one domain and the AWS internal IP addresses from our lab.

Once the enrichment of the observables is complete, you should see your IP address denoted as a 'target' in the resulting graph.

<img width="1398" alt="image" src="https://user-images.githubusercontent.com/10421515/167261531-68659cca-c8b5-4f04-b320-221f9afc36e1.png">

5. Use the drop-down to show the SecureX orchestration response actions that can be ran against this host, including the one you imported and uniquely named.

<img width="1440" alt="image" src="https://user-images.githubusercontent.com/10421515/173713168-a4a55394-ef21-4d51-bb2b-b8939234ca58.png">

**THANK YOU** for attending this DevNet Workshop and learning about SecureX orchestration.  I'll be putting some links and code into the Webex space associated with this session following this session!  Enjoy your time at Cisco Live!
