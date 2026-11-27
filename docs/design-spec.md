# TaskHive — Product & Design Specification

**Date**: 2026-11-23 (exported 2026-11-27)
**Status**: draft (living spec)
**Owner**: Łukasz Kot (with Claude Code)
**Jira**: PA-86 (epic), label `taskhive`

> This file is a Markdown snapshot of the TaskHive spec Claude Doc, which stays the source of truth. Requirement changes go to the doc first and are then re-exported here. Diagrams are not exported; they live in the doc.

This spec is adapted one decision at a time, and every change is recorded in the Decision log near the end.

**Reading rule for implementers:** text marked **To decide for TaskHive** is not yet a requirement.

## Overview

TaskHive is a task and agents manager. It is also a plain to-do app for tasks the owner completes themselves, and a habit tracker for good and bad habits with statistics. It is not a manager for small teams. Besides doing tasks personally, the owner can assign them to AI agents or to a few human collaborators. It is built on one idea: **work is discussed inside tasks, not over email or chat**. Its core hierarchy is Space → Project → Section → Task → Comment.

**Positioning and philosophy**

- **Task-based communication.** Each task has its own comment thread, so a conversation stays next to the work it is about.
- **GTD heritage.** Incoming works as the GTD inbox, tags work as contexts, and Priority works as "next actions".
- **Personal views on top of team projects.** Priority, Incoming and Single tasks are personal. Projects are shared.
- **Target users:** one owner, plus a few human collaborators such as reviewers. Not small teams.
- **Platforms:** a React web app and a native Swift iPhone app (no iPad layout), both offline-first and synced in real time through the TaskHive backend. There are no macOS, Windows, Android or Apple Watch apps.

**Pricing:** TaskHive is free. There are no plans, billing or plan-based limits. Wherever this spec mentions a Free, Premium or Business restriction, the feature is available to everyone.

## Domain model

Each space is a tenant, and its data is isolated from other spaces. Users and spaces are many-to-many: a user can belong to many spaces. Only owners and admins invite users, and each invitation is for one space. All content, agents included, lives inside one space. Tasks belong to exactly one project or to the user's Single tasks. Tags cut across projects.

| Entity | Belongs to | Key attributes | Rules |
| --- | --- | --- | --- |
| Account (user) | — | Name, email, avatar, password or Google / Apple / Microsoft sign-in, 2FA, notification settings, vacation dates | Owns or joins many spaces. Always has at least one space; a user removed from their last space gets a new empty space. |
| Agent (new in TaskHive) | Space | Name, avatar, description of what it does, connection (MCP client or API token), granted projects, enabled or disabled | An AI worker. A task's assignee can be a person or an agent, and both follow the same task model. |
| Space | Account (owner) | Name, avatar, members, agents, personal or team mode | Owner, admins, members. Invitations expire after 14 days. Removing a space deletes all its data. |
| Project | Space | Name, purpose (description), colour, access (open, limited, private), managers, members, time attributes on/off, state (active or ended) | The creator becomes manager. Can be shared with one other space. Must be ended before it can be deleted. |
| Section | Project | Name, order, archived | Can be cloned, converted to a project, archived and restored. |
| Task | Project or Single tasks | Name, assignee (a person or an AI agent), section, tags, priority star, due date and time, reminders, recurrence, time needed, time spent, followers, status (active, in progress, needs review, completed, abandoned), reviewed or unread per user | Completed tasks are archived after 3 am. Deletion is permanent and restricted. |
| Habit (new in TaskHive) | Project (optionally a section) or standalone | Name, type (good or bad), icon, colour, goal (value, unit, period), repeat, start date, reminders, end condition, owner, archived | Placed like a task: in a project or standalone. Only its creator owns it and checks in. Streaks and statistics are calculated from its statuses, never stored. |
| Habit log (new in TaskHive) | Habit, date | Value, unit, created at | Several logs a day are added together. Undoing a day removes that day's logs. |
| Habit status (new in TaskHive) | Habit, period (day, week, month or year) | In progress, completed, skipped, failed (succeeded for bad habits) | Decided by the period's log total, or set by hand (skip, fail). Off Mode sets skipped. |
| Comment | Task | Type (text, checklist, attachment), author, pinned, reactions | Pinned comments are kept when a task is cloned, converted or renewed. Habits have comment threads too. |
| Checklist item | Comment | Text, checked | Can hold task links and @mentions. |
| Tag | Space | Name, icon, colour, private or shared | Groups tasks across projects. Can be pinned as a sidebar tile. |
| Template | Space | Source project structure | A new project can start from a template with a reference date. |
| Reminder | Task | Absolute time, or relative to the due date; urgent (alarm) on iOS | A task can have several. |
| Activity entry | Task or project | Change type, actor, time | Quick successive changes are grouped as "X changes". |

