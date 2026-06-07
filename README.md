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

A "question" in the Community/Discussions feature is stored as two record types: a Discussion Topic (holding the title and owner) and one or more Discussion Reply records (the body and any follow-ups), linked to the topic via the reply's topic field. Creating a question in DiscussionModal.vue inserts a Discussion Topic followed by a first Discussion Reply.

The current UI only offers deletion of individual replies. deleteReply() in DiscussionReplies.vue:220-232 calls frappe.client.delete on the Discussion Reply doctype and nothing else. There is no action that deletes the Discussion Topic itself (Discussions.vue renders topic rows with no menu).

Root cause: Deleting a question's replies leaves the parent Discussion Topic record intact.

### Proposed Solution

Add the ability for a topic's owner to delete the entire question topic, removing both the Discussion Topic and all of its child Discussion Reply records together. I'll implement a whitelisted backend method that validates ownership and deletes the replies and topic in one operation (avoiding orphaned replies), and wire it to an owner-gated Delete action in the UI.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 

A user can delete the replies that make up their question, but the question's Discussion Topic is never deleted, so the question still shows in the list (empty title/body, author visible). The user should be able to fully delete a question topic they own, leaving no trace.

**Match:** 
- Reply deletion is the closest existing pattern: an owner-gated <Dropdown> + frappe.client.delete + resource reload (DiscussionReplies.vue:29-51, 220-232).
- Cascading topic+reply deletion already exists on the backend in lms/lms/api.py:1016-1026 (delete_batch_discussions uses frappe.db.delete("Discussion Topic", ...) and the replies) — a model for my whitelisted delete method.
- Ownership data is already available client-side: get_discussion_topics() returns owner and name per topic, and $user is injected in both components.

**Plan:** 
1. Backend: add a whitelisted delete_discussion_topic(topic) in lms/lms/utils.py that (a) loads the topic, (b) checks frappe.session.user == topic.owner (or has delete perms) and throws otherwise, (c) deletes all Discussion Reply where topic == name, then (d) deletes the Discussion Topic. Single source of truth, no orphaned replies.
2. Frontend: add an owner-gated Delete action for the topic — either a <Dropdown> on each topic row in Discussions.vue (with @click.stop so it doesn't trigger showReplies), or a menu in the thread header of DiscussionReplies.vue. Gate with user.data.name == topic.owner && !readOnlyMode.
3. Add a deleteTopic(topic) handler that calls the new backend method, then topics.reload() (and returns to the topic list if called from the thread view). Show a toast on error, matching existing handlers.
4. Optionally emit/listen on a socket event so other viewers' lists refresh, consistent with the existing new_discussion_topic / delete_message socket usage.

**Implement:** [Link to your branch/commits as you work] Not implemented yet.

**Review:** 
1. Follow Contribution.md: feature branch + Conventional Commit messages (repo uses fix(...), test(...), style(...) — e.g. fix(discussions): allow owners to fully delete their question topics).
2. Self-review checklist: ownership enforced on the backend (not just hidden in UI) so a non-owner can't delete via API; read-only mode respected; no orphaned replies remain; __() used for new strings; ruff formatting on Python, project formatting on Vue.

**Evaluate:** 

Log in as student1@example.com, create a question topic, confirm a Delete option now appears on it, delete it, and confirm it disappears from the list. Confirm a topic owned by another user shows no Delete option (security check).

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
