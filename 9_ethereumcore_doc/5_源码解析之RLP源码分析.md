##### 源码分析之RLP源码解析

RLP是Recursive Length Prefix的简写。是以太坊中的序列化方法，以太坊的所有对象都会使用RLP方法序列化为字节数组。RLP源码相关的代码都在rlp目录下，功能较为单一，比较好理解。

- decode.go ：解码器，负责把RLP数据解码为go的数据结构
- decode_tail_test.go ：解码器测试代码文件
- decode_test.go ：解码器测试代码文件
- doc.go：文档代码
- encode.go ：编码器，负责把Go数据结构序列化为字节数组
- encode_example_test.go ：编码器测试代码文件
- raw.go ：未解码的RLP数据
- raw_test.go ：测试代码文件
- typecache.go：类型缓存，类型缓存记录了类型->（编码器｜解码器）的内容。

##### typecache.go : 如何根据类型找到对应的编码器和解码器

在C++或者Java中，都存在重载，如下示例：

```
string encode(int);
string encode(string);
string encode(struct test*);
```

但是Go语言本身不支持重载，也没有范型，所以函数的分派需要我们自己来实现。typecache主要就是用来实现该功能的一个文件，typecache通过自身的类型来快速的找到自己的编码器函数和解码器函数。核心的数据结构如下：

```
var(
	typeCacheMutex sync.RWMutex   //读写锁，用来在多线程的时候保护typeCache 这个map
	typeCache = make(map[typekey]*typeinfo) //核心数据结构，保存了类型->编解码器函数
)
type typeinfo struct{
    decoder
    writer
}
type typekey struct{
    reflect.Type
    tags
}
```

上述核心数据结构中，核心数据结构就是typeCache这个Map，Map的key是类型，value是对应的编码和解码器结构体对象。

用户通过两个方法可以获取编码和解码器函数：

```
func cachedTypeInfo(typ reflect.Type, tags tags) (*typeinfo, error) {
	typeCacheMutex.RLock()
	info := typeCache[typekey{typ, tags}]
	typeCacheMutex.RUnlock()
	if info != nil {
		return info, nil
	}
	// not in the cache, need to generate info for this type.
	typeCacheMutex.Lock()
	defer typeCacheMutex.Unlock()
	return cachedTypeInfo1(typ, tags)
}

func cachedTypeInfo1(typ reflect.Type, tags tags) (*typeinfo, error) {
	key := typekey{typ, tags}
	info := typeCache[key]
	if info != nil {
		// another goroutine got the write lock first
		return info, nil
	}
	// put a dummmy value into the cache before generating.
	// if the generator tries to lookup itself, it will get
	// the dummy value and won't call itself recursively.
	typeCache[key] = new(typeinfo)
	info, err := genTypeInfo(typ, tags)
	if err != nil {
		// remove the dummy value if the generator fails
		delete(typeCache, key)
		return nil, err
	}
	*typeCache[key] = *info
	return typeCache[key], err
}
```

getTypeInfo方法是用来生成新的类型的解码器和编码器，如下函数所示：

```
func genTypeInfo(typ reflect.Type, tags tags) (info *typeinfo, err error) {
	info = new(typeinfo)
	if info.decoder, err = makeDecoder(typ, tags); err != nil {
		return nil, err
	}
	if info.writer, err = makeWriter(typ, tags); err != nil {
		return nil, err
	}
	return info, nil
}
```

在genTypeInfo函数中，调用了makeDecoder和makeWriter两个函数用来指定分别生成解码器和编码器。makeDecoder生成的是解码器；makeWriter方法生成的是编码器。两个函数详情如下：

