# Wiring this into AWX (the web GUI)

Goal: a single AWX dashboard with friendly buttons — Create Domain, Add OU,
Add User, Add Group, Add Admin, Join Computer, List OUs — each opening a form.

## 1. Execution Environment (important)

WinRM needs `pywinrm` (and `pykerberos` if you use Kerberos) inside the AWX
Execution Environment. The default EE may not include them. Build a custom EE
that pip-installs `pywinrm` and includes the `microsoft.ad`, `ansible.windows`,
and `community.windows` collections (point it at this repo's `requirements.yml`).

## 2. Project

Push this repository to git, then create a **Project** in AWX pointing at it.
On sync, AWX reads `requirements.yml` and installs the collections.

## 3. Inventory

Create an **Inventory** with two groups, `dc` and `windows_clients`, matching
`inventory/hosts.yml`. Add your hosts.

## 4. Credentials (the credential split)

Create **Machine Credentials** and attach the right one per template:

- **DC – local admin** -> used only by *Create Domain* (no domain exists yet).
- **DC – domain admin** -> used by every *manage* template and *List OUs*.
- **Client – local admin** -> used by *Join Computer*.

AWX injects these as `ansible_user`/`ansible_password`, so no passwords sit in
the repo.

## 5. Job Templates + Surveys

Create one **Job Template** per playbook. For each: pick the inventory, the
playbook, and the matching credential, then turn on **Survey** and import the
matching spec from `awx/surveys/`. Because each survey's variable names match
the playbook's prompts, the form fills the values and the prompts are skipped.

Friendly touches already baked into the surveys: scope/category/admin-group are
dropdowns; passwords are masked fields; optional fields default to blank.

A note on OU dropdowns: AWX survey choices are static, so the OU fields are
text. Give users the *List OUs* template so they can see valid names first. A
truly live OU dropdown needs a custom web app (it can query AD on the fly).

## 6. Optional: a Workflow

Chain *Create Domain -> Add OU -> Add User* into a **Workflow Template** for a
one-click full build, while keeping the individual buttons for day-to-day use.
