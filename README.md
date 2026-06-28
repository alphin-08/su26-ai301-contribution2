# Contribution [2]: [Bug] EConsult2Action: frontend()/login() return null instead of NONE after redirect; redundant isValidTask protocol check

**Contribution Number:** 2  
**Student:** Alphin Shajan  
**Issue:** https://github.com/carlos-emr/carlos/issues/3068 
**Status:** Phase I / II
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

- **Commit showing reproduction:** 
- **Screenshots/logs:** Not applicable since this is a code convention issue and not a visible crash.
- **My findings:**  I confirmed that both the frontend and login methods fall through to return null on both the success path and the error path, making them unreliable.
The protocol injection block in the isValidTask method is definitely unreachable because the filter above it rejects any string containing a colon, and a colon is required for every payload the block was trying to catch.

---

## Solution Approach

### Analysis



### Proposed Solution



### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 

**Match:**

**Plan:** 


**Implement:** 


**Review:**

**Evaluate:**

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: 
- [ ] Test case 2: 
- [ ] Test case 3: 

### Integration Tests

I didn't need to write any integration tests for this update. The fix was really just about adjusting how return values are formatted in the code. It doesn't touch the database or change how different parts of the app talk to each other.

### Manual Testing

Since this was just fixing code style and not an actual glitch you'd see on the screen, I didn't need to manually click around and test it in the browser. I know everything works correctly because all the unit tests passed and the whole project compiled perfectly with a clean BUILD SUCCESS.

---

## Implementation Notes

### Week [1] Progress



### Week [2] Progress



### Week [3] Progress


### Code Changes

- **Files modified:** 
- **Key commits:** 
- **Approach decisions:** 

---

## Pull Request

**PR Link:** 

**PR Description:** 

**Maintainer Feedback:** 

**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained


### Challenges Overcome


### What I'd Do Differently Next Time

 
---

## Resources Used

- Link to helpful documentation: https://github.com/carlos-emr/carlos/blob/develop/CONTRIBUTING.md, https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md, https://github.com/carlos-emr/carlos/blob/develop/.devcontainer/README.md
