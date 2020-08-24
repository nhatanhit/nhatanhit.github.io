---
layout: post
title: Install Postgres on MAC
comments: false
bigimg: /img/path.jpg
tags: [postgres]
---

# Download and install PostgreSQL
Remove previous versions of PostgreSQL
```bash
brew uninstall --force postgresql
```

Delete all Files of Postgres
```bash
rm -rf /usr/local/var/postgres
```

Install Postgres with Homebrew
```bash
brew install postgres
```

Start PostgreSQL server
```bash
pg_ctl -D /usr/local/var/postgres start
```

Start PostgreSQL by homebrew
```bash
brew services start postgresql
```

# Install PGAdmin

Download PGAdmin

Run the setup, copy PgAdmin.app into Application Directory

# Create Role to access

```bash
/usr/local/opt/postgres/bin/createuser -s postgres
```
# switch to postgres user

```bash
psql -U postgres
```

