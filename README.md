## Electron: GSoC 2022


Project Title : Github Issues Automation
 
## Personal Information
Name : Sauvik Nath 
<br/>
Major : Computer Applications (Masters of Computer Applications)
<br/>
University : Assam Science and Technology University
<br/>
Institute : Assam Engineering College, Guwahati
<br/>
GitHub : https://github.com/Sauvikn98
<br/>
Email : sauviknath10@gmail.com
<br/>
Phone : (+91) 9089934714
<br/>
Location : Guwahati, Assam, India
<br/>
Postal Address : H/NO 54, Isha Enclave 3rd floor, Bhairavtola, Binovanagar - Kalapahar, near Kahilipara High School, Guwahati, Assam India - 781018
<br/>
Time Zone : IST( UTC + 5:30)
<br/>
Slack : Sauvik Nath
<br/>
Discord : Sauvikn98#4642
<br/>
Skype : live:.cid.63fdbe8d5ab82678
<br/>
Linkedin : https://www.linkedin.com/in/sauviknath/
<br/>
 
## Motivation for the Proposal
Anything about automation always had my interest as I always rely on this concept on “Anything taking you more than 10 seconds of your time, Automate it”. This concept motivated me to search for a project on GSoC, 2022 that in some or the other way would serve a use case to help reduce manual workload for an open source organization. I was familiar with Electron as I recently read a Github Gist on how Slack was created using the framework and how powerful it was, so my interest inclined to work with the organization on new features. As I was going through the idea list of Electron for GSoC, 2022 I found my best area of interest and started researching more about Github Action Bots. As I started investigating more about the project I started getting new ideas to implement and eventually even reached an alpha software repository of Electron named “Bugbot” that would later serve an important role in the project. I also mailed David Sanders and Keeley Hammond VerteDinde asking about the proposed idea on Bugbot integration and David Sanders replied to my mail saying it would be a great idea to integrate Bugbot in the standalone Issue Bot to make it even more powerful. That was a huge boost of motivation towards the proposal of this project.

## Synopsis
A Github App that will be architectured using Probot framework to automate the workflow of Issues and it will function as a standalone bot that can handle events based on the configuration discussed below and handle multiple webhooks to communicate with the Github API. The architecture will be layered upon two types of system configuration i.e. using RegEx match of the Issue Title (We will be using Minimatch to test for RegEx objects), Deep Testing of Issues using a modified version of Bugbot that will be integrated in the project. Bugbot will be handy to check for versions and Operating Systems affected by a Bug or a Crash to provide an extra level of system check when an Issue is Opened then report back in-issue to provide additional details about Versions affected by the Issue.
 
In addition to the above, the actions and data provided by the payload when a webhook is fired will be used to specify a set of instructions that will tell how the Github API will communicate using comments to automatically close an issue or remind a reporter about an issue that is under blocked/needs-repro for more than 60 days since the Issue was last updated. 
 
## Project Proposal 
As per the Idea is summarised, it is given that we want three problems to be solved and automated on Issue Tracking and maintenance. I would like to discuss in detail how the problems can be solved with least memory leak and high accuracy metrics:
1) Automatically Add Tags:
For adding Tags/Labels to group Issues based on similar types we will use Minimatch to check for RegEx in the Issue title that can be configured easily. We will create a .github/labeler.yml configuration file with a list of labels and minimatch globs to match to apply the label. Configuration can be stored as a plain list of label matchers, which consist of a label and a set of conditions for each. When all conditions for a label match, then the Action will set the given label. When any condition for a label does not match, then the Action will unset the given label for the Issue.
Here is the syntax of a matcher configuration for label "Bug":
<label>: "Bug"
<condition_name>: <condition_parameters>
<condition_name>: <condition_parameters>
 
For example:- 
.github/labeler.yml contains a single matcher with a single  condition:
version: 1
labels:
- label: "BUG"
title: "^BUG:.*"
An Issue with title "BUG: this is a bug" would be labelled as BUG. If the Pull Request title changes to "This is done", then the BUG label would be removed.
 
