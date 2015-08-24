# databasex/sql - Golang database/sql API with Multi Result Set support.

This is a dirty hack to golang's database/sql package.

The motivation to create this package, comes from that the standard database/sql package lacks the support for Multi ResultSet feature. This is not a big issue for common usage, but for users who using stored procedures, they will encounter "Command out of sync" error. 

Developers of `go-sql-driver/mysql` has confirmed that they cannot support this feature until golang's database API change first.

So I made this dirty hack, an independent package named `databasex` to avoid name conflicts. 

### Modifications
This hack based on the following code:

1. source code of `database/sql` package from golang 1.5
   - add a Sibling() method for sql.Rows type.
   - add MoreResults() / Sibling() method to driver.Rows interface
2. source code of `go-sql-driver/mysql` based on commit: 3dd7008ac1529aca1bcd8a9db75228a71ba23cac
  - implement the sql.driver.Rows interfaces

All modifications are listed in the orig.diff file in both sql and mysql repo.

### Usage 

1. `go get -u github.com/databasex/sql`
2. `go get -u github.com/databasex/mysql`
3. For usage, check the (multi_test.go)[https://github.com/databasex/mysql/blob/master/multi_test.go]