> _Diagram (domain model · containment hierarchy) — see the Claude Doc._

A task sits in a project (optionally in a section) or in its owner's Single tasks. Tags cut across projects.

## Core features

TaskHive offers three personal views (Priority, Incoming and Single tasks) on top of shared projects. Each task carries a full comment thread.

### Personal views

| View | Contents | Rules |
| --- | --- | --- |
| **Priority** | Starred tasks from any project. Optional automatic rules: assigned to you, due today, overdue, you were mentioned, today's reminders. | A task created here is stored in Single tasks. Priority is a lens, never a location. |
| **Incoming** | Everything that needs attention, in groups: Due today, Overdue, Reminders, Assigned to you, You were mentioned in, Needs review (tasks your agents finished; new in TaskHive), Recently added (email, Quick Add, integrations, widgets). | Mark a task reviewed with the orange button or a right swipe; mark a whole group with "Mark as Reviewed". Commenting, renaming or changing a field also marks it reviewed, with Undo. A reviewed overdue task comes back the next day. |
| **Single tasks** | Tasks with no project: Quick Add, email, tasks created in Priority, Calendar or Tags. | Private to the owner by default. "Task Me" on a member's profile drops a task into their Single tasks. |
| **Calendar** | Tasks with due dates and reminders; future recurrences greyed out with ⟳. | Week or next-30-days mode. Days collapse. Dragging a task to another day changes its due date. Filter by member. |
| **Tags** | All tasks with a tag, across projects. | Grouped by project by default. Turning grouping off allows manual drag sorting. Export to CSV, stats. |
| **Habits (new in TaskHive)** | Your habits due today, from every project and standalone. Day view grouped as In progress, Completed, Skipped and Failed; Week view with one row per habit. | A separate view: habits don't appear in Priority, Incoming or Single tasks. Filter by project. Check in from the row. |

Incoming timing rules:

- A task due on a date appears at midnight; a notification fires at 7:00.
- A task due at a time appears at midnight; a notification fires 30 minutes before.
- Recurring tasks renew between 0:00 and 4:00.
- A reminder with a time appears at that time.

### Tasks

- **Fields:** name, responsible person, project and section, tags, priority star, due date with optional time, several reminders, recurrence, time needed and time spent (switched on per project), followers.
- **Quick add with #:** typing `#` after the name opens a picker for project, section, responsible person, date, time, recurrence, reminder, tags and priority. A partial word filters it. A task can be added to the top or the bottom of a list.
- **Overdue state:** the date turns red with a flame icon.
- **Recurrence:** needs a due date. Presets are daily, every 2 days, weekdays, weekly, every 2 weeks, monthly and yearly. Custom is every X days, weeks (with weekdays), months (with a day) or years. It ends never, on a date, or after X times. An edit applies to "Only this task" or "This and all following tasks". A red counter shows skipped occurrences. Reminders can be relative to the due date.
- **Lifecycle:** active → completed or abandoned. Both show under "Completed today" or "Abandoned today" until 3 am, then go to history. Abandoned tasks can be restored. Deleting is permanent. Only the owner, admins and project managers may delete, or the author within 5 minutes of creating the task.
- **Read state:** each user has a reviewed or unread flag per task. A blue dot marks new activity.
- **Following:** a user follows a task automatically when assigned, mentioned, or when adding it to a project they don't follow.
- **Multi-edit:** select several tasks (⌘/Ctrl-click, Shift-click). Then complete, abandon, star, mark unread, move, assign, tag, date, remind, set time, comment, rename, follow, or move to another space.
- **Clone:** copies fields and all comments or only pinned ones. The clone's first comment links to the original.
- **Convert to project:** checklist items become tasks. Other content goes into a "Project resources" section. The original task is marked abandoned.
- **Task links:** a pasted task link becomes a hyperlink. It is struck through when the target is completed, which allows task chains.
- **Share a task by email:** the guest gets a reusable magic link and needs no account. They can comment, assign back, and set a date or reminder.