```
func makeDecoder(typ reflect.Type, tags tags) (dec decoder, err error) {
	kind := typ.Kind()
	switch {
	case typ == rawValueType:
		return decodeRawValue, nil
	case typ.Implements(decoderInterface):
		return decodeDecoder, nil
	case kind != reflect.Ptr && reflect.PtrTo(typ).Implements(decoderInterface):
		return decodeDecoderNoPtr, nil
	case typ.AssignableTo(reflect.PtrTo(bigInt)):
		return decodeBigInt, nil
	case typ.AssignableTo(bigInt):
		return decodeBigIntNoPtr, nil
	case isUint(kind):
		return decodeUint, nil
	case kind == reflect.Bool:
		return decodeBool, nil
	case kind == reflect.String:
		return decodeString, nil
	case kind == reflect.Slice || kind == reflect.Array:
		return makeListDecoder(typ, tags)
	case kind == reflect.Struct:
		return makeStructDecoder(typ)
	case kind == reflect.Ptr:
		if tags.nilOK {
			return makeOptionalPtrDecoder(typ)
		}
		return makePtrDecoder(typ)
	case kind == reflect.Interface:
		return decodeInterface, nil
	default:
		return nil, fmt.Errorf("rlp: type %v is not RLP-serializable", typ)
	}
}

// makeWriter creates a writer function for the given type.
func makeWriter(typ reflect.Type, ts tags) (writer, error) {
	kind := typ.Kind()
	switch {
	case typ == rawValueType:
		return writeRawValue, nil
	case typ.Implements(encoderInterface):
		return writeEncoder, nil
	case kind != reflect.Ptr && reflect.PtrTo(typ).Implements(encoderInterface):
		return writeEncoderNoPtr, nil
	case kind == reflect.Interface:
		return writeInterface, nil
	case typ.AssignableTo(reflect.PtrTo(bigInt)):
		return writeBigIntPtr, nil
	case typ.AssignableTo(bigInt):
		return writeBigIntNoPtr, nil
	case isUint(kind):
		return writeUint, nil
	case kind == reflect.Bool:
		return writeBool, nil
	case kind == reflect.String:
		return writeString, nil
	case kind == reflect.Slice && isByte(typ.Elem()):
		return writeBytes, nil
	case kind == reflect.Array && isByte(typ.Elem()):
		return writeByteArray, nil
	case kind == reflect.Slice || kind == reflect.Array:
		return makeSliceWriter(typ, ts)
	case kind == reflect.Struct:
		return makeStructWriter(typ)
	case kind == reflect.Ptr:
		return makePtrWriter(typ)
	default:
		return nil, fmt.Errorf("rlp: type %v is not RLP-serializable", typ)
	}
}
```

以上的makeDecoder方法和makeWriter方法分别在文件中decode.go和encode.go文件。

在以上的两个函数中，对于基本的数据类型，可以直接调用相应的方法；对于引用类型的，则需要复杂一点，比如结构体的处理如下：

```
func makeStructWriter(typ reflect.Type) (writer, error) {
	fields, err := structFields(typ)
	if err != nil {
		return nil, err
	}
	writer := func(val reflect.Value, w *encbuf) error {
		lh := w.list()
		for _, f := range fields {
			//f是field结构，f.info是typeinfo的指针，所以这里其实是调用字段的编码器方法
			if err := f.info.writer(val.Field(f.index), w); err != nil {
				return err
			}
		}
		w.listEnd(lh)
		return nil
	}
	return writer, nil
}
```

生成的fields属于field类型。field定义如下：

```
type field struct {
	index int
	info  *typeinfo
}
```

##### encode.go：编码器

encode.go文件主要就是用来实现编码器的和编码相关的功能。

```
var (
	EmptyString = []byte{0x80}
	EmptyList = []byte{0xC0}
)
```

定义了空字符串和空List的值，分别是0x80和0xC0。而整形的0值的对应值也是0x80。

在encode文件中，还定义了一个最重要的encode接口。

```
// Encoder is implemented by types that require custom
// encoding rules or want to encode private fields.
type Encoder interface {
	// EncodeRLP should write the RLP encoding of its receiver to w.
	// If the implementation is a pointer method, it may also be
	// called for nil pointers.
	//
	// Implementations should generate valid RLP. The data written is
	// not verified at the moment, but a future version might. It is
	// recommended to write only a single value but writing multiple
	// values or no value at all is also permitted.
	EncodeRLP(io.Writer) error
}
```

在encode文件中，最最重要的一个方法要属Encode方法，源码定义如下：

```
func Encode(w io.Writer,val interface{})error{
    if outer,ok := w.(*encbuf);ok{
		return outer.encode(val)
	}
	eb := encbufPool.Get().(*encbuf)
	defer encbufPool.Put(eb)
	eb.reset()
	if err := eb.encode(val);err != nil{
		return err
	}
	return eb.toWriter(w)
}
func (w *encbuf) encode(val interface{}) error {
	rval := reflect.ValueOf(val)
	ti, err := cachedTypeInfo(rval.Type(), tags{})
	if err != nil {
		return err
	}
	return ti.writer(rval, w)
}
```

