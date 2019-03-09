---
title: "Things to Consider While Creating XP Based Project Release Plan"
date: 2019-02-25T21:08:05+05:30
tags: [
    "Agile",
    "Extreme Programming",
    "Project planning",
]
---
<p align= "center">
    <img src="/images/ProjectPlanning_Banner.jpg" width="90%">
    <p align="center">
      <a href="https://www.google.com/search?q=inception+agile&tbm=isch&tbs=rimg:CU1WmTLpmIoAIjiuiodQH5cMERjnNZLRGuQBvI2PfcJ41S9mpjSN_1RV00mJDeFn4QDXlWXR2gXcnSiLULLfDdTGYQyoSCa6Kh1AflwwREd4PIbps6ILOKhIJGOc1ktEa5AERjUE2ixJvCkwqEgm8jY99wnjVLxHzoztorYTNBioSCWamNI39FXTSEVw5wYEeOojeKhIJYkN4WfhANeURbnrmiwGZP6cqEglZdHaBdydKIhGVYR6yIJ9WmSoSCdQst8N1MZhDEfnVOyQOfd4g&tbo=u&sa=X&ved=2ahUKEwj9ksiKyPXgAhVNinAKHdLEA6wQ9C96BAgBEBs&biw=1344&bih=719&dpr=2.5#imgrc=TVaZMumYigC2_M:">Image source</a>
    </p>
</p>

This blog lists down the common things which developers and QAs should consider while doing project planning exercise. However before jumping on to the list, let me cover some background first.

@<a href="https://www.thoughtworks.com/">ThoughtWorks</a> we follow XP practices. Any typical project starts with an Inception. It is a sort of workshop which typically starts with discovering what problem client is trying to solve. And it ends with a rough plan of how this problem can be solved. 

To come up with a plan or a project schedule we

<ol>
  <li>Prioritize the requirements and finalize the scope</li>
  <li>Break down the scope into features and user storiesç</li>
  <li>Estimate user stories</li>
  <li>Calculate raw velocity</li>
  <li>Plan iterations and milestones</li>
</ol>

Assuming the user stories are created, we use complexity based estimation technique to estimate stories.

## Complexity based estimates

Some people use Fibonacci numbers while some uses T-shirt size (Small, medium, large) nomenclature to estimate the stories.
Important thing to consider, this number denotes the complexity of the task but not the effort required to accomplish a task.
Team picks up the simplest story from the story deck and estimate it. Then this story is used as a base to relatively estimate other stories.

Next step is to do a Raw velocity exercise.

## Raw velocity exercise
<ul>
  <li>After estimating the entire scope now it’s essential to come up with a plan based on efforts i.e. plan in man hours/ months. This is what the client is actually looking for. To convert the estimates to a rough plan we do a Raw velocity exercise as follows.
     <ul>
        <li>BA/ PM from Team picks up a subset of the stories and hides estimates of those.</li>
        <li>Team is asked to tell how many of those can be done in 1 iteration by 1 dev pair (Iteration is typically of 1, 2 or 3 weeks). This exercise is repeated for 3-4 subsets of stories.</li>
        <li>BA/PM will average out the number which denotes the average velocity at which Team is going to churn the scope.</li>
     </ul>
  </li>
  <li>Based on the velocity and possible parallelization the number of iterations required to complete the scope are calculated. With this approximate end date is calculated.</li>
</ul>

Simple story so far, right? However I have seen people tend to forget things which they should ideally consider while doing the raw velocity exercise. This leads to incorrect calculations and wrong expectation settings with clients. 

## Checklist
Following is a checklist which I follow. Some of these will impact the velocity calculation while some items deserves a separate story. 
<ul>
 <li>Existing code base understanding</li>
 <li>Environment setup (Dev, QA, UAT, Prod)</li>
 <li>API
   <ul>	
     <li>Documentation e.g. Swagger</li>
     <li>Monitoring (Kibana, Grafana, Prometheus etc.)</li>
     <li>Liveness endpoints</li> 
     <li>Readiness endpoint</li>
   </ul>
 </li>
 <li>Loggers</li>
 <li>Security
   <ul>
     <li>Threat modelling</li>
     <li>CIA</li>
     <li>PII</li>
     <li>SSL</li>
     <li>Vaults</li>
   </ul>
 </li>
 <li>Tech. analysis of next iteration stories</li>
 <li>Addressing tech. debts</li>
 <li>Setting up correct ‘Test pyramid’
    <ul>
     <li>Integration tests</li>
     <li>Functional tests</li>
     <li>Consumer driven contract tests</li>
     <li>E2E tests (if functional tests mock other systems)</li>
     <li>Setting up mock server for testing</li>
    </ul>
 </li>
 <li>Quality metrics
    <ul>
     <li>Code coverage</li>
     <li>Sonarcube</li>
     <li>IDE setup e.g. stylecheck plugin, following single indentation scheme</li>
     <li>Code reviews (minimal)</li>
    </ul>
 </li>
 <li>Git
    <ul>
     <li>Pre-commit, pre-push hooks to run the unit/integration tests</li>
     <li>Scan repo for vulnerabilities</li>
     <li>Activating 2FA</li>
    </ul>
 </li>
 <li>CI Pipelines
    <ul>
     <li>Build, unit tests & integration tests</li>
     <li>Functional tests</li>
     <li>Smoke tests</li>
     <li>Consumer driven contract test pipeline</li>
     <li>Deployment pipelines</li>
    </ul>
 </li>
<li>Exploratory testing</li>
<li>End to end integration stories if development is done against mocks</li>
<li>Automating release cuts</li>
<li>Documentation (design diagrams, architecture diagrams, workflow diagrams)</li>
<li>Automating dev machine setup</li>
<li>Ramping up client devs if they are part of the development team</li>
<li>Appropriate versioning for CI artifacts</li>
<li>Tech. showcases within the Team or with client</li>
<li>Crash reporting tools integration like Crashalytics</li>
<li>User action tracker tools like Omniture/ Heap analytics</li>
</ul>