### Agents (new in TaskHive)

An agent is an AI worker that belongs to a space and takes tasks the way a person does. TaskHive does not run agents: they run elsewhere and connect to TaskHive.

- **Agent record:** name, avatar, a description of what it does, its connection, the projects it may access, and an enabled or disabled switch. An agent belongs to exactly one space and never works across spaces.
- **Connection:** MCP is the main channel. Each agent has its own credentials (an MCP OAuth client or an API token), limited to its space and its granted projects. The REST API and outgoing webhooks are also available; a webhook tells the agent when a task is assigned to it. Agent credentials are separate from user API tokens: every agent action is recorded under the agent's name, and revoking one agent affects nothing else.
- **Work lifecycle:** Active → In progress (the agent picks the task up) → Needs review (the agent reports it done) → Completed (the reviewer approves). The reviewer can send the task back to In progress with a comment. An agent cannot complete its own task.
- **Reviewer:** the person who assigned the task to the agent. Tasks waiting for them appear in the Needs review group of their Incoming.
- **Reporting:** the agent posts progress and results as task comments (text, checklists, attachments). Every agent action appears in the task's activity log under the agent's name.
- **People:** Tasks assigned to you show no In progress or Needs review controls; you just complete them. Human collaborators may use those statuses, but don't have to.

### Habits (new in TaskHive)

Habits sit next to tasks. A habit is created the same way as a task: inside a project (optionally in a section) or standalone, with no project. Only its creator owns it and checks in. In a shared project other members can see the habit and its statistics, but can't check in on it.

- **Good habit:** a goal to reach, with a value, a unit and a period (per day, week, month or year), for example 2 L of water a day or 3 workouts a week. Units are counts, time, distance, volume, weight or energy, or a custom count unit.
- **Bad habit:** either a limit not to exceed in a period, or quit (zero allowed). A day at 0 or within the limit becomes Succeeded automatically at the end of the day.
- **Repeat:** every day, chosen weekdays, chosen dates of the month, or every N days. A start date can be in the past (to backfill) or the future.
- **Check-in:** a smart button (+1 for counts, a keypad for other units), Complete (logs the full goal), Log a value, Skip or Fail. Going past the goal is allowed, and the last action can be undone.
- **Statuses:** In progress, Completed, Skipped, Failed (Succeeded for bad habits). A skip keeps a daily streak going but doesn't protect weekly, monthly or yearly streaks.
- **Reminders:** several time reminders per habit (optional weekdays), location reminders (arriving at or leaving a place), and habit stacking (remind after another habit is completed, skipped, failed or reminded, immediately or after a delay).
- **End conditions:** finish on a date, after a streak length, after a number of successful periods, or after a total amount. Reaching one shows a milestone celebration.
- **Off Mode:** a planned break of at least 2 days. It is the same break as vacation mode: one set of dates marks every habit Skipped, pauses habit reminders and task notifications, and keeps streaks.
- **Archive** hides a habit and keeps its history; **delete** removes everything.
- **Not included:** timer, mood and reflection, Apple Health and other auto-logging, Time of Day blocks, widgets, Habit Gate, Circles and Challenges, Habi AI.
- **Comments:** habits have the same comment thread as tasks (text, checklists, attachments, mentions), used for notes and photos about a habit.
- **AI assistant:** your own AI assistant, acting as you through MCP, can list your habits and log check-ins (for example "log 2 L of water"). Agents can't own habits.
- **Visibility:** a habit in a shared project is visible to everyone in that project, statistics included. Standalone habits are private.

### Comments

