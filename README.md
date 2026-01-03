# sqlite

SQLite database driver for Quadrate.

## Installation

```bash
quadpm get https://github.com/quadrate-language/sqlite
```

## Requirements

- SQLite3 development library (`libsqlite3-dev` on Debian/Ubuntu, `sqlite` on Arch)

## Usage

```qd
use sqlite

fn main() {
    // Open database (creates if not exists)
    ":memory:" sqlite::open! -> db

    // Create table
    "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)"
    db sqlite::exec!

    // Insert with prepared statement
    "INSERT INTO users (name, age) VALUES (?, ?)" db sqlite::prepare! -> stmt
    "Alice" 1 stmt sqlite::bind_text!
    30 2 stmt sqlite::bind_int!
    stmt sqlite::step! drop
    stmt sqlite::finalize

    // Query data
    "SELECT id, name, age FROM users" db sqlite::prepare! -> query
    query sqlite::step! while {
        0 query sqlite::column_int -> id
        1 query sqlite::column_text -> name
        2 query sqlite::column_int -> age
        id print ": " print name print " (" print age print ")" print nl
        query sqlite::step!
    }
    query sqlite::finalize

    db sqlite::close
}
```

## API

### Database Operations

- `open(path:str -- db:ptr)!` - Open database (use ":memory:" for in-memory)
- `close(db:ptr -- )` - Close database connection
- `exec(sql:str db:ptr -- )!` - Execute SQL without results

### Prepared Statements

- `prepare(sql:str db:ptr -- stmt:ptr)!` - Prepare SQL statement
- `step(stmt:ptr -- has_row:i64)!` - Execute and step to next row
- `reset(stmt:ptr -- )!` - Reset statement for re-execution
- `finalize(stmt:ptr -- )` - Free prepared statement

### Parameter Binding

- `bind_text(value:str index:i64 stmt:ptr -- )!` - Bind string (1-based index)
- `bind_int(value:i64 index:i64 stmt:ptr -- )!` - Bind integer
- `bind_float(value:f64 index:i64 stmt:ptr -- )!` - Bind float
- `bind_null(index:i64 stmt:ptr -- )!` - Bind NULL

### Column Access

- `column_count(stmt:ptr -- count:i64)` - Get number of columns
- `column_name(index:i64 stmt:ptr -- name:str)` - Get column name (0-based)
- `column_type(index:i64 stmt:ptr -- type:i64)` - Get column type
- `column_int(index:i64 stmt:ptr -- value:i64)` - Get integer value
- `column_float(index:i64 stmt:ptr -- value:f64)` - Get float value
- `column_text(index:i64 stmt:ptr -- value:str)` - Get text value

### Transactions

- `begin(db:ptr -- )!` - Begin transaction
- `commit(db:ptr -- )!` - Commit transaction
- `rollback(db:ptr -- )!` - Rollback transaction

### Utility

- `last_insert_rowid(db:ptr -- rowid:i64)` - Get last insert rowid
- `changes(db:ptr -- changes:i64)` - Get rows changed

## Error Codes

| Constant | Value | Description |
|----------|-------|-------------|
| ErrOpen | 2 | Failed to open database |
| ErrExec | 3 | SQL execution failed |
| ErrPrepare | 4 | Failed to prepare statement |
| ErrBind | 5 | Failed to bind parameter |
| ErrStep | 6 | Failed to step statement |
| ErrInvalidArg | 7 | Invalid argument |

## Column Types

| Constant | Value | Description |
|----------|-------|-------------|
| TypeInteger | 1 | INTEGER |
| TypeFloat | 2 | FLOAT |
| TypeText | 3 | TEXT |
| TypeBlob | 4 | BLOB |
| TypeNull | 5 | NULL |

## License

MIT
