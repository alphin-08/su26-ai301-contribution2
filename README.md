# Contribution [2]: [Bug] EConsult2Action: frontend()/login() return null instead of NONE after redirect; redundant isValidTask protocol check

**Contribution Number:** 2  
**Student:** Alphin Shajan  
**Issue:** https://github.com/carlos-emr/carlos/issues/3068 
**Status:** PR Merged
---

## Why I Chose This Issue
I picked this issue because it follows the exact same return NONE contract pattern I already worked on in issue 2885. That familiarity made me confident I could handle it without needing to learn a completely new part of the codebase. I also wanted to keep building momentum on the same project rather than starting fresh somewhere else.


---

## Understanding the Issue

### Problem Description

The Java class EConsult2Action handles eConsult redirects in the CARLOS system. Two methods called frontend and login send a web redirect and then return null. The project rules state that any action sending a redirect must return NONE right away. Returning null is risky because the Struts framework might still try to load a page after the redirect. There is also a problem in a helper method called isValidTask. It has a section of code meant to block dangerous web links but this section is dead code because a filter above it already blocks any text containing a colon. This dead code offers no extra protection but triggers a scanner warning.


### Expected Behavior

The frontend and login methods should return NONE right after a successful redirect. This tells Struts that the response is complete. If the redirect fails and causes an error, the methods should return the word error instead of doing nothing. The dead code in isValidTask should be removed because the filter above it already catches those bad links.


### Current Behavior

The frontend and login methods try to send a redirect and then return null no matter what happens. This means a failed redirect fails silently instead of showing an error page. The dead code in isValidTask is still there on lines 419 to 424. It triggers an IMPROPER UNICODE scanner warning even though the code never runs.

### Affected Components

The main file needing changes is the EConsult2Action Java file. The test file EConsult2ActionUnitTest must also be updated to check the return values for the frontend and login methods which includes testing what happens when the redirect fails.

---

## Reproduction Process

### Environment Setup

I did not have to do any further environment setup other than what I did for my last issue since it is the same project. I set up the development environment using Docker Desktop and VS Code with the Dev Containers extension on Windows. The biggest challenge was enabling WSL integration in Docker Desktop, since without it the container could not find the Docker daemon. Once that was enabled under Settings, Resources, and WSL Integration, everything connected properly. I also had to run git config --global --add safe.directory /workspace inside the container because Git flagged an ownership mismatch that was blocking the build.

### Steps to Reproduce
Because this is a code rule problem and not a software crash, you have to read the code to see it.

Confirming the return null after redirect:
1. Open src/main/java/io/github/carlos_emr/carlos/encounter/oscarConsultationRequest/pageUtil/EConsult2Action.java.
2. Find the frontend method so you can review the code structure and go to line 144 where the code calls the send redirect function to commit the response, and notice that the catch block on lines 145 to 147 only logs the error without returning anything.
3. Observe how execution falls through to return null on line 149 regardless of whether the redirect succeeded or failed, and confirm this single return null is reached on both outcomes.
4. The project rules require the success path to return the word NONE immediately and the failure path to return the word error instead of silently falling through.
5. Look at the login method and confirm the exact same pattern where the redirect is called on line 179, the catch block on lines 180 to 182 only logs the error, and execution falls through to return null on line 184.


Confirming the dead code in the isValidTask method.

1. Find the isValidTask method in that same Java file, and look at the text filter on line 410 which only allows letters, digits, underscores, hyphens, and forward slashes.
2. Notice that the filter does not allow a colon character under any circumstances, which is important for the next section of code.
3. Look at the protocol check block on lines 419 to 424 where it checks for dangerous payloads like http, https, javascript, data, and vbscript.
4. Recognize that every single one of those dangerous payloads contains a colon character, so the text filter on line 410 already rejects them before execution ever reaches line 419.
5. Confirm that the protocol block is completely dead code because it can never be reached, and the lowercase text conversion on line 420 is what triggers the static analysis warning for no added protection.
   
### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/alphin-08/carlos/commit/492b67f99bb2a6bad1f082f2dadb2750369e7440 
- **Screenshots/logs:** Not applicable since this is a code convention issue and not a visible crash.
- **My findings:**  I confirmed that both the frontend and login methods fall through to return null on both the success path and the error path, making them unreliable.
The protocol injection block in the isValidTask method is definitely unreachable because the filter above it rejects any string containing a colon, and a colon is required for every payload the block was trying to catch.

---

## Solution Approach

### Analysis

After doing some analysis, I realized both problems came from letting the code drop down to a default answer instead of telling it exactly when to stop. The frontend and login methods send a redirect but never return anything right after. The code just keeps going down to a single null return at the bottom, which means the success and failure paths end the exact same way. Once a redirect is sent, the response is committed to the browser, so the program should stop immediately with a NONE return. The second problem was the dead code inside the helper method that checks if the task parameter is safe. A pattern filter at the top already rejects any string with a colon, which means the lower block checking for dangerous protocols can never run. This lower block gives zero extra protection and causes a scanner warning, so removing it entirely was the safest move.


### Proposed Solution

My plan was to make both redirect methods return on purpose at every exit. On the success path, the method should return NONE right after the redirect call. On the failure path, the method should return the error result instead of doing nothing and dropping down to a null return. This ensures a failed redirect routes to an error page instead of failing silently. I also planned to delete the dead protocol block from the helper method since the pattern filter above it already does the real work of rejecting unsafe characters. Finally, I planned to add unit tests to confirm both methods return NONE after a successful redirect and return an error when the redirect fails.


### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 
The frontend and login methods must return NONE after a successful redirect and return an error when the redirect fails. The dead protocol check must be removed because the pattern filter already blocks those payloads.

**Match:**
This matches the same problem I worked on in a previous issue where actions writing directly to the response must end with NONE. They also need to handle their own error responses.

**Plan:** 
First I would fix the two redirect methods by adding the correct returns for success and failure, and then I would delete the leftover null returns. Next I would remove the dead protocol block and add unit tests for all four success and failure cases.

**Implement:** 
I edited the file so the methods return NONE after the redirect and return an error on the failure path. I removed the unreachable protocol block and added a nested test class that mocks the required components so the tests can run safely in parallel.

**Review:**
I checked that the success tests confirm a NONE return and that the failure tests confirm an error return. I also made sure removing the protocol block did not change what the helper method rejects.

**Evaluate:**
Both methods now end cleanly at every exit point, and a failed redirect routes to an error page. The static analysis warning is gone, and the new tests lock the behavior in place so the old pattern cannot return.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: login() returns NONE after a successful redirect. This test mocks the response so the redirect goes through, then confirms login returns NONE and that sendRedirect was actually called.
- [ ] Test case 2: login() returns error when the redirect throws an IOException. This test forces sendRedirect to fail and confirms login returns the error result instead of falling through.
- [ ] Test case 3: frontend() returns NONE after a successful redirect. This test sets up a valid session token and email, lets the redirect succeed, then confirms frontend returns NONE and that sendRedirect was called.
- [ ] Test case 4: frontend() returns error when the redirect throws an IOException. This test uses the same valid session but forces sendRedirect to fail, then confirms frontend returns the error result.

### Integration Tests

I didn't need to write any integration tests for this update. The fix was really just about adjusting how return values are formatted in the code. It doesn't touch the database or change how different parts of the app talk to each other.

### Manual Testing

Since this was just fixing code style and not an actual glitch you'd see on the screen, I didn't need to manually click around and test it in the browser. I know everything works correctly because all the unit tests passed and the whole project compiled perfectly with a clean BUILD SUCCESS.

---

## Implementation Notes

### Week [1] Progress

Week one went quickly because I did not have to set up the environment again. It was already ready from the first issue I solved on this same project. Most of the week was spent searching for the right issue to take on. Once I found this one, I waited to be assigned to it by the maintainers. After that I spent my time digging deep into what the issue was actually asking for. I read through the EConsult2Action class carefully, traced how the redirect methods ended, and confirmed the dead code claim myself before writing anything. I also worked on the documentation during this time so my understanding of the problem was written down clearly before I touched any code

