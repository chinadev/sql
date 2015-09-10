## Golang `database/sql` with stored procedure support.

This is a dirty hack to golang's **`database/sql`** package. 

The motivation to create this package, comes from that the standard **`database/sql`** package lacks the support for Multi ResultSet feature. This is not a big issue for common usage, but for users who using stored procedures, they will encounter "Command out of sync" error. 

Developers of **`go-sql-driver/mysql`** has confirmed that they cannot support this feature until golang's database API change first. So, it will remain dirty until Golang provide its offical way to support multiResultSet returned by stored procedure (or multi statements).

I named this package to an independent package named `databasex` to avoid name conflicts. If you like, you can directly patch the source code of **`database/sql`** and  **`go-sql-driver/mysql`** packages.

### Features
- Support return multi result sets in text protocol , ex `CALL sp_test(1,2,3)`
- Support return multi result sets in prepared statement protocol, ex `CALL sp_test(?,?,?)`
- Supoort return out paramater of stored procedure as a new Result Set
- For usage, reference [multi_test.go](https://database/mysql/blob/master/multi_test.go)

### API Changes
- **`database/sql/Rows`** type added one method.

```go
	// Sibling forward the next result set in current context.
	//
	// To support multi result set feature, append clientMultiResults=true
	// to DSN string passed to sql.Open()
	//
	// Currently only support stored procedure to generate multi resultsets,
	// using multi statements also possible , welcome to contribue ;-）
	func (rs *Rows) Sibling() (Result, bool)
   ```
   
- **`database/sql/driver/Rows`** interface added 2 methods.
```go
	// MoreResults returns if there are other result sets exists
	//
	// After iterate the current result set and Next() returns false,
	// then call this method to determine if there are some onther result
	// sets exists, if it does, call Sibling to change to that Rows.
	MoreResults() bool

	// Sibling is called to pululate multi results in one query.
	//
	// Sibling should return io.EOF when there are no more results
	Sibling() (Rows, error)
```

### Modifications
This hack based on the following code:

1. source code of **`database/sql`** package from golang 1.5
   - add a Sibling() method for sql.Rows type.
   - add MoreResults() / Sibling() method to driver.Rows interface
2. source code of **`go-sql-driver/mysql`** based on commit: 3dd7008ac1529aca1bcd8a9db75228a71ba23cac
  - implement the sql.driver.Rows interfaces

All modifications are listed in the orig.diff file in both sql and mysql repo.
- [Changes to go's database/sql package](https://database/sql/blob/master/database.patch)
- [Changes to go-sql-driver/mysql pakcage](https://database/mysql/blob/master/orig.diff)

### Usage 

1. `go get -u github.com/databasex/sql`
2. `go get -u github.com/databasex/mysql`
3. cd $GOROOT/src/database/sql, then apply the patch `patch -p1 < $GOROOT/src/github.com/databasex/sql/database.patch`
3. Replace `_ go-sql-driver/mysql` imports with **`_ github.com/database/mysql`**
4. When calling sql.Open, append `"&clientMultiResults=true"` to your DSN.
5. To enable return multi result sets with prepared statement, append `"&clientPSMultiResults=true"` to your DSN.
6. For detail, reference [multi_test.go](https://database/mysql/blob/master/multi_test.go)
