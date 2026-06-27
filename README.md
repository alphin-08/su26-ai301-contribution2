# Contribution [2]: [Bug] EConsult2Action: frontend()/login() return null instead of NONE after redirect; redundant isValidTask protocol check

**Contribution Number:** 2  
**Student:** Alphin Shajan  
**Issue:** https://github.com/carlos-emr/carlos/issues/3068 
**Status:** Phase I

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

I set up the development environment using Docker Desktop and VS Code with the Dev Containers extension on Windows. The biggest challenge was enabling WSL integration in Docker Desktop, since without it the container could not find the Docker daemon. Once that was enabled under Settings, Resources, and WSL Integration, everything connected properly. I also had to run git config --global --add safe.directory /workspace inside the container because Git flagged an ownership mismatch that was blocking the build.

### Steps to Reproduce



### Reproduction Evidence

- **Commit showing reproduction:** 
- **Screenshots/logs:** Not applicable since this is a code convention issue and not a visible crash.
- **My findings:** 

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