- **Types:** text, checklist, attachments. One comment can combine them.
- **Markdown:** bold, italics, strikethrough, inline and block code, `#` and `##` headers, rules, `==highlight==`, links, `tel:` links.
- **Attachments:** paperclip or drag and drop. Thumbnails and in-app previews for images, GIFs, PDFs, video, YouTube and Vimeo. A grid view shows all attachments in one comment.
- **Size limits:** text over 12 KB is kept as a first line plus a TXT attachment. The checklist limit is 80 KB.
- **Actions:** pin (orange, filterable), edit, "make editable by others" (pins it and cannot be undone), reply, copy, delete (logged in history), create a task from the comment.
- **Reactions:** five fixed reactions: thumbs up, thumbs down, love, haha, sad.
- **Mentions:** `@name`, `@following`, `@all`. A mention sends a push notification and puts the task in the person's Incoming. Your mentions show blue; mentions of you show orange.
- **Email-in:** every task has its own address; an email sent there becomes a comment.

### Projects

- **Fields:** name, purpose, colour, access, time attributes.
- **Sections:** add, reorder, collapse, clone, convert to a project, archive and restore. Each section has its own add-by-email address.
- **Following and favourites:** following sends the project's new comments to Activity. Favourites sit above the projects list and are followed automatically.
- **Ending:** an ended project shows crossed out and can be restored. Deleting requires ending it first and typing its name.
- **Clone and move:** a project can be cloned or moved to another space. Moving removes access for people outside the destination space.
- **Stats:** active and completed counts, time progress, and "Pace" by day, week and month.
- **Export:** CSV export and printing to PDF.

### Templates

- Created from scratch or from a project, with or without completed tasks, and with all or only pinned comments.
- Due dates are relative: an offset from the project's creation date, or from a chosen **reference date**. Recurrences are kept.
- Visible to the whole space or to selected users.

### Search and filters

- **Jump to:** opened with ⌘K or Ctrl+K. Finds tasks, projects, sections, tags and members. Fuzzy by default; can be switched off.
- **Prefixes:** `p:` projects, `t:` tasks, `m:` members, `s:` sections, `tag:` tags. Separate modes search comments and completed tasks.
- **Filters:** name, project, section, responsible person, tags, due date (overdue, from, until, none), time needed, priority, granted access. Each sort option has ascending and descending order and shows as a removable chip.

### AI project generation

Dropped in TaskHive. An agent connected through MCP can draft a project instead of a built-in AI feature.

## Habit statistics (new in TaskHive)

Every habit has its own dashboard, and a Progress screen summarises all of them. Good and bad habits are reported separately, and habits are grouped by project.

**Habit dashboard**