Also, To make the issue automation bot more powerful we will integrate a modified version of Bugbot that is an alpha software currently but I would like to integrate it for better testing and report to see what releases of Electron are affected by a Bug or a System Crash then report back in-issue to provide additional details about versions and operating systems affected by the code. That way we can maintain code easily and follow up with the bugs to be fixed in the specific versions and platforms.
We want to introduce more features to Bugbot so that it can identify and check for versions of Electrons that are part of a Crash, Memory Leak and other deadlock based problems so that the Issues can have additional description on what platforms and versions are affected.  
Basically, Bugbot currently uses an exitCode 0 if Electron works or non-zero value if Electron has a bug. So, Bugbot will use the exitCode available in the Issue repro [must for all issues] that needs to be generated by the user using a test case (Electron Fiddle is a good software to make small test cases).
This lets Bugbot see what releases are affected by the bug by checking your test against different Operating Systems and Electron versions. Some features of Bugbot will be modified and integrated in the standalone Issue tracker Bot to make it more powerful and robust. 
 
2) Add automated responses under certain conditions:
 
 - a reporter is missing a reproduction gist
 
To send automated responses using an Action Bot the first thing we require is to be informed when the Issue is created. So we will ask GitHub to alert us when this happens. GitHub alerts us via a webhook. The webhook is triggered by the issue being created and then makes a request to a Nodejs function, which we can define, passing along the payload of the issue that was created. Refer to the Figure for the flowchart of the entire process.

<img src = "https://user-images.githubusercontent.com/46704901/194745145-b562cad5-a755-4715-9d3f-c71181733fac.jpg" width="400">
 
We will create a Nodejs function that accepts the webhooks request and inspects its payload. We will parse out the creator and other details in the payload and format a response to the issue. Now that we have the data we need to create the comment on the issue, we need a way to talk back to the same issue and create the comment. Then we will call the GitHub API to create a comment on the issue, using a token that allows the function to make the call.
So, while parsing the payload data we will check if the reporter has provided a reproduction gist alongwith a test case report and if the data is not present then we will call the Github API to provide a comment in the issue telling “a reproduction gist is missing, Kindly review the issue”. Also, to be extra cautious we will label the issue as blocked/need-repro until and unless the reporter provides a reproduction gist with a pass/fail test necessary for input to BugBot testing

  - remind reporter if an issue is "blocked/needs-repro" for more than 60 days
 
We will first set a parameter for idle number of days or inactivity as 60 days before sending an automated response from the Bot reminding them of the blocked/needs-repro issue. Using the payload data we can check the Github Issue field (updated_at) and if the last update is older than the idle number of days i.e. more than 60 days then we will generate an automated response sending a comment in the Issue reminding the reporter that “the issue is under blocked/needs-repro for more than 60 days and needs immediate attention from the user”. Also any updates made, or any comments added to the issues will restart the counter of days before sending another automated message after 60 more days or eventually leading to closing the Issue if left unattended for {x} number of days.
 
3) Automatically close issues:
  - for versions of Electron older than the supported versions
  - if an issue has been marked “blocked” for more than {x} days
 
Here we will specify two special parameters:-
 
First, describing the supported version of Electron based on the current paradigms of usage and version control. If the supported version of the Electron matches with the one stated in the Issue then the Issue will be up and running but if the supported version does not match with the parameter then the Action bot will perform two tasks in a stepwise manner.  First, it will give a comment saying that “this Issue does not match the supported version of Electron so the Issue has been closed to avoid inconsistency”. Second, it will set a RegEx as “Closed” to automatically close the Issue and label the Issue as “Closed”.
Second, describing the {x} number of days an Issue can stay open. Once the issue crosses the {x} number of days parameter then the Action bot will perform two tasks in a stepwise manner. First, it will give a comment saying that “this Issue has no activity for more than {x} number of days so the Issue has been closed”. Second, it will set a RegEx as “Closed” to automatically close the Issue and label the Issue as “Closed”. 
The message that will be added as a comment to the issues when the Github Action workflow closes it automatically after being idle for too long or the version of Electron is older than the supported version of electron. 
Default value:- unset
Required Permission:-  issues: write
 Benefits to Electron

