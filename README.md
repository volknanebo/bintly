# Bintly (super fast binary serialization for go) 

[![GoReportCard](https://goreportcard.com/badge/github.com/viant/bintly)](https://goreportcard.com/report/github.com/viant/bintly)
[![GoDoc](https://godoc.org/github.com/viant/bintly?status.svg)](https://godoc.org/github.com/viant/bintly)

This library is compatible with Go 1.11+

Please refer to [`CHANGELOG.md`](CHANGELOG.md) if you encounter breaking changes.

- [Motivation](#motivation)
- [Introduction](#introduction)
- [Contribution](#contributing-to-bintly)
- [License](#license)

## Motivation

The goal of library to provide super fast binary oriented decoding and encoding capability for any go data type, critical
for low latency applications.


## Introduction

Typical streamlined binary serialization format store primitive types with their native size, and all collection type
got pre seeded with the repeated data size. Imagine the follow struct:

```go
type Employee struct {
	ID int
	Name string
	RolesIDs []int
	Titles []string
    DeptIDs []int
}

var emp := Employee{
    ID: 100,
    Name: "test",
    RolesIDs: []int{1000,1002,1003},
    Titles: []string{"Lead", "Principal"},
    DeptIDs: []int{10,13},
}
```
This maps to the following binary stream representation:
```
100,4,test,3,1000,1002,1003,2,4,Lead,9,Principal,2,10,13
```

In examples presented coma got preserved only for visualisation, also numeric/alphanumerics usage is for simplification.

When decoding this binary format each repeated type requires new memory allocation, in this case 6 allocations:
3 for slices, and 3 for string type. 

Since it's possible to copy any primitive slice to memory back and forth, we can go about binary serialization way faster than the originally presented approach.
Instead of allocation memory for each repeated type (string,slice), we could simply reduce number allocation to number of 
primitive data type used + 1 to track allocations.
In that case binary data stream for emp variable will look like the following. 

```yaml
alloc: [4,3,2,4,9,2] 
ints: [100,1000,1002,1003,10,13]
uint8s: [test,Lead,Principal]
```


## Usage

```go
func Example_Marshal() {
	emp := Employee{
		ID:       100,
		Name:     "test",
		RolesIDs: []int{1000, 1002, 1003},
		Titles:   []string{"Lead", "Principal"},
		DeptIDs:  []int{10, 13},
	}
	data, err := bintly.Marshal(emp)
	if err != nil {
		log.Fatal(err)
	}
	clone := Employee{}
	err = bintly.Unmarshal(data, &clone)
	if err != nil {
		log.Fatal(err)
	}
}

//DecodeBinary decodes data to binary stream
func (e *Employee) DecodeBinary(stream *bintly.Reader) error {
	stream.Int(&e.ID)
	stream.String(&e.Name)
	stream.Ints(&e.RolesIDs)
	stream.Strings(&e.Titles)
	stream.Ints(&e.DeptIDs)
	return nil
}

//EncodeBinary encodes data from binary stream
func (e *Employee) EncodeBinary(stream *bintly.Writer) error {
	stream.Int(e.ID)
	stream.String(e.Name)
	stream.Ints(e.RolesIDs)
	stream.Strings(e.Titles)
	stream.Ints(e.DeptIDs)
	return nil
}
```

#### Working with Map


#### Working with Objects


### Benchmark

Benchmark uses [BenchStruct](stress/bench.go)  where slices got populated with 80 random items.

```bash
BenchmarkUnmarshalBintly
BenchmarkUnmarshalBintly-16    	  644394	      1626 ns/op	    3698 B/op	       6 allocs/op
BenchmarkMarshalBintly
BenchmarkMarshalBintly-16      	 1160833	      1165 ns/op	    2305 B/op	       1 allocs/op
BenchmarkUnmarshalBinary
BenchmarkUnmarshalBinary-16    	  331450	      3502 ns/op	    3088 B/op	      74 allocs/op
BenchmarkMarshalBinary
BenchmarkMarshalBinary-16      	  211546	      5565 ns/op	    4536 B/op	       7 allocs/op
BenchmarkMarshalGob
BenchmarkMarshalGob-16         	   89200	     12076 ns/op	   11107 B/op	      39 allocs/op
BenchmarkMarshalCbor
BenchmarkMarshalCbor-16        	  211923	      5518 ns/op	    1793 B/op	       1 allocs/op
BenchmarkUnmarshalCbor
BenchmarkUnmarshalCbor-16      	   99536	     10938 ns/op	    3424 B/op	      80 allocs/op
BenchmarkUnMarshalGob
BenchmarkUnMarshalGob-16       	   43074	     28224 ns/op	   13608 B/op	     318 allocs/op
BenchmarkJSONUnmarshal
BenchmarkJSONUnmarshal-16      	   17559	     64142 ns/op	   15360 B/op	     309 allocs/op
BenchmarkJSONMarshal
BenchmarkJSONMarshal-16        	   56258	     20362 ns/op	    4210 B/op	       2 allocs/op  
```


<a name="License"></a>
## License

The source code is made available under the terms of the Apache License, Version 2, as stated in the file `LICENSE`.

Individual files may be made available under their own specific license,
all compatible with Apache License, Version 2. Please see individual files for details.

<a name="Credits-and-Acknowledgements"></a>

## Contributing to Bintly

Bintly is an open source project and contributors are welcome!

See [TODO](TODO.md) list

## Credits and Acknowledgements

**Library Author:** Adrian Witas

