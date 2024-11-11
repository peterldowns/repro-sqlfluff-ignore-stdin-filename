Setup and background:

```sh
# sqlfluff --version
sqlfluff, version 3.2.5

# cat .sqlfluff
[sqlfluff]
dialect = postgres

# cat .sqlfluffignore
pkg/database/schema.sql

# cat pkg/database/schema.sql
SELECT col_a a FROM foo; -- lint error, missing AS
```

The observed behavior is correct (expected) for linting/fixing the file by its path. The correct
behavior is to do nothing but return successfully, because the file is ignored by `.sqlfluffignore`.

```sh
# sqlfluff lint pkg/database/schema.sql
WARNING    Exact file path pkg/database/schema.sql was given but it was ignored by an ignore pattern set in .sqlfluffignore, re-run with `--disregard-sqlfluffignores` to not process ignore files.
All Finished 📜 🎉!

# sqlfluff fix pkg/database/schema.sql
==== finding fixable violations ====
WARNING    Exact file path pkg/database/schema.sql was given but it was ignored by an ignore pattern set in .sqlfluffignore, re-run with `--disregard-sqlfluffignores` to not process ignore files.
==== no fixable linting violations found ====
All Finished 📜 🎉!
```


The observed behavior is *incorrect* for linting/fixing the file by stdin but passing its path with `--stdin-filename`. The correct
behavior is to do nothing but return successfully, because the file is ignored by `.sqlfluffignore`. The actual behavior is to fix/lint the
file as if it were not ignored.

```sh
# cat pkg/database/schema.sql | sqlfluff fix --stdin-filename pkg/database/schema.sql -
SELECT col_a AS a FROM foo; -- lint error, missing AS

# cat pkg/database/schema.sql | sqlfluff lint --stdin-filename pkg/database/schema.sql -
== [stdin] FAIL
L:   1 | P:  14 | AL02 | Implicit/explicit aliasing of columns.
                       | [aliasing.column]
All Finished 📜 🎉!
```
