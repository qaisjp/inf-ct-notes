# _Semantic Analysis: Types_
_2018-10-09_ - Lecture 9

## What are types used for?

Checks that identifiers are declared and used correctly is not the only thing that needs to be verified in the compiler.

In most languages, _expressions have a type_. Expression types need to be checked. Statements don't have types tho.

For example:

- operands of `+` need to be ints
- operands of `==` must be compatible (`int` with `int`, `char` with `char`)
- number of args passed to functions need to be equal to the number of parameters

## Strongly typed != statically typed

|             | strong | weak       |
| ----------- | ------ | ---------- |
| **static**  | Java   | C/C++      |
| **dynamic** | Python | JavaScript |

- Static: checks before execution. Dynamic otherwise.
- Strong: type error causes error

## visitFunCall

```java
public type visitFunCall(FunCall fc) {
  fc.fd; // fc.fd is the function declaration
  List<Expression> fc.args; // fc.args is the arguments lf the call
  List<Vardecl> fc.fd.params; // fc.fd.somefieldname, allowed parameters
  
  if (args.length != args.length) {
    error();
    return null; // you might want to return an error type instead. save on NullPointerExceptions ;)
    
    // In theory you could make it return the ReturnType, allowing the compiler to compile the rest of the program
  }
  
  for (i from 0 to args.length) {
    Expression arg = args[i];
    VarDecl vd = params[i];
    
    Type argT = arg.accept(this);
    Type vdT = vd.type;
    
    // We don't do implicit typecasting so we directly compare
    // But if we want char->int, then we could automatically cast here
    if (!argT.equal(vdT)) {
      error();
      // return null; // don't return null yet, keep loopin'
    }
  }
  
  // Might wanna do a "yo errors return null" thing here
  // Not doing it (i.e. recovering) so we can report multiple errors
  
  // Set the funcall type
  fc.type = fc.fd.type;
  return fc.type;
}

// Example
int foo(int i, float f);
foo(2, c);
```
