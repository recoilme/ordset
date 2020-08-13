# `ordset`

[![GoDoc](https://godoc.org/github.com/recoilme/ordset?status.svg)](https://godoc.org/github.com/recoilme/ordset)

Package ordset provide ordered set, with strings/binary comparator, backed by arrays

## Status

WIP

## Usage

install Go and run ```go get```:

```go
go get github.com/recoilme/ordset
```

## Motivation

Set usualy based on Trees. Trees are:

- use pointers => many allocations
- use pointers => memory fragmentation
- rebalansed on the fly => perfomance degradation, or not safe for traversting

## Architecture

`ordset` based on custom data structure. Data stored ih fixed size arrays, pages: ```[256]string``` and string slice with pages indexes (max/min). Then you put key, `ordset` will store data in descending order with binary comparator.

If pageSize = 4, and insert ```"1","3","5","7","9"``` -  data will look's like:

```go
index: "9","3","1","1"
pages: page0["9", "7", "5", "3"] page1["1","","",""]
```

On overflow - page split on the mid.
On insert: 
 - scan index with binary search
 - scan page with binary search
 - insert

Let insert "6":

```go
index: "9","6","5","3","1","1"
pages: page0["9", "6", "", ""] page1["5","3","",""] page2["1","","",""]
```

That's all. 

### New/Put

Put is **safe** for use from multiple goroutines.

```go
	set := ordset.New()
	set.Put("a")
	set.Put("b")
	fmt.Println(set.Keys())
	//[b a]
```

### Buckets

Buckets are keys with same prefix. Methods of buckets are **safe** for concurrent usage.

```go
	set := ordset.New()
	users := ordset.Bucket(set, "user")//Bucket name may be ommited
	users.Put("rob")
	users.Put("bob")
	users.Put("pike")
	users.Put("alice")
	fmt.Println(users.Keys())
	// output: [rob pike bob alice]
    
	items := ordset.Bucket(set, "item")
	items.Put("003")
	items.Put("042")
	fmt.Println(items.Keys())
	// output: [042 003]
```

### Iterating over keys

`ordset` stores its keys in byte-sorted descending order. This makes sequential iteration over these keys extremely fast. To iterate over keys we'll use a `Cursor`:

```go
	set := New()
	keys := randKeys(7)
	bkt := Bucket(set, "")
	for _, key := range keys {
		bkt.Put(key)
	}
	c := bkt.Cursor()
	for k := c.Last(); k != ""; k = c.Prev() {
		fmt.Printf("[%s] ", k)
	}
	//[6] [5] [4] [3] [2] [1] [0]
```

The cursor allows you to move to a specific point in the list of keys and move forward or backward through the keys one at a time.

The following functions are available on the cursor:

```go
Last()   Move to the last key.
Prev()   Move to the previous key.
```

You must seek to a position using Last() before calling Prev(). If you do not seek to a position then these functions will return a empty key.

Cursor is method of bucket and safe for concurrent usage. Data in cursor ara consistent.

### Benchmark

**BenchmarkParallel:**
```
Put: 10,000 ops over 8 threads in 5ms, 2,094,220/sec, 477 ns/op, 274.9 KB, 28 bytes/op
```

**BenchmarkSequental:**
```
goos: darwin
goarch: amd64
pkg: github.com/recoilme/ordset
BenchmarkKeys-8          	9279559               111 ns/op              89 B/op          0 allocs/op
BenchmarkAddAsc-8                2047423               680 ns/op              39 B/op          0 allocs/op
BenchmarkAddAscBin-8             2069086               673 ns/op              39 B/op          0 allocs/op
BenchmarkAddDesc-8               2908315               545 ns/op              39 B/op          0 allocs/op
BenchmarkAddDescBin-8            3202050               498 ns/op              40 B/op          0 allocs/op
BenchmarkAddRand-8               1204304              1057 ns/op              28 B/op          0 allocs/op
BenchmarkAddRandBin-8            1000000              1001 ns/op              28 B/op          0 allocs/op
BenchmarkParallel-8              1235898              1026 ns/op              29 B/op          0 allocs/op
BenchmarkHas-8           	 1000000              1035 ns/op               0 B/op          0 allocs/op
```

**Left-Leaning Red-Black (LLRB) implementation of 2-3 balanced binary search trees**
(github.com/google/btree)

Sequental:
```
BenchmarkAddRandGoogle-8 1000000              1505 ns/op              36 B/op          0 allocs/op
```

### TODO

 - delete
 - switch from slice on fixed array in index
 - seek()
 - first()
 - next()

## Contact

Vadim Kulibaba [@recoilme](http://t.me/recoilme)

## License

`ordset` source code is available under the MIT [License](/LICENSE).
