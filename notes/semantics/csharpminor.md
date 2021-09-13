# C Sharp Minor
* Defined in `CSharpMinor.v`
* Binary Ops the same as CMinor
```
Definition binary_operation : Type := Cminor.binary_operation.
```

```
Definition eval_binop := Cminor.eval_binop.
```

* `eval_binop`
```
  | Oshll => Some (Val.shll arg1 arg2)
  | Oshrl => Some (Val.shrl arg1 arg2)
  | Oshrlu => Some (Val.shrlu arg1 arg2)
```

## Val.shll, Val.shrl, and Val.shrlu
### Val.shll
* If the first value is a Vlong and the second is a Vint,
  and if the second value is less than
  an Int64 wordsize
  * Perform Int64.shl'
  * Otherwise fail

#### Int64.shl'
```
Definition shl' (x: int) (y: Int.int): int :=
  repr (Z.shiftl (unsigned x) (Int.unsigned y)).
```

### Val.shrl
* If the first value is a Vlong and the second is a Vint,
  and the second is less than an Int64 wordsize
  * Perform Int64.shr'
  * Otherwise fail

#### Int64.shr'
```
Definition shr' (x: int) (y: Int.int): int :=
  repr (Z.shiftr (signed x) (Int.unsigned y)).
```

### Val.shrlu
* If the first value is a Vlong and the second is a Vint,
  and the second is less than an Int64 wordsize
  * Perform Int64.shru'
  * Otherwise fail

#### Int64.shru'
```
Definition shru' (x: int) (y: Int.int): int :=
  repr (Z.shiftr (unsigned x) (Int.unsigned y)).
```