Encode方法供大多数的EncodeRLP方法调用。在Encode方法中，首先会获取encbuf对象outer，然后调用encode方法进行编码。在encode方法中，首先获取对象的反射类型，然后根据反射类型获取到编码器，然后调用编码器的writer的方法。从而和typecache结合起来。

##### 关于encbuf的介绍

encobuf大意为encode buff的缩写。encbuf是定义在encode.go文件中的一个数据结构，如下所示：

```
type encbuf struct {
	str     []byte      // string data, contains everything except list headers
	lheads  []*listhead // all list headers
	lhsize  int         // sum of sizes of all encoded list headers
	sizebuf []byte      // 9-byte auxiliary buffer for uint encoding
}
type listhead struct {
	offset int // index of this header in string data
	size   int // total size of encoded data (including list headers)
}
```

encbuf结构体出现在encode方法中，再结合其名字命名，我们大致可以确定其就是在编码的过程中充当buffer的作用。在如上定义中，str为byte数组类型，代表要编码的内容，除了列表的头部。列表的头部记录在lheads字段中。lhsize字段纪录了lheads的长度，sizebuf是9个字节大小的辅助buffer，专门用来处理uint的编码的。listheads两个字段组成，offset字段纪录了列表数据在str字段的哪个位置，size字段记录了包含列表头的编码后的数据的总长度。

##### Writer介绍

```
func writeBool(val reflect.Value, w *encbuf) error {
	if val.Bool() {
		w.str = append(w.str, 0x01)
	} else {
		w.str = append(w.str, 0x80)
	}
	return nil
}
func writeString(val reflect.Value, w *encbuf) error {
	s := val.String()
	if len(s) == 1 && s[0] <= 0x7f {
		// fits single byte, no string header
		w.str = append(w.str, s[0])
	} else {
		w.encodeStringHeader(len(s))
		w.str = append(w.str, s...)
	}
	return nil
}
```

可以看到，一系列的writeXX方法就是将要编码的内容调用append方法填充到encbuf中。

##### 解码器decode.go

和encode流程相反，decode的功能作用主要就是解码。如下为代码部分：

```
func makeDecoder(typ reflect.Type, tags tags) (dec decoder, err error) {
	kind := typ.Kind()
	switch {
	case typ == rawValueType:
		return decodeRawValue, nil
	case typ.Implements(decoderInterface):
		return decodeDecoder, nil
	case kind != reflect.Ptr && reflect.PtrTo(typ).Implements(decoderInterface):
		return decodeDecoderNoPtr, nil
	case typ.AssignableTo(reflect.PtrTo(bigInt)):
		return decodeBigInt, nil
	case typ.AssignableTo(bigInt):
		return decodeBigIntNoPtr, nil
	case isUint(kind):
		return decodeUint, nil
	case kind == reflect.Bool:
		return decodeBool, nil
	case kind == reflect.String:
		return decodeString, nil
	case kind == reflect.Slice || kind == reflect.Array:
		return makeListDecoder(typ, tags)
	case kind == reflect.Struct:
		return makeStructDecoder(typ)
	case kind == reflect.Ptr:
		if tags.nilOK {
			return makeOptionalPtrDecoder(typ)
		}
		return makePtrDecoder(typ)
	case kind == reflect.Interface:
		return decodeInterface, nil
	default:
		return nil, fmt.Errorf("rlp: type %v is not RLP-serializable", typ)
	}
}
// Decode decodes a value and stores the result in the value pointed
// to by val. Please see the documentation for the Decode function
// to learn about the decoding rules.
func (s *Stream) Decode(val interface{}) error {
	if val == nil {
		return errDecodeIntoNil
	}
	rval := reflect.ValueOf(val)
	rtyp := rval.Type()
	if rtyp.Kind() != reflect.Ptr {
		return errNoPointer
	}
	if rval.IsNil() {
		return errDecodeIntoNil
	}
	info, err := cachedTypeInfo(rtyp.Elem(), tags{})
	if err != nil {
		return err
	}

	err = info.decoder(s, rval.Elem())
	if decErr, ok := err.(*decodeError); ok && len(decErr.ctx) > 0 {
		// add decode target type to error so context has more meaning
		decErr.ctx = append(decErr.ctx, fmt.Sprint("(", rtyp.Elem(), ")"))
	}
	return err
}
```