This project will be of great benefit to Electron as we know how troublesome and problematic it is to manually create and control Issues and sometimes while creating Issues we tend to miss crucial information about an Issue such as a reproduction gist necessary to know where and how the bug or crash is happening. We need to have a proper record and doing all of this manually could be the most difficult task at the end of the day. So this could be resolved with the help of an automation Bot that manages to triage the problem of Issue Tracking and maintenance of Issue. Currently Cation and Trop are fully functional to help automate Backporting process and monitor Pull Requests from contributors but it would be great for the workflow of Electron if we can add another Bot to help triage the automation of Issues and group them as per the label/tag provided.
The Bot will come handy as the use case is perpetual because a user needs to create and open issues to share a feature or a bug. So when an Issue is created it is necessary that a user follows a proper template with proper documentation of the issue and it is necessary to group the type of issue as well so that other contributors can filter the type of issue they want to work on and for doing that automation could save a lot of time and effort.
 
## Project Goals

The Primary objective of this project is to create a Github Action Bot that can handle Issue related automation for Electron’s workspace.
The Github Action Bot will allow developers and contributors to reduce manual workload over creating or updating an Issue, as the Bot will provide almost 80% of the required template for proper documentation of an Issue including grouping them under tags so that other developers can filter Issues based on tags and start working on the ones which they may prefer to.
The Bot will include additional details on which versions of Electron are likely affected by an Issue by reporting back in-issue directly. The details will directly point to the release versions affected so it would be easy to stacktrace the affected platforms. 
The Bot will automatically Close Issues under certain conditions like for example if an Issue is left unattended for more than {x} number of days or if the version of Electron is too old than the supported version according to the current paradigms.
 
## Deliverables

## Week 1 (13th June - 20th June) :
Read the documentation of Probot. (Investigation)
Initialize and configure a Probot app. (Required) 
Receive webhooks for opening or editing an Issue in Github. (Required)
 
 
 module.exports = (app) => {
    app.on(["issues.opened", "issues.edited"], async (context) => {
      // An issue was opened or edited, what should we do with it?
      app.log.info(context);
    });
  };
  
## Week 2 (21st June - 28th June) :
Work on converting glob expressions into Javascript based RegEx objects (Minimatch) to put a label/tag on an Issue. We will use forward slash “ / ” for glob expressions and check using RegEx objects. (Required)
Mandate glob expression for all Issue so that Issue Bot can be functional. (Required)
## Week 3 (29th June - 6th July) : 
Study about Bugbot and go through its source code to understand the usage and how it is being implemented. (Investigation)
Start modifying Bugbot so that it can test for other problems like Crash, Memory Leak or Deadlock and return the type of Crash or additional details in-issue with pointers pointing to the affected versions of Electron. (Required)
## Week 4 (7th July - 14th July) : 
 
Start Integrating the modified Bugbot in the standalone Issue Tracking Bot using Composite Actions defined in the YAML file with reference to the actions of Bugbot and test it locally. (Required) (May take more time as expected)
The composite actions will be something like this:
 
 
 name: "Issue Action Bot"
 description: "Triage Issues and report additional details  
 in-issue about versions and releases affected"
 
 inputs:
   created_at:
     description: “When the issue was created”
     required: true
   updated_at:
     description: “When the issue was last updated”
     required: true
   exitCode:
     description: “Pass/fail test for Bugbot”
     required: true
 
 runs:
   using: "composite"
   steps:
       - uses: electron/labeller-action@v1
 
       - uses: electron/bugbot-action@v1
         with:
           exitCode: ${{inputs.exitCode}}
       
       - uses: electron/build-push-action@v2
         with:
           context: .
           push: true
           tags: user/app:latest
 
## Week 5 (15th July - 22nd July) : 
 
Create a Nodejs function that will serve as a pipeline to listen to the webhooks and inspect the incoming payload so that important information can be fetched and used further to communicate using the Github API after generating a Personal Access Token (PAT) for the communication. (Required)
 
 
 
  const Trigger: NodeFunction = async function(context: Context, 
  req: Request): Promise<void> {
  const { body: payload } = req;
 
  // Gather the data from the payload from the webhook
  const repo = payload.repository.name;
  const owner = payload.repository.owner.login;
  const issue_number = payload.issue.number;
  const user = payload.issue.user.login;
  const action = payload.action;
 
  let body = 'Nothing to see here';
 
  context.res = { status: 200, body };
};
 
## Week 6 (23rd July - 30th July) : 
 
Research about the best and efficient approach to parse payload data and using npm and other javascript / typescript based utility modules available. (Optional) (Investigation)
 
## Week 7 (31st July - 7th August) : 
 
