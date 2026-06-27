# su26-ai301-contribution2

Contribution [2]: [Bug] EConsult2Action: frontend()/login() return null instead of NONE after redirect; redundant isValidTask protocol check
 #3068
Contribution Number: 2
Student: Alphin Shajan
Issue: carlos-emr/carlos#3068 Status: Phase 1

Why I Chose This Issue
I picked this issue because I get to work on the real backend code of a medical records system. My job is to check a specific Java file and make sure certain methods return the right value instead of returning null. This fits my skills perfectly since I already know Java. I can practice reading the code to find out which parts send data directly to the user. I also need to write some tests to prove my code changes actually work. This builds right on top of the testing skills I learned during my senior capstone project. Fixing this issue will help me prepare for my new engineering job. It will show me exactly how big teams keep their code organized and secure.

Understanding the Issue
Problem Description


Expected Behavior


Current Behavior


Affected Components


Reproduction Process
Environment Setup
I set up the development environment using Docker Desktop and VS Code with the Dev Containers extension on Windows. The biggest challenge was enabling WSL integration in Docker Desktop, since without it the container could not find the Docker daemon. Once that was enabled under Settings, Resources, and WSL Integration, everything connected properly. I also had to run git config --global --add safe.directory /workspace inside the container because Git flagged an ownership mismatch that was blocking the build.

Steps to Reproduce


Reproduction Evidence
Commit showing reproduction:
Screenshots/logs: Not applicable since this is a code convention issue and not a visible crash.
My findings:

Solution Approach
Analysis


Proposed Solution


Implementation Plan
Using UMPIRE framework (adapted):

Understand: 
Match: 

Plan:


Implement:

Review: 

Evaluate: 

Testing Strategy
Unit Tests

Integration Tests


Manual Testing


Implementation Notes
Week [1] Progress


Week [2] Progress


Week [3] Progress


Code Changes
Files modified: 
Key commits: 
Approach decisions: 

Pull Request
PR Link:

PR Description:

Maintainer Feedback:

Status:


Learnings & Reflections
Technical Skills Gained


Challenges Overcome


What I'd Do Differently Next Time

Link to helpful documentation: https://github.com/carlos-emr/carlos/blob/develop/CONTRIBUTING.md, https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md, https://github.com/carlos-emr/carlos/blob/develop/.devcontainer/README.md