在上面的两个方法，Decode()方法属于Stream流对象，因此是有明确的数据流对象时，或者用户主动创建时来调用改方法用的；而对于makeDecoder方法，是当重载出现时，会根据switch case原则，选择对应的类型，进行解码器构造，该方法返回一个构造器。同样，我们分析结构体的解码器构造过程：

```
func makeStructDecoder(typ reflect.Type) (decoder, error) {
	fields, err := structFields(typ)
	if err != nil {
		return nil, err
	}
	dec := func(s *Stream, val reflect.Value) (err error) {
		if _, err := s.List(); err != nil {
			return wrapStreamError(err, typ)
		}
		for _, f := range fields {
			err := f.info.decoder(s, val.Field(f.index))
			if err == EOL {
				return &decodeError{msg: "too few elements", typ: typ}
			} else if err != nil {
				return addErrorContext(err, "."+typ.Field(f.index).Name)
			}
		}
		return wrapStreamError(s.ListEnd(), typ)
	}
	return dec, nil
}
```

在makeDecoder方法中，首先获取到传入的反射类型对应的类型，根据knind对应到结构体reflect.Struct,执行makeStructDecoder方法构造属于结构体的解码器。同样的流程，先根据反射类型拿到所有的字段数据，然后遍历，对每个字段进行解码。这个过程有List和ListEnd的过程。

##### 关于Stream结构的分析

在上面Decode方法中，我们可以看到，Decode方法属于Stream结构体对象才能调用的方法。总结来说就是，解码器的其他代码和编码器的结构差不多，但是有一个特殊的结构是Stream。Stream结构体主要是用来读取流式数据的方式的解码RLP的一个辅助类。如前文所述，解码过程大致就是通过Kind方法获取到需要解码的对象的类型和长度，然后根据类型和长度进行数据的解码。那么我们如何处理结构体的字段又是结构体的数据呢，在之前我们说过有一个List和ListEnd方法，恰好这两个方法就属于Stream结构体对象所属，因此，我们来专门看一下是如何实现的。

```
type Stream struct {
	r ByteReader

	// number of bytes remaining to be read from r.
	remaining uint64
	limited   bool

	// auxiliary buffer for integer decoding
	uintbuf []byte

	kind    Kind   // kind of value ahead
	size    uint64 // size of value ahead
	byteval byte   // value of single byte in type tag
	kinderr error  // error from last readKind
	stack   []listpos
}
type listpos struct{ pos, size uint64 }

// List starts decoding an RLP list. If the input does not contain a
// list, the returned error will be ErrExpectedList. When the list's
// end has been reached, any Stream operation will return EOL.
func (s *Stream) List() (size uint64, err error) {
	kind, size, err := s.Kind()
	if err != nil {
		return 0, err
	}
	if kind != List {
		return 0, ErrExpectedList
	}
	s.stack = append(s.stack, listpos{0, size})
	s.kind = -1
	s.size = 0
	return size, nil
}

// ListEnd returns to the enclosing list.
// The input reader must be positioned at the end of a list.
func (s *Stream) ListEnd() error {
	if len(s.stack) == 0 {
		return errNotInList
	}
	tos := s.stack[len(s.stack)-1]
	if tos.pos != tos.size {
		return errNotAtEOL
	}
	s.stack = s.stack[:len(s.stack)-1] // pop
	if len(s.stack) > 0 {
		s.stack[len(s.stack)-1].pos += tos.size
	}
	s.kind = -1
	s.size = 0
	return nil
}
```

- List方法：先调用Kind方法获取类型和长度，类型不匹配会返回错误；在接下来接着把一个listpos对象压入到堆栈，这里是核心。该对象的pos字段纪录了当前这个list已经读取了多少字节的数据，所以刚开始的时候为0，size字段纪录了这个list对象一共需要读取多少字节数据。这样在处理后续的每一个字段的时候，每读取一些字节，就会增加pos这个字段的值，处理到最后就会对比pos字段和size字段是否相等，如果不相等，则抛出异常。
- ListEnd方法：如果当前读取的数据数量pos不等于声明的数据长度size，抛出异常；然后对堆栈进行pop操作，如果当前栈不为空，那么就在堆栈的栈顶的pos加上当前处理完毕的数据长度。