Start communicating with the Nodejs function by firing webhooks and create a comment telling the reporter that:
a reproduction gist is missing for the Issue (Required)
an issue is under "blocked/needs-repro" for more than 60 days and an immediate action is needed to be performed before the issue is closed automatically (Required)
 
 
  let body = 'Nothing to see here';
  if (action === 'opened') {
  body = `Thank you @${user} for creating this issue!\n\but a 
  reproduction gist is missing for the issue so the issue has been 
  labelled as blocked/needs-repro until a gist is provided`;
  context.log(body);
 }
 
                           OR
 
  let body = 'Nothing to see here';
  if (action === 'opened') {
  body = `Thank you @${user} for creating this issue!\n\but this 
  issue is under blocked/needs-repro for more than 60 days so an 
  immediate action is required before the issue is marked Closed`;
  context.log(body);
 }
 
## Week 8 (8th August - 15th August):  
 
Come up with two algorithms to communicate with the Nodejs function to close an Issue automatically :-
for versions of Electron older than the supported versions. We will set the supported version of Electron in the -set field and then compare the payload with the release version used for the Issue. If the electron version used for the issue is older than the -set version then the Issue will be immediately closed and labelled as close. (Required)
if an issue has been marked “blocked” for more than {x} days. First a comment will be given using the Github API informing about the same. We will check the created_at and updated_at payload data and check the -set{x} field. If it crosses the -set{x} days then the Issue will be automatically closed. (Required)
 
## Week 9 (16th August - 23rd August): 
 
Fine tune the actions and clean the architecture of the Probot App by Refactoring unnecessary modules / actions and focus on testing the workflow until Week 8. Also check for system bugs and redundancy pools. (Required) (Testing)
 
## Week 10 (24th August - 31st August): 
 
Document the entire workflow of the Github Action Bot. (Required)
Update the Readme.md with the required installation and configuration for other developers to contribute in the project
Provide valid Licensing under Electron’s workspace
 
## Week 11 (1st September - 8th September): 
 
Deploy the Github app in Electron’s workflow and modify the .yaml file to render the Github App in the main workspace and open a test Issue to check for all necessary features correctly working. (Required)
 
## Week 12 (9th September - 13th September): 
 
Write a Blog Post regarding the entire process of the project and share it in the GSoC community for other contributors to learn more about the project and add more features in the future updgrades. (Optional)
 
## Other Commitments:
I might have a National level All India based Hackathon contest probably in June or July for which I might have to travel to the allotted centre. Although the official schedule for the hackathon is not yet released. Prior to that I have no other commitments during the coding period and will be able to work for 40-45 hours per week. Having successfully completed other major open source projects, I am confident that I will be able to manage my time efficiently and devote as much time as required to complete the project.
 
## Why Me?
1) For this project we do require a good knowledge and enthusiasm to read up core concepts of Object Oriented Programming which I have, as I am majoring in Computer Applications
2) I have a cumulative percentile index of 8.0 ( university average is around 6.5) this shows that I am already profoundly familiar with the core foundational concepts.
3) I have a decent experience in programming and Javascript, above all I can independently solve and debug hard issues.
4) I have high problem solving capabilities as before I can proceed further, I try to formulate a pseudocode based on the logic of an algorithm that can be functional and efficient.
5) Also a plus is I take regular part in Competitive Programming Contests that help me prepare my logic for an algorithm under a time constraint situation.
6) I aspire to become a Full Stack Developer in the future. Due to academic constraints I didn't get the chance to contribute to any major open source project. 
7) Since any internship or research program requires some professional approach, this project would be a great opportunity for me to learn.
8) To me this project means a whole lot more value than any regular open source or GSoC project, this project has the capability to set up a foundation for my career. 9) That is why, I chose to apply for this project and put all my efforts into the proposal for this project.
10) Since, I have addressed most of the upcoming challenges - I am confident in myself, my skills and above all, my enthusiasm. If given a chance I will leave no stone unturned to excel in this project within the given period.
11) Link to my Resume: https://docs.google.com/document/d/1jXICobH00Ki1lSOaJrOL1L9Z3qaV0Q0Z/edit?usp=sharing&ouid=118175444602501880179&rtpof=true&sd=true
 
## My System Configuration
Operating System: Windows 11
<br/>
Electron Version: 18.0.3
<br/>
Npm version: 8.1.2
<br/>
Git Version: 2.34.1.windows.1