### Week [2] Progress

Week two was fully focused on implementing the fix and testing it. I made the changes to the two redirect methods, removed the dead code from the validation helper, and then wrote the unit tests to cover both the success and failure paths. Once the tests passed and the build came back clean, I raised a pull request toward the end of the week. That pull request is still open right now. I am waiting patiently for a maintainer to review it and approve it.

### Week [3[ Progress

Week three was spent mostly on writing the documentation for this pr. I went back through everything I had done and wrote it all out clearly so the work would be easy to follow. Alongside that I kept an eye on the open pull request and waited for a maintainer to reach back to me with feedback, which has not come yet. I also started looking through the issue tracker again to find another good issue to take on next. I want to pick something that fits what I already know and is actually open for me to work on. So this week was a mix of finishing the documentation, waiting patiently for a review on the current pull request, and searching for what to work on after this one.


### Code Changes

- **Files modified:** src/main/java/io/github/carlos_emr/carlos/encounter/oscarConsultationRequest/pageUtil/EConsult2Action.java for the actual fix, and src/test/java/io/github/carlos_emr/carlos/encounter/oscarConsultationRequest/pageUtil/EConsult2ActionUnitTest.java for the new tests.
- **Key commits:** https://github.com/alphin-08/carlos/commit/492b67f99bb2a6bad1f082f2dadb2750369e7440 
- **Approach decisions:** I chose to return NONE on the success path and error inside the catch block so every exit is handled on purpose instead of falling through to null. I decided to delete the dead protocol block completely rather than stopping the scanner warning, since the pattern filter above it already blocks those payloads. For the tests, I mocked the CarlosProperties singleton with a thread local mockStatic instead of changing the global instance so the tests stay safe to run in parallel.

---

## Pull Request

**PR Link:** https://github.com/carlos-emr/carlos/pull/3109 

**PR Description:** Fixes two small things in EConsult2Action for issue #3068. The frontend and login methods used to send a redirect and then return null. Now they return NONE after a good redirect, and they return ERROR if the redirect fails instead of doing nothing. I also removed a dead if block in isValidTask. The regex at the top already blocks anything with a colon, so that block never ran, and removing it also gets rid of a scanner warning. The redirect links themselves did not change, only what the methods return.

Related issue is #3068, which this pull request fixes.

For testing, I added unit tests in EConsult2ActionUnitTest that checks if frontend and login returns NONE on a good redirect and ERROR when the redirect throws an IOException. Running mvn test on that class showed 22 tests run with zero failures and zero errors and a clean BUILD SUCCESS. All the older tests still pass too.

**Maintainer Feedback:** I have not received any maintainer feedback yet. The pull request has been open for more than two weeks now. I even tried tagging one of the maintainers in hopes that they would notice it, but no one has messaged me back so far. The pull request is still open and I am waiting patiently for someone to review it.

**Status:** Open and awaiting review.

---

## Learnings & Reflections

### Technical Skills Gained

I got more comfortable with the return NONE contract that Struts actions have to follow when they send a redirect. I also learned how to spot dead code by tracing the logic above it, since the regex filter made that code block impossible to reach. On the testing side I learned how to mock static methods so the tests could run at the same time without interfering with each other.

### Challenges Overcome

The trickiest part was proving the protocol block was truly dead code instead of just assuming it. I had to walk through the regex carefully and confirm that a colon could never get past it. Writing the tests was also a challenge because the action sets up its dependencies the moment it is created, so I had to mock the request, the response, the SecurityInfoManager, and the CarlosProperties singleton before building the action.

### What I'd Do Differently Next Time

Next time I would reach out to the maintainers a little earlier and more clearly when a pull request sits without a review for a while. I waited a long time before tagging anyone, and looking back I could have followed up sooner in a polite way to keep the review moving.
 
---

## Resources Used

- Link to helpful documentation: https://github.com/carlos-emr/carlos/blob/develop/CONTRIBUTING.md, https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md, https://github.com/carlos-emr/carlos/blob/develop/.devcontainer/README.md
