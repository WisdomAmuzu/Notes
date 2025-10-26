# Git Per-Folder Identity and Credential Setup

## Global Git Configuration (`~/.gitconfig`)

```ini
[user]
    name = Default Name
    email = default@example.com

[credential]
    namespace = personal

[includeIf "gitdir:~/Projects/"]
    path = ~/.gitconfig-personal

[includeIf "gitdir:~/Company/"]
    path = ~/.gitconfig-work
```

> This file sets the global default identity and credential namespace.
> It also includes folder-specific configs based on your repo location.

---

## Personal Folder Configuration (`~/.gitconfig-personal`)

```ini
[user]
    name = Personal Name
    email = personal@email.com

[credential "personal"]
    namespace = personal
    email = personal@email.com
    name = Personal Name
    helper = manager
```

> Any Git repo under `~/Projects/` will automatically use this identity and credential namespace.

---

## Work Folder Configuration (`~/.gitconfig-work`)

```ini
[user]
    name = Work Name
    email = work@email.com

[credential "work"]
    namespace = work
    email = work@email.com
    name = Work Name
    helper = manager
```

> Any Git repo under `~/Company/` will automatically use this identity and credential namespace.

---

## How to Verify

Go into your repo folder and run:

```bash
git config user.name
git config user.email
git config credential.namespace
```

* Should display **personal identity** for `~/Projects/` repos
* Should display **work identity** for `~/Company/` repos

---

This setup ensures **automatic switching of Git user identity and credentials per folder**.

---

If you want, I can also make a **single-file version** that eliminates the separate `.gitconfig-personal` and `.gitconfig-work` files, keeping everything in **one global file** for simplicity.

Do you want me to create that too?
