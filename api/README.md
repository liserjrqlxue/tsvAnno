# api
--
    import "github.com/brentp/vcfanno/api"


## Usage

```go
const BOTH = "both_"
```

```go
const INTERVAL = ""
```

```go
const LEFT = "left_"
```

```go
const RIGHT = "right_"
```

```go
var Reducers = map[string]Reducer{
	"self":   Reducer(self),
	"mean":   Reducer(mean),
	"max":    Reducer(max),
	"min":    Reducer(min),
	"concat": Reducer(concat),
	"count":  Reducer(count),
	"uniq":   Reducer(uniq),
	"first":  Reducer(first),
	"flag":   Reducer(vflag),
}
```

#### type Annotator

```go
type Annotator struct {
	Sources []*Source
	Strict  bool // require a variant to have same ref and share at least 1 alt
	Ends    bool // annotate the ends of the variant in addition to the interval itself.
}
```

Annotator holds the information to annotate a file.

#### func  NewAnnotator

```go
func NewAnnotator(sources []*Source, js string, ends bool, strict bool) *Annotator
```
NewAnnotator returns an Annotator with the sources, seeded with some javascript.
If ends is true, it will annotate the 1 base ends of the interval as well as the
interval itself. If strict is true, when overlapping variants, they must share
the ref allele and at least 1 alt allele.

#### func (*Annotator) AnnotateEnds

```go
func (a *Annotator) AnnotateEnds(r interfaces.Relatable, ends string) error
```
AnnotatedEnds makes a new 1-base interval for the left and one for the right end
so that it can use the same machinery to annotate the ends and the entire
interval. Output into the info field is prefixed with "left_" or "right_".

#### func (*Annotator) AnnotateOne

```go
func (a *Annotator) AnnotateOne(r interfaces.Relatable, strict bool, end ...string) error
```
AnnotateOne annotates a relatable with the Sources in an Annotator. In most
cases, no need to specify end (it should always be a single arugment indicting
LEFT, RIGHT, or INTERVAL, used from AnnotateEnds

#### func (*Annotator) SetupStreams

```go
func (a *Annotator) SetupStreams() ([]string, error)
```
SetupStreams takes the query stream and sets everything up for annotation.

#### func (*Annotator) UpdateHeader

```go
func (a *Annotator) UpdateHeader(r HeaderUpdater)
```
UpdateHeader adds to the Infos in the vcf Header so that the annotations will be
reported in the header. func (a *Annotator) UpdateHeader(h HeaderUpdater) {

#### type HeaderUpdater

```go
type HeaderUpdater interface {
	AddInfoToHeader(id string, itype string, number string, description string)
}
```


#### type Reducer

```go
type Reducer func([]interface{}) interface{}
```


#### type Source

```go
type Source struct {
	File string
	Op   string
	Name string
	// column number in bed file or ...
	Column int
	// info name in VCF. (can also be ID).
	Field string
	// 0-based index of the file order this source is from.
	Index int

	Js *otto.Script
	Vm *otto.Otto
}
```

Source holds the information for a single annotation to be added to a query.
Many sources can come from the same file, but each must have their own Source.

#### func (*Source) AnnotateOne

```go
func (src *Source) AnnotateOne(v interfaces.IVariant, vals []interface{}, prefix string)
```

#### func (*Source) IsNumber

```go
func (s *Source) IsNumber() bool
```
IsNumber indicates that we expect the Source to return a number given the op

#### func (*Source) JsOp

```go
func (s *Source) JsOp(v interfaces.IVariant, js *otto.Script, vals []interface{}) string
```
JsOp uses Otto to run a javascript snippet on a list of values and return a
single value. It makes the chrom, start, end, and values available to the js
interpreter.

#### func (*Source) UpdateHeader

```go
func (src *Source) UpdateHeader(r HeaderUpdater, ends bool)
```
func (src *Source) UpdateHeader(h HeaderUpdater, ends bool) {
