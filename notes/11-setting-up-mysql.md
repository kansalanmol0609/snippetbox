You can install mysql on your computer using following commands -

```bash
brew install mysql

sudo apt install mysql-server
```

To connect MySQL from your terminal as the `root` user -

```bash
sudo mysql
```

### Creating our first database

```sql
-- Create a new UTF-8 `snippetbox` database.
CREATE DATABASE snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Switch to using the `snippetbox` database.
USE snippetbox;

-- Create a `snippets` table.
CREATE TABLE snippets(
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

-- Add an index on the created column.
CREATE INDEX idx_snippets_created ON snippets(created);
```

Each record in this table will have an integer `id` field which will act as the unique identifier for the text snippet. It will also have a short text title and the snippet content itself will be stored in the content field. We’ll also keep some metadata about the times that the snippet was created and when it expires.

Let's add some placeholder entries -

```sql
INSERT INTO snippets(title, content, created, expires) VALUES(
    'An old silent pond',
    'An old silent pond...\nA frog jumps into the pond,\nsplash! Silence again.\n\n– Matsuo Bashō',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'Over the wintry forest',
    'Over the wintry\nforest, winds howl in rage\nwith no leaves to blow.\n\n– Natsume Soseki',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
 );

INSERT INTO snippets (title, content, created, expires) VALUES (
    'First autumn morning',
    'First autumn morning\nthe mirror I stare into\nshows my father''s face.\n\n– Murakami Kijo',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 7 DAY)
 );
```

### Creating a new user

From a security point of view, it’s not a good idea to connect to MySQL as the `root` user from a web application. Instead it’s better to create a database user with restricted permissions on the database.

```sql
CREATE USER 'web'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';-- Important: Make sure to swap 'pass' with a password of your own choosing.
ALTER USER 'web'@'localhost' IDENTIFIED BY 'pass';
```

### Test the new user

You should now be able to connect to the snippetbox database as the web user using the following command. When prompted enter the password that you just set -

```bash
mysql -D snippetbox -u web -p
```

If the permissions are working correctly, you should find that you're able to perform `SELECT` and `INSERT` operations on db but other commands such as `DROP TABLE` and `GRANT` will fail -

```sql
SELECT id, title, expires FROM snippets;

DROP TABLE snippets;
```
