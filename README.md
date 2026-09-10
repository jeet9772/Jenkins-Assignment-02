
# CI/CD Assignment 2 – Jenkins User Authentication & Authorization

Submitted by Jeetendra Singh

**Part 1:** Role-based access control for 3 teams (Developer, Testing, DevOps) across 9 jobs and 3 views, using Jenkins' Role-Based Authorization Strategy.
**Part 2:** Google SSO login for the admin user.

## Part 1: Role-Based Authorization

### Install required plugins

```
Manage Jenkins → Plugins → Available plugins
```

Installed: **Role-based Authorization Strategy** and **Google Login**.

> **Screenshot:** Add your plugin installation screenshot here.

```
Download progress - all success
```

> **Screenshot:** Add your plugin download/progress screenshot here.

### Create the 9 jobs

Each is a Freestyle project with an Execute shell build step:

```bash
echo "Job Name: $JOB_NAME"
echo "Build Number: $BUILD_NUMBER"
```

Repeated for: `dev-1/2/3`, `test-1/2/3`, `devops-1/2/3`.

> **Screenshot:** Add your 9 jobs screenshot here.

> **Screenshot:** Add your Execute Shell configuration screenshot here.

### Create 3 views

List View, filtered by job name pattern for each team:

* **Developer View** → dev-1, dev-2, dev-3
* **Testing View** → test-1, test-2, test-3
* **DevOps View** → devops-1, devops-2, devops-3

> **Screenshot:** Add your views screenshot here.

### Create the users

```
Manage Jenkins → Users → Create User
```

Created: developer-1, developer-2, testing-1, testing-2, devops-1, devops-2, admin-1.

> **Screenshot:** Add your users screenshot here.

### Enable Role-Based Strategy

```
Manage Jenkins → Security → Authorization → Role-Based Strategy → Save
```

This strategy was chosen (over Legacy, Project-based, or Matrix-based) because it lets permissions be assigned by **regex pattern on job names** (`dev-.*`, `test-.*`, `devops-.*`), which maps directly onto the team/job-prefix structure asked for here - Matrix-based would need per-job checkboxes for every user, and Project-based would need per-job role assignment one at a time.

> **Screenshot:** Add your Role-Based Strategy screenshot here.

### Configure Global roles

```
Manage Jenkins → Manage Roles → Global roles
```

* `admin` → Overall/Administer (full access)
* `user-read` → Overall/Read (so logged-in users can at least see the dashboard shell)

> **Screenshot:** Add your Global Roles screenshot here.

### Configure Item roles (the core of the access control)

```
Manage Jenkins → Manage Roles → Item roles
```

| Role        | Pattern     | Permissions                       |
| ----------- | ----------- | --------------------------------- |
| dev-full    | `dev-.*`    | Build, Configure, Read, Workspace |
| dev-view    | `dev-.*`    | Read only                         |
| test-full   | `test-.*`   | Build, Configure, Read, Workspace |
| test-view   | `test-.*`   | Read only                         |
| devops-full | `devops-.*` | Build, Configure, Read, Workspace |
| devops-view | `devops-.*` | Read only                         |

> **Screenshot:** Add your Item Roles screenshot here.

### Assign roles to users - Global roles

```
Manage and Assign Roles → Assign Roles
```

`admin-1` (and `admin`) → `admin`. All other users → `user-read` (basic dashboard access).

> **Screenshot:** Add your Global Role Assignment screenshot here.

### Assign roles to users - Item roles

| User                     | Roles assigned                           |
| ------------------------ | ---------------------------------------- |
| developer-1, developer-2 | `dev-full`                               |
| testing-1, testing-2     | `dev-view` + `test-full`                 |
| devops-1, devops-2       | `dev-view` + `devops-full` + `test-view` |

This matches the requirement exactly: developers only touch dev jobs; testers get full control of test jobs and can view dev jobs; devops gets full control of devops jobs and can view both dev and test jobs.

> **Screenshot:** Add your Item Role Assignment screenshot here.

## Verify using every user login

### developer-1

Only sees `dev-1/2/3`, only the "Developer View" tab.

> **Screenshot:** Add your developer-1 login screenshot here.

### developer-2

Same as developer-1.

> **Screenshot:** Add your developer-2 login screenshot here.

### devops-1

Sees all 9 jobs (dev, devops, test) across all 3 view tabs, but build (▶) buttons only appear on devops jobs.

> **Screenshot:** Add your devops-1 login screenshot here.

### devops-2

Same as devops-1.

> **Screenshot:** Add your devops-2 login screenshot here.

### testing-1

Sees dev + test jobs only (no devops), build buttons only on test jobs.

> **Screenshot:** Add your testing-1 login screenshot here.

### testing-2

Same as testing-1.

> **Screenshot:** Add your testing-2 login screenshot here.

## Part 2: Enable Google SSO for Admin

### Create a new Google Cloud project

```
console.cloud.google.com → New Project: "Jenkins-SSO"
```

> **Screenshot:** Add your Google Cloud project screenshot here.

### Configure OAuth consent screen

App name "Jenkins SSO", support email set.

> **Screenshot:** Add your OAuth consent screen screenshot here.

Scopes requested: `userinfo.email`, `userinfo.profile`, `openid`.

> **Screenshot:** Add your OAuth scopes screenshot here.

### Create credentials (OAuth client ID)

```
APIs & Services → Credentials → Create Credentials → OAuth client ID
Application type: Web application
```

> **Screenshot:** Add your OAuth client ID configuration screenshot here.

Client ID and secret generated:

```
Client ID:     <your-client-id>
Client Secret: <your-client-secret>
```

> **Screenshot:** Add your generated OAuth credentials screenshot here.

### Configure Jenkins Security Realm

```
Manage Jenkins → Security → Authentication → Security Realm → Login with Google
```

Client ID and Client Secret pasted in.

> **Screenshot:** Add your Jenkins Google Login configuration screenshot here.

### Grant the admin's Google email the admin role

Added `<your-google-email>` (admin's Google account) to Global roles and checked `admin`.

> **Screenshot:** Add your admin Google email role assignment screenshot here.

### Login flow via Google

"Sign in with Google" prompt, continuing to "Jenkins SSO".

> **Screenshot:** Add your Google Sign-In prompt screenshot here.

### Successfully logged into Jenkins via Google

Logged in as the admin Google account with full profile access - confirming Google SSO works end to end for the admin user.

> **Screenshot:** Add your successful Google SSO login screenshot here.

## Note on Authorization Strategies

Went through all four before picking one:

| Strategy                  | Why not used here                                                                                                                                                                    |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Legacy mode**           | No per-user control at all - any logged-in user gets full access. Not usable for team separation.                                                                                    |
| **Project-based Matrix**  | Per-job permission grids, but each job needs its own manual setup - doesn't scale to "all jobs starting with dev-" cleanly.                                                          |
| **Matrix-based**          | Global grid only, no per-job or per-pattern granularity - can't restrict a user to just the `devops-*` jobs.                                                                         |
| **Role-Based Strategy** ✅ | Supports regex-pattern item roles (`dev-.*`, `test-.*`, `devops-.*`), so one role definition covers a whole team's jobs and new jobs matching the pattern are automatically covered. |
