# Transformations
* CompCert C to Clight

## Clight
* `SimplExprspec.v`
* `Clight.v`
* `Ebinop` type
* Splits C into two types:
  1. Expressions - "Pure" C
  2. Statements - Assignments, conditionals, etc.
```
(** [eval_expr ge e m a v] defines the evaluation of expression [a]
  in r-value position.  [v] is the value of the expression.
  [e] is the current environment and [m] is the current memory state. *)
Inductive eval_expr: expr -> val -> Prop
| eval_Ebinop: forall op a1 a2 ty v1 v2 v,
      eval_expr a1 v1 ->
      eval_expr a2 v2 ->
      sem_binary_operation ge op v1 (typeof a1) v2 (typeof a2) m = Some v ->
      eval_expr (Ebinop op a1 a2 ty) v
```
* What is `ge`?
  * `ge` is a `genv`
  * Not actually used in evaluating semantics of shift
    but I can see why it would be needed in say doing an add.
* What is a `genv`?
  * Seems to be a record that holds two environments
    1. `genv_genv`
    2. `genv_cenv`
* What is `sem_binary_operation`

* Semantics of shift defined in `Cop.v`
  * "Arithmetic and logical operators for the Compcert C and Clight languages"

### Shift Semantics in Cop.v
* `Inductive classify_shift_cases : Type`
* `Definition classify_shift (ty1: type) (ty2: type)`
* `Definition sem_shift`
* `Definition sem_shl (v1:val) (t1:type) (v2: val) (t2:type) : option val`
* `Definition sem_shr (v1:val) (t1:type) (v2: val) (t2:type) : option val`

Combined semantics
```
Definition sem_binary_operation
    (cenv: composite_env)
    (op: binary_operation)
    (v1: val) (t1: type) (v2: val) (t2:type)
    (m: mem): option val :=
  match op with
  | Oadd => sem_add cenv v1 t1 v2 t2 m
  | Osub => sem_sub cenv v1 t1 v2 t2 m
  | Omul => sem_mul v1 t1 v2 t2 m
  | Omod => sem_mod v1 t1 v2 t2 m
  | Odiv => sem_div v1 t1 v2 t2 m
  | Oand => sem_and v1 t1 v2 t2 m
  | Oor  => sem_or v1 t1 v2 t2 m
  | Oxor  => sem_xor v1 t1 v2 t2 m
  | Oshl => sem_shl v1 t1 v2 t2               <<
  | Oshr  => sem_shr v1 t1 v2 t2              <<
  | Oeq => sem_cmp Ceq v1 t1 v2 t2 m
  | One => sem_cmp Cne v1 t1 v2 t2 m
  | Olt => sem_cmp Clt v1 t1 v2 t2 m
  | Ogt => sem_cmp Cgt v1 t1 v2 t2 m
  | Ole => sem_cmp Cle v1 t1 v2 t2 m
  | Oge => sem_cmp Cge v1 t1 v2 t2 m
  end.
```

#### Shift Left
* Int.shl and Int64.shl <<- `Integers.v`
  * Int and Int64 are modules that define integer semantics for wordsizes
    32 bit and 64 bit respecitvely
  * Seems to just be `Z.shiftl` and doing unsigned shift

#### Shift Right
```
(fun sg n1 n2 =>
  match sg with
    | Signed => Int.shr n1 n2
    (* Z Right Shift where n1 signed and n2 unsigned *)
    | Unsigned => Int.shru n1 n2
    (* Z Right Shift where n1 n2 are unsigned *)
end)

(fun sg n1 n2 =>
  match sg with
  | Signed => Int64.shr n1 n2
  | Unsigned => Int64.shru n1 n2 end)
```
* Uses Z Shifting functions as well

### So what is `sem_shift`?
* Based on the type of the operands, choose a particular shift operation
to do

#### `classify_shift`
* Matches `typeconv` of each type
* Matches on TInt and TLong (fails on everything else)
* Possible types are:
```
  | shift_case_ii(s: signedness)         (**r int , int *)
  | shift_case_ll(s: signedness)         (**r long, long *)
  | shift_case_il(s: signedness)         (**r int, long *)
  | shift_case_li(s: signedness)         (**r long, int *)
  | shift_default.
```
* Values need to be Vint or Vlong

#### `Vint`
* Constructor for `val` type
* Defined in `Values.v`

#### `typeconv`
* `CTypes.v`
* Promotes small integers to Signed int32
* Not relevent to project but good to know: Degrades array types and function
  types to pointer types

### Cases of `sem_shift`
1. Int and Int
  * Both values must be `Vint`
  * `Int.ltu n2 Int.iwordsize` must be "true"
    * That is, unsigned n2 is less than Int.iwordsize = 32 bits
    * Returns given semantics for integer shift
  * Any failures return none
  * If both values are Vint types, and unsigned n2 is less than 32 bits
    then do the left shift function or right shift function. In the case of
    left shift, it is just Z.shiftl unsigned. In the case of right shift it
    will be Z Right shift with n1 being signed or unsigned, and n2 is always signed
2. Int and Long
  * First value (v1) is an int and second value (v2) is a long
  * Int64.ltu: n2 is unsigned and less than Int64.repr 32
  * If so, do the appropriate int shift where we shift n1 by Int64.loword n2
  * **Returns int!**
3. Long and Int
  * If `Int.ltu n2 Int64.iwordsize`
  * Then `Some(Vlong(sem_long sg n1 (Int64.repr (Int.unsigned n2))))`
    * n2 converted to a unsigned Coq integer from an Int
    * Then converted into a machine representation of Int64 integer
    * Then shifting n1 by the resulting n2
  * Otherwise None
4. Long and Long
  * Essentially case 1 but Int64s as opposed to Ints
  * Check that n2 is less than Int64 word size (unsigned comparison)
  * If so, perform shift

#### What is `Int.ltu`?
  * Performs zlt on unsigned versions of given operands
#### What is `Int.iwordsize`?
  * Word size as a machine integer
#### What is Int.repr?
* Machine representation of a given Coq integer
#### What is Int.loword?
* Lower order 32 bits of a 64 bit integer
