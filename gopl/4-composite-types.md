# Composite Types
1. Arrays and structs are aggregate types; their values are concatenations of other values in memory. Arrays are homogeneous and structs are heterogeneous.
2. Both arrays and structs are fixed size. In contrast, slices and maps are dynamic data structures that grow as values are added.

## Arrays
1. We can use an array literal to initialize an array with a list of values. if an ellipsis `...` appears in place of the length, the array length is deter- mined by the number of initializers.

    ```
    var q [3]int = [3]int{1, 2, 3}
    q := [...]int{1, 2, 3}  // same as above
    q := []int{1, 2, 3} // q is a slice in this case!
    ```
  
2. It is possible to specify a list of index and value pairs in array literal. In this form, indices can appear in any order and some may be omitted; as before, unspecified values take on the zero value for the element type. 

    ```
    type Currency int
    const (
      USD Currency = iota
      EUR
      GBP
      RMB
    )
    symbol := [...]string{USD: "$", EUR: "9", GBP: "!", RMB: """}
    
    r := [...]int{99: -1}      // this defines an array of 100 int
    ```
    
3. If an array’s element type is comparable then the array type is comparable too, so we may directly compare two arrays of that type using the `==` operator, which reports whether all corresponding elements are equal. 

4. When a function is called, a copy of each argument value is assigned to the corresponding parameter variable, so the function receives a copy, not the original. Passing large arrays in this way can be inefficient, and any changes that the function makes to array elements affect only the copy, not the original. This behavior is different from languages that implicitly pass arrays by reference. So use slices for function paramerters.

## Slices
1. Since a slice contains a pointer to an element of an array, passing a slice to a function permits the function to modify the underlying array elements.
2. Left shift a slice by `n` elements can be achieved by reverse the leading `n` elements, reverse the remaining elements, and then reverse the whole slice. Right shift can be achieved by doing the whole reverse first. This is not most time efficient, but is done in place without extra space.
3. Unlike arrays, slices are not comparable, so we cannot use == to test whether two slices contain the same elements. There are two reasons for the language not providing a deep comparison: 1. slices may refer to itself. 2. slices used as map key.
4. The zero value of a slice type is nil. A nil slice has no underlying array. The only legal slice comparison is against nil
5. The built-in function `make` can be used to create a slice of a specified element type, length, and capacity.
6. The build-in funciton `append`can be used to append items to slices. Internally, it checks the capacity, if greater than length, extend the slice by defining a larger slice and add the new item. They share the same underlying array. If not enough capacity, allocate a new array enough to hold the new element, copy all existing elements and the new element. 
7. The strategy for allocation of new array in `append` is to double the existing length, which can achieve amortized linear complexity. (why?)
8. the built-in function `copy` can be used to copy elements from one slice to another of the same type. The total number of elements that are copied is returned, which is the smaller of the length of destination and source slices.
9. The `append` can accept any number of final arguments. This is defined using `y ...int`, and called with `...y`.

## Maps
1. A map is a reference to a hash table, and a map type is written `map[K]V`, where `K` and `V` are the types of its keys and values.
2. The key type `K` must be comparable using `==`, i.e. slices cannot be keys for a map. Though floating-point numbers are comparable, it’s a bad idea to compare floats for equality.
3. The built-in function `make` can be used to create a map: `ages := make(map[string]int)`
4. Map elements can be removed with the built-in function `delete`: `delete(ages, "alice")`
5. Accessing a map element by subscripting always yields a value. If the key is present in the map, you get the corresponding value; if not, you get the zero value for the element type. To test the presence of a key, subscripting a map can yield two values; the second is a boolean that reports whether the element was present.
6. A map element is not a variable, and we cannot take its addres, because growing a map might cause rehashing of existing elements into new storage locations, thus potentially invalidating the address.
7. `range` can be used to enumerate all key/value pairs in the map, but the order is random, varying from one execution to the next. To enumerate the key/value pairs in order, we must sort the keys explicitly first.
8. `len` can be used to get all the elements in the map.
9. As with slices, maps cannot be compared to each other; the only legal comparison is with `nil`.
10. If we need use any non-comparable type for map keys, e.g. slice, we can encode it into string using some functions (e.g. `fmt.Sprintf`) and use the encoded string for the key.

## Structs
1. The fields of a struct can be addressed: `&person.Position` returns a pointer to the `Position` field of the person.
2. The pointer to the whole struct can be used directly with dot notation, same as using the struct name directly. `(&person).Position` is same as `person.Position`
3. Fields are usually written one per line, with the field’s name preceding its type, but consecutive fields of the same type may be combined. Field order is significant to type identity. 
4. The name of a struct field is exported if it begins with a capital letter; this is Go’s main access control mechanism. A struct type may contain a mixture of exported and unexported fields.
5. A named struct type `S` can’t declare a field of the same type `S`: an aggregate value cannot contain itself. (An analogous restriction applies to arrays.) But `S` may declare a field of the pointer type `*S`, which lets us create recursive data structures like linked lists and trees.
6. The zero value for a struct is composed of the zero values of each of its fields.
7. The struct type with no fields is called the empty struct, written struct{}. It has size zero and carries no information.
8. There are two forms of struct literal. The first form requires that a value be specified for every field, in the right order. The second form is used, in which a struct value is initialized by listing some or all of the field names and their corresponding values.
9. In a call-by-value language like Go, larger struct types are usually passed to or returned from functions indirectly using a pointer, and this is required if the function must modify its argument.
10. If all the fields of a struct are comparable, the struct itself is comparable, so two expressions of that type may be compared using == or !=. Comparable struct types, like other comparable types, may be used as the key type of a map.
11. Go lets us declare a struct field with a type but no name; such fields are called anonymous fields. The type of the field must be a named type or a pointer to a named type (why we need pointer?). With embedding, we can refer to the names at the leaves of the implicit tree without giving the intervening names. Unfortunately, there’s no corresponding shorthand for the struct emedding literal syntax and the struct literal must follow the shape of the type declaration. 
12. Because anonymous fields do have implicit names, you can’t have two anonymous fields of the same type since their names would conflict.
13. Because the name of the field is implicitly determined by its type, so too is the visibility of the field. If an embedded type is not exported (i.e. lower case), so is the field unaccessible.
14. Anonymous fields need not be struct types; any named type or pointer to a named type will do. This allows the outer struct type gains not just the fields of the embedded type but its methods too. This mechanism is the main way that complex object behaviors are composed from simpler ones. Composition is central to object-oriented programming in Go.

## JSON
1. A field tag is a string of metadata associated at compile time with the field of a struct. A field tag may be any literal string, but it is conventionally interpreted as a space-separated list of key:"value" pairs; since they contain double quotation marks, field tags are usually written with raw string literals. 
2. The json key controls the behavior of the encoding/json package, and other encoding/... packages follow this convention. The first part of the json field tag specifies an alternative JSON name for the Go field. The tag may have an additional option, `omitempty`, which indicates that no JSON output should be produced if the field has the zero value for its type (false, here) or is otherwise empty.

    ```
     Year  int  `json:"released"`
     Color bool `json:"color,omitempty"`
    ```

3. By defining suitable Go data structures, we can select which parts of the JSON input to decode and which to discard. 

    ```
    var titles []struct{ Title string }
    if err := json.Unmarshal(data, &titles); err != nil {
        log.Fatalf("JSON unmarshaling failed: %s", err)
    }
    fmt.Println(titles)
    ```

4. The matching process that associates JSON names with Go struct names during unmarshaling is case-insensitive, so it’s only necessary to use a field tag when there’s an underscore in the JSON name but not in the Go name.
