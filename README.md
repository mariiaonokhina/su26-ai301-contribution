# Contribution #1: Ability to delete one's question topics

**Contribution Number:** 1

**Student:** Mariia Onokhina

**Issue:** https://github.com/frappe/lms/issues/908

**Status:** Phase II Complete

---

## Why I Chose This Issue

This issue interests me because it's an actual bug that affects user experience on the learning platform called Frappe Learning. The users are not able to delete their own questions after posting, which takes away control over their own content.

I've worked on full-stack applications before with a similar tech stack to this project (Python, JavaScript, TypeScript, HTML), and I have an extensive experience creating scripts and games in Python, as well as interactive web apps with JavaScript. I'm hoping to learn more TypeScript, have the experience of working with Vue, and I would like to contribute to a larger open-source codebase.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

1. Fork the original repository (https://github.com/frappe/lms).
2. In VS Code or another IDE of your choice, run `git clone <link-to-forked-repo>`.
3. You need Docker, docker-compose and git setup on your machine. Refer to [Docker documentation](https://docs.docker.com/) for installation.
4. Run `mkdir frappe-learning`, then `cd frappe-learning`.
5. Download the docker-compose file by running `wget -O docker-compose.yml https://raw.githubusercontent.com/frappe/lms/develop/docker/docker-compose.yml`.
6. Download the setup script by running `wget -O init.sh https://raw.githubusercontent.com/frappe/lms/develop/docker/init.sh`.
7. Run the container and daemonize it by running `docker compose up -d`.
8. Wait for a couple of minutes. The website should then be running locally at [http://localhost:8000/lms](http://localhost:8000/lms).

Use the default credentials to log in:
- Username: Administrator
- Password: admin

### Steps to Reproduce

1. Log in from the Administrator account (Administrator:admin).
2. Go to Frappe Learning.
3. Make sure that there is at least one course visible in "Courses" and that it is published.
4. Add a test student to the course by running the following in the terminal:
   ```python
   docker exec lms-frappe-1 bash -lc "cd ~/frappe-bench && bench --site lms.localhost console <<'PYEOF'
   import frappe
   email = 'jane@example.com'
   pwd = 'Maple-River-3381!'
   if not frappe.db.exists('User', email):
     u = frappe.get_doc({
        'doctype': 'User',
        'email': email,
        'first_name': 'Jane',
        'last_name': 'Doe',
        'send_welcome_email': 0,
        'enabled': 1,
        'new_password': pwd,
    }).insert(ignore_permissions=True)
   else:
    u = frappe.get_doc('User', email)
    u.new_password = pwd
    u.save(ignore_permissions=True)
   if 'LMS Student' not in [r.role for r in u.roles]:
    u.add_roles('LMS Student')
   frappe.db.commit()
   print('User:', email, '| Roles:', [r.role for r in u.roles])
   PYEOF"
   ```
   This will create a user with the name Jane Doe, email jane@example.com and password Maple-River-3381!
6. From the administrator's view, go to Courses -> Select any course available -> Dashboard -> + Enroll. Then, search for student with the email jane@example.com and add them.
7. Log out from the administrator account and log in as a student (jane@example.com:Maple-River-3381!).
8. Go to Courses -> Enroll into course -> Click on any available module in the course -> Community -> New Question. Set up a question with any topic and details.
9. Click on the question you just created. Then, press on the three dots next to it and select "Delete".
10. **IMPORTANT:** I think someone already fixed the issue but not completely. The question gets deleted but you should still be able to see the fact that the question was created and by whom (with empty topic and details), which it shouldn't be.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/mariiaonokhina/lms (follow the steps above to reproduce the issue).
- **Screenshots/logs:**

<img width="648" height="160" alt="IMG_8123" src="https://github.com/user-attachments/assets/d5625e93-d096-4fb9-b3b1-b4ad23953aaa" />

<img width="817" height="472" alt="IMG_2628" src="https://github.com/user-attachments/assets/c56b32a0-acfe-4783-b6a0-322eb92e056e" />

<img width="690" height="457" alt="IMG_8371" src="https://github.com/user-attachments/assets/6f9c8d8c-ff28-4c2c-b8c2-bc181f575bff" />

<img width="699" height="346" alt="IMG_6668" src="https://github.com/user-attachments/assets/5a9963ff-44c7-408f-a73d-d128f8012ca3" />

- **My findings:**
You are able to delete the questions that you make, but a trace of them still shows up with the question topic and the username. So, the questions don't get fully deleted.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