- Current streak and streak history; weekly, monthly and yearly goals show completion per period instead.
- Success, Failed, Skipped and Total (in the goal's unit), each compared with the previous range.
- Calendar: full circle for completed, partial circle for partial progress, an arrow for skipped, a cross for failed.
- Column chart by day, week, month or year, with the goal line and the average per period.
- Consistency Index for daily good habits over 30, 60, 90 or 180 days.
- Progress bar towards the end condition, if set.

**Progress screen** (date range 7 days to 6 months or custom; filter by habit or project)

| Report | Blocks |
| --- | --- |
| Good habits | Consistency heatmap, average daily score with trend, Nothing / Partial / Complete days, Weekly Rhythm (completion per weekday), Missed habits (skipped versus failed, most skipped and most failed), one card per project, one card per periodic goal, one card per daily habit. |
| Bad habits | Clean versus slip days calendar, Days Clean with clean rate, Slip Days, current and max clean streak, Slip Rhythm per weekday, limit charts (overage, average, peak), quit chains (slips, max streak, success rate). |

All statistics are calculated from statuses and logs, never stored, so they work offline on both clients.

## Collaboration

Collaboration runs at three levels: space, project and task. Delegation is a task's responsible person; discussion is its comment thread. Notifications land in Incoming, and followed work lands in Activity.

**TaskHive change:** the app serves one owner and the people they work with. One application instance serves many spaces; each space is a tenant, and users and spaces are many-to-many. AI agents are assignees too (see Agents). There is no billing, so plan rules below do not apply. Roles are space Owner, Admin and Member, plus the project roles below. Only owners and admins invite, one space per invitation. Member groups and project groups are dropped.

### Roles and permissions

| Level | Role | Can do |
| --- | --- | --- |
| Space | Owner | Everything an admin can, plus deleting the space. If the owner leaves, an admin (else any member) becomes owner. |
| Space | Admin | Invite, accept and remove members; promote or demote admins; manage space settings. Cannot remove the owner or transfer ownership. |
| Space | Member | Create and join projects, assign tasks, comment, leave the space. |
| Project | Manager | Edit project settings, colour and access; invite or remove people; grant manager rights; end and delete the project. The last manager must hand over before leaving. |
| Project | Member | Add and edit tasks, assign, comment, attach files. |
| Project | Guest | View and comment only, in the invited project only. Cannot edit or assign tasks. Suited to clients and contractors. |

### Project access

- **Open** (full circle icon): every space member. This is the default.
- **Limited** (donut icon): selected people only.
- **Private** (donut with padlock): the creator only.
- **Shared with another space:** the project becomes limited access. Each project can be shared with one other space. Invitees can be people without a TaskHive account.

### Delegation and membership

- **Assignee:** one per task: the owner, a collaborator or an agent. A new task with no assignee belongs to the person who created it. Assigning puts the task in the assignee's Incoming. It also makes them a follower.
- **Task Me:** creates a task straight in a colleague's Single tasks.
- **Invitations:** by email or by a link code that the owner confirms. Only owners and admins can invite, and each invitation is for one space. Invitations expire after 14 days.
- **Removing a member:** their active tasks are reassigned to the space owner. Their comments stay, marked with a grey dot. Their projects are kept.
- **Vacation mode:** a start and end date per space. It pauses that person's notifications and reminders, and is the same break as habit Off Mode, so their habits are marked Skipped and their streaks kept. Colleagues see the dates when they assign or mention the person.

### Notifications and Activity

- **Triggers:** reminder fires, task assigned to you, @mention, task becomes due, agent finished a task (Needs review), habit reminder.
- **Channels:** push (device and browser), email, and in-app (Incoming). Each is toggled in Settings → Notifications.
- **Email reports:** dropped in TaskHive. Email goes out only for assignments, mentions and tasks waiting for review.
- **Activity view:** new comments and events in followed tasks and projects. Ordered by: tasks you are responsible for, then tasks you took part in, then favourite projects, then followed projects. Opening a task or pressing "Read" clears its blue dot; a header button marks a whole list read.

## UI and UX design

The layout is a customisable sidebar of tiles plus one main panel per view. Every list behaves the same way: a task row, a details view with its comment thread, multi-select, filters, and an add (+) button.

### Sidebar

1. **Space switcher:** avatars of the user's spaces, plus "+ Add a Space" (create or join).
2. **Search** (Jump to) and the **settings** gear.
3. **Pinned tiles**, in the default order Priority, Incoming, Habits, Calendar, Single tasks. The user can reorder them by drag and drop.
4. **Favorites:** projects, people, tags or templates.
5. **Active projects**, then **More**: Tags, Followed projects, Members, Activity, Templates.
6. **Help:** contact, help centre, shortcuts, app tour.
7. **Master Plus (+):** add a task (Quick Add), project, tag or member. Always visible.

Customisation:

- Pin or unpin any view, member, tag or template as a tile.
- Change each tile's colour. "Minimalist tiles" colours only the icon.
- Collapse groups.
- Each space keeps its own sidebar layout.
- A personal (single-user) space shows a simplified two-tile interface. It switches to the full layout automatically when the first member is invited.

### Main panel and task row

- **Header:** the view name, a "…" menu for view details, settings, pin, colour, copy link, export to CSV and print, then filters, sort and in-view name search.
- **Task row:** completion circle, name, then compact badges: project or section, responsible person's avatar, due date (red with a flame when overdue), reminder bell ("+X" for more), recurrence, tags, comment count, priority star, and an unread blue dot.
- **Incoming rows** add an orange "reviewed" button.
- **Task details:** the fields as chips at the top, then pinned and regular comments, the inline activity log, and the comment editor at the bottom. Fullscreen and focus modes are available.
- **Completed and abandoned today** stay visible in a collapsed group at the end of the list until 3 am.

### Interaction patterns

- **Keyboard first on web:** `1`–`4` Priority, Incoming, Activity, Single tasks; `5`–`8` favourites 1–4; `F` search; `D` next 30 days; `N` new task; ⌘/Ctrl+K jump to.
- **Task shortcuts:** ⌥/Alt+D complete, X abandon, S star, R rename, 1–6 set project, section, responsible, tags, due date, reminders; M comment; J/K previous and next task.
- **Global Quick Add:** ⌘⌥N on web, Ctrl+Alt+N on Windows and Linux browsers.
- **Mobile gestures:** swipe right to mark reviewed or read; swipe left to open the task's project; long-press the app icon for Quick Add, Search and Switch space.
- **Undo bar:** shown after automatic state changes such as auto-review.
- **Drag and drop:** tasks between sections and calendar days; files into comments; sidebar tiles.

### Platforms and system surfaces

| Platform | Notable surfaces |
| --- | --- |
| Web | Edge, Firefox, Chrome and Safari. Push notifications in Chrome, Firefox and Edge only. Theme follows the browser's light or dark setting. Language and time zone follow the system. |
| iOS (iPhone only) | Interactive widgets, icon badge (Incoming, Activity or Priority count), share extension, Apple Shortcuts via App Intents, urgent alarm reminders, Spotlight search. |

> _Diagram (desktop layout · sidebar, main panel, task details) — see the Claude Doc._

The sidebar and main panel follow the help docs. The task-details pane on the right is an assumption; the docs do not say where details open (see Open questions).

## Integrations, sync and non-functional requirements

TaskHive has two clients, web and iOS, that sync through one backend. Only the integrations below are kept.

### Integrations

| Integration | Behaviour |
| --- | --- |
| Email-in | Each user has a unique address; custom addresses are also supported. Forwarded mail becomes a task in Single tasks and Incoming, with attachments as comments. Hashtags in the subject set project, section, responsible person, date, recurrence, reminder, tags and priority (English only). Each section and each task also has its own address. |
| Import from Nozbe (new in TaskHive) | A one-time migration per space through Nozbe's REST API. It brings over projects, sections, tasks, comments and tags. |
| REST API | Header `Authorization: apikey <token>`, JSON. Filter operators such as `[ne]`, `[gt]`, `[lte]`; `fields`, `sortBy` (`-` for descending), `limit` and `offset`. OpenAPI 3.0 spec published. Tokens cover one space or all spaces. |
| Outgoing webhooks | Fire when a project, task or comment is created or updated, limited to items the token owner can access. Sent to one or more URLs. |
| MCP | Remote server with OAuth, for clients such as Claude, ChatGPT and Cursor. TaskHive has the remote server only, as the main channel for agents; there is no local server. |
| OS surfaces | Calendar feed (ICS subscription) and Apple Shortcuts (App Intents) on iOS. Not included: x-callback-url schemes, custom URL-scheme links, and importers other than the one above. |

### Sync and offline

- **Offline-first:** both clients read and write a local database and keep working fully without a connection. Changes made offline sync when the connection returns. If one field was changed in two places, the last change wins. Comments and habit logs are only ever added, so they never conflict.
- **Real-time sync:** while online, changes from other clients, other users and agents appear immediately, pushed by the backend. No manual refresh.
- **Backend:** holds the master copy of all data and passes changes between clients.
- **One API:** the web app, the iOS app and agents all use the same REST API, and MCP is a layer over it. Permissions are checked once, in the API.
- **Agents** work online only, through the backend.
- **Reference:** WatermelonDB is an open-source offline-first database for React and React Native that syncs with a backend. The React web app can use it. The Swift app needs its own local store, for example SwiftData or SQLite, speaking the same sync protocol.

### Non-functional requirements

TaskHive is hosted on the portfolio platform, following the same pattern as the email-triage app (sources: the local `platform` and `email-triage` repos).

| Area | Requirement |
| --- | --- |
| Hosting | Azure, in the same subscription as the other portfolio apps. TaskHive gets its own repo, its own resource group (`rg-taskhive`) and its own Terraform. Its API runs as a container app in the shared Container Apps environment provisioned by the platform repo. |
| Infrastructure as code | Terraform in two configs, as in email-triage. `infra/bootstrap` is applied once, locally, by you: state storage, resource group, and the GitHub Actions OIDC identity. `infra/main` runs only in GitHub Actions: plan on pull requests, apply on merge to `main`. No client secrets, only OIDC federated credentials scoped to the resource group. |
| Region and data residency | EU. Resource group in Poland Central; resources in West Europe wherever Poland Central lacks a service, as email-triage does. |
| Identity and sign-in | The shared Microsoft Entra External ID tenant provisioned by `platform`. TaskHive is one more app registration in it, which gives single sign-on with the other portfolio apps. The tenant ID and authority come from `platform` outputs, copied into TaskHive's GitHub Actions variables. |
| Tenancy | One application instance serves all spaces. Each space is a tenant with isolated data. |
| Secrets | Key Vault with RBAC. Services read secrets through managed identity. No secret values in repos or Terraform state. |
| Observability | Application Insights on a Log Analytics workspace with 30-day retention. |
| Transport | TLS 1.2 or higher on all connections. |
| Engineering rules | The platform rules apply: Jira project PORT, one PR per ticket, ADRs in the repo, and TaskHive added to the C4 model in `platform/workspace.dsl`. |
| Size limits | Attachments up to 200 MB. Comment text up to 12 KB, checklists up to 80 KB. |
| Localisation | UI language follows the browser, with English as the fallback. Time follows the device's time zone. |
| Accessibility | To decide. |

**Architecture decisions:**

- **Database:** Azure SQL serverless with EF Core. Every table carries the space ID, a last-changed timestamp and soft deletes, which offline sync needs.
- **Clients:** a native Swift app for iPhone and a separate React web app. Each keeps its own offline store, and both use the same sync API.
- **Attachments:** Azure Blob Storage, uploaded and downloaded through short-lived signed links.
- **Push notifications:** sent directly through APNs (iPhone, token-auth key in Key Vault) and Web Push (VAPID), the same as meal-math. No Notification Hubs. Runtime and hosting: .NET 11 API and Jobs as container apps, images in GHCR, React web app on Azure Static Web Apps (Free).
- **Compute:** one Azure Container Apps environment shared by all portfolio apps, provisioned in the `platform` repo (an ADR there records it). TaskHive's API is a container app that scales to zero replicas when idle, with no usage charge at zero ([Container Apps scaling](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)). The first request after an idle period waits while a replica starts. Offline-first clients hide this: people keep working locally and sync catches up.
- **Real-time channel:** Azure SignalR Service holds the live connections to clients, so the API can still scale to zero replicas. The Free tier allows 20 concurrent connections and 20,000 messages a day, with no SLA ([pricing](https://azure.microsoft.com/en-us/pricing/details/signalr-service/)). Move to Standard if those limits are reached.
- **Backups:** Azure SQL's built-in automatic backups with point-in-time restore, kept for the default 7 days. Blob Storage soft delete covers deleted attachments.

**To decide for TaskHive:** the email provider used for notification emails and for email-in.

## Open questions

These points are still open.

- [ ] Where do task details open on desktop: a right-hand pane, or in place of the list?
- [ ] What sort options do views offer?
- [ ] What are the API rate limits?
- [ ] What visual tokens does TaskHive use: colours, type scale, icon set?
- [ ] Is there a section board (kanban) view?
- [ ] Are "Project Admin" and "manager" the same role? This spec treats them as one.

## Decision log

Newest first. Each row is one change you asked for, with the sections it touched.

| Date | Decision | Sections affected |
| --- | --- | --- |
| 2026-11-27 | References to the apps this spec was researched from removed, so the spec stands on its own; the only one kept is the import integration. Source links, the Sources section and the research-only open questions removed; the sync-conflict question dropped as already decided. | All |
| 2026-11-25 | Push goes directly through APNs and Web Push, not Notification Hubs, matching meal-math. .NET 11, GHCR and React on Static Web Apps (Free) recorded. Repo is public (MIT). Email provider still to decide. Backlog created in Jira project PA with label taskhive. | Integrations, sync and non-functional requirements |
| 2026-11-23 | Habits get comment threads like tasks. Your AI assistant can log habits through MCP. Off Mode and vacation mode are one break. Habits in shared projects are visible to all members. | Domain model, Core features (Habits), Collaboration, Open questions |
| 2026-11-23 | Habits added: good and bad habits, in a project or standalone like tasks, owned only by their creator, shown in a separate Habits view, with statistics. Included: advanced reminders, Off Mode, end conditions. Not included: timer, mood, health auto-logging, Time of Day, widgets. | Overview, Domain model, Core features, Habit statistics (new), Integrations and sync, UI and UX, Open questions |
| 2026-11-23 | Compute changed from AKS to Azure Container Apps: AKS's always-on minimum (about $180–250 a month) was too expensive. One shared Container Apps environment; apps scale to zero at no charge. | Integrations, sync and non-functional requirements |
| 2026-11-23 | A new task belongs to its creator by default, and a removed member's active tasks go to the space owner. Tasks assigned to you hide the agent statuses and are just completed. | Core features (Agents), Collaboration |
| 2026-11-23 | TaskHive is also a plain to-do app for tasks the owner completes themselves, alongside delegating to agents and collaborators. | Overview |
| 2026-11-23 | Real-time updates through Azure SignalR Service (Free tier to start). Backups: Azure SQL built-in point-in-time restore (7 days) plus Blob Storage soft delete. | Integrations, sync and non-functional requirements |
| 2026-11-23 | Compute: a small AKS cluster shared by all portfolio apps, chosen over Azure Container Apps. The system pool is always on (minimum 2 nodes); TaskHive's user pool and pods scale to zero. | Integrations, sync and non-functional requirements |
| 2026-11-23 | Database: Azure SQL serverless with EF Core. Clients: native Swift iPhone app plus separate React web app. Attachments in Blob Storage; push through Notification Hubs. Compute still under discussion (AKS or Container Apps). | Overview, Integrations, sync and non-functional requirements |
| 2026-11-23 | Offline on both web and iOS confirmed. Sync conflicts: last change wins per field. Hosting follows the platform and email-triage pattern: Azure, Terraform via GitHub Actions OIDC, shared Entra External ID tenant, Key Vault, Application Insights, EU region. | Integrations, sync and non-functional requirements; Open questions |
| 2026-11-23 | Both clients are offline-first, with real-time sync while online. iPhone only, no iPad. One REST API for the web app, the iOS app and agents. One-time import added. Email reports dropped. | Overview, Collaboration, UI and UX, Integrations, Open questions |
| 2026-11-23 | Platforms are web and iOS, synced through the backend. Integrations cut to email-in, REST API, webhooks, remote MCP, calendar feed and Apple Shortcuts. Messages (direct, group and space chat) dropped. | Overview, Domain model, Core features, Collaboration, UI and UX, Integrations, Open questions |
| 2026-11-23 | One instance serves all spaces. Agents never work across spaces. Only owners and admins invite, one space per invitation. Member groups and project groups dropped. Agent credentials separate from user tokens. Built-in AI project generation dropped. | Domain model, Core features, Collaboration, UI and UX, Open questions |
| 2026-11-23 | Added an agent work lifecycle with review, MCP-first agent connection, and a Needs review group in Incoming. Spaces stay, with each space as a tenant and users and spaces many-to-many. TaskHive is free: plans, billing and the Business entity removed. | Overview, Domain model, Core features, Collaboration, Integrations, Open questions |
| 2026-11-23 | Agents are AI agents and people; a task's assignee can be either. Users are one owner plus a few human collaborators. | Overview, Domain model, Collaboration, Open questions |
| 2026-11-23 | TaskHive is a task and agents manager, not a manager for small teams. The meaning of "agent" and the target users are still to decide. | Overview, Open questions |
| 2026-11-23 | Spec re-based as the TaskHive spec. Pricing and hosting marked "To decide". | All |
