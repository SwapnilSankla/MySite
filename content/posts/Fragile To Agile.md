---
title: "Fragile To Agile"
date: 2018-02-24T21:08:05+05:30
tags: [
    "Agile",
    "Extreme Programming",
    "Engineering Practices",
    "Team",
]
---
Folks, you must be wondering what kinda title is this! But I would like to share my experience of how we turned the Fragile development into working Agile development.

@<a href="https://www.thoughtworks.com/">ThoughtWorks</a> every project follows Agile software development practice. Our project is no exception. We were following most of the rituals. 

<ul>
  <li>Pair programming</li>
  <li>Continuous Integration</li>
  <li>TDD, BDD</li>
  <li>Agile ceremonies like standup, iteration planning meet-up, retros blah blah blah…</li>
</ul>

List seems sufficient to qualify for Agile, isn’t it? However there were many things which we were doing incorrectly. So instead of Agile, the process was actually qualifying for ‘Fragile development’

Ok so what were we doing wrong?

<ul>
  <li>Everyday starts with a standup however not everyone in Team is attending standup.</li>
  <li>CI practices are followed but no one bothers about the failures in the Functional pipelines. Only core build and unit test pipelines receives that respect.</li>
  <li>We begin iteration with a planning meeting, we commit to complete X points however almost every time we miss to achieve it. Still no one recons the smell. Instead of solving the problem, someone tossed up a new term ‘Healthy spillover’</li>
  <li>Let’s assume that while working on a story, dev pair identifies 10 things to refactor. But they able to knock off only 5 due to various reasons. Sad part is, details of the smells which were identified but not addressed are not captured anywhere.</li>
  <li>We typically follow a ratio of 1 QA person per 2/3 dev pairs. Developers focus on story implementation and move stories to QA bucket. As devs churn work fast, the QA bucket keeps on growing. QA need to write automation test, perform manual test around the story and do sanity test of app. This creates a panic during last days on the iteration. Developer assumes implementation done => story complete. S/he picks up a new story but don’t feel like helping QA so that the stories would be completed faster.</li>
  <li>Due to the panic which QA Team faces while testing the story, they don’t get time to write the Functional/Automation tests. Stories are moved to DONE without the tests. And these tests are written in next iteration iff QA Team gets time.</li>
</ul>

<p align="center">
<img src="/images/AmIAgile.png">
<a href="https://www.google.co.in/search?q=gorilla+thinking+image&tbm=isch&tbo=u&source=univ&sa=X&ved=0ahUKEwiw4eaghMDZAhUMLI8KHcP3D-UQsAQIJQ&biw=1307&bih=669#imgrc=29KG_6j3ZyST_M:">Image source</a>
</p>

Sounds like we are doing enough wrong things and hence we cannot say that we are Agile. Apparently we started new project. My BA(Vardhan) and I decided to address these concerns. Simple tricks helped us. Below is how.

<ul>
  <li>We started penalties for people who miss standup and for latecomers. Few bucks for every miss! Cool isn’t it! However as money is the easiest thing to pay, people started paying the dues without realising that missing standup is a problem.
  
  <p><i><font size="3" color="purple">‘Every person who miss standup or come late has to do X push ups in front of the Team. Crazy right! But it worked, now everyone is on time every time’</font></i></p></li>

  <li>We made a rule that developers cannot push a code if the functional pipeline is red. Initially people resisted the idea however later on everyone started owning up these functional pipelines just as they own the core build pipelines.
  
  <p><i><font size="3" color="purple">‘The success rate of functional pipeline is more than 75%. There are no flaky tests. Random/flaky behaviours are detected and fixed early’</font></i></p></li>

  <li>We made sure that we complete whatever we signup in the iteration planning meeting. Of course sometimes this can be tricky when there are factors which you cannot control e.g. unavailability of APIs. However apart from such external dependencies; we should not give any other excuse.
  
  <p><i><font size="3" color="purple">‘No spillover in last 7 iterations. First time on the account in last 5–6 years’</font></i></p></li>

  <li>We started tracking refactoring which a pair identified but could not complete. We started managing this in a separate tech. task board. Anyone can signup to complete these at anytime.
  
  <p><i><font size="3" color="purple">‘Separate board to track remaining refactoring/ tech. tasks. We trust our tests and hence we can carry out such refactoring at any time’</font></i></p>
  </li>

  <li>Developers started owning up the Automation tests. Just like unit and integration tests, devs write functional tests before moving story to QA bucket. QA can pair with the devs for these tests.

  <p><i><font size="3" color="purple">‘Devs to own Functional automation along with QAs’</font></i></p></li>

  <li>Devs/ QA pairs up with BA for requirement gathering, analysing API responses. During the last 2–3 panic days of iteration, devs and BA contributes in testing stories.
  
  <p><i><font size="3" color="purple">‘Think beyond building cross functional Teams. Make sure that people are open to help other roles as well. Every Individual can do/help in analysis, development, testing etc’</font></i></p></li>
</ul>

With these small things everyone started feeling motivated. Everyone in the Team cherish moments when story is marked as done. In an iteration of 2 weeks, we now complete signed up items in 8–9 days. We save at least a day every iteration! Awesome isn’t it? But what to do with this extra day? We started a new experiment in which every Team member gets a day to explore whatever thing s/he wants to do. It can be literally anything like learning a new language, knocking off tech. tasks or even enhancing Table Tennis skills! It’s now 3 iterations since we started this and everyone waits for that day to come.

This blog is originally published <a href="https://medium.com/dev-data/fragile-agile-4611ecfceb32">here</a>.