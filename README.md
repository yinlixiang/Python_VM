# Cpython 源码阅读

## Python 源码执行过程

<https://hackmd.io/s/ByMHBMjFe>

repo: <https://github.com/python/cpython/tree/29d018aa63b72161cfc67602dc3dbd386272da64>


```
Main [Programs/python.c] 
   => Py_Main [Modules/main.c] 
       => pymain_main
           => pymain_init
               => _PyRuntime_Initialize
               => _Py_InitializeFromWideArgs
                   => init_python
                       => _Py_InitializeMainInterpreter
           => _Py_RunMain
               => PyRun_AnyFileExFlags
                   => PyParser_ASTFromFileObject
                       => PyParser_ParseFileObject
                           => PyTokenizer_FromFile
                           => parsetok: for (;;) {PyTokenizer_Get}
                       => PyAST_FromNodeObject
                   => run_mod
                       => PyAST_CompileObject
                           => PySymtable_BuildObject: 
                               symtable_visit_stmt(st,stmt_ty) for stmt_ty in asdl_seq
                           => compiler_mod
                               => compiler_enter_scope
                               => compiler_body: 
                                   VISIT(c, stmt, stmt_ty) for stmt_ty in asdl_seq
                               => compiler_exit_scope
				               => assemble
                       => run_eval_code_obj
                           => PyEval_EvalCode
                               => PyEval_EvalCodeEx
                                   => _PyEval_EvalCodeWithName
                                       => _PyFrame_New_NoTrack
                                       => PyEval_EvalFrameEx
                                           => eval_frame 
                                               => _PyEval_EvalFrameDefault: 
                                                   main_loop
```

## Design of CPython’s Compiler

<https://cpython-devguide.readthedocs.io/compiler>


Compiler process:
1. Parse source code into a parse tree (`Parser/parsetok.c`) 
2. Transform parse tree into an Abstract Syntax Tree (`Python/ast.c`) 
3. Transform AST into a Control Flow Graph (`Python/compile.c`) 
4. Emit bytecode based on the Control Flow Graph (`Python/compile.c`) 

Excution:
5. Executes byte code (`Python/ceval.c`)

### Parse Trees 

an LL(1) parser: *Compilers: Principles, Techniques, and Tools* 

Python grammar: `Grammar/Grammar` `Include/graminit.h`

Python tokens: `Grammar/Tokens` `Include/token.h`

The parse tree: `Include/node.h`

- `CHILD(node *, int)`
- `RCHILD(node *, int)`
- `NCH(node *)`: Number of children
- `STR(node *)`
- `TYPE(node *)`
- `REQ(node *, TYPE)`
- `LINENO(node *)`



`Parser/parsetok.c`

- `parsetok`



### Abstract Syntax Trees (AST)

[The Zephyr Abstract Syntax Description Language - Princeton CS](https://www.cs.princeton.edu/~appel/papers/asdl97.pdf)

Python AST nodes: `Parser/Python.asdl` `Parser/asdl.py`

`Python/asdl.c` `Include/asdl.h`

`Python/Python-ast.c` `Include/Python-ast.h`

`xxx_ty`: AST node

`asdl_seq *`: a sequence of AST nodes

- `_Py_asdl_seq_new(Py_ssize_t, PyArena *)`
- `asdl_seq_GET(asdl_seq *, int)`
- `asdl_seq_SET(asdl_seq *, int, stmt_ty)`
- `asdl_seq_LEN(asdl_seq *)`



### Memory Management

an arena: a memory is pooled in a single location for easy allocation and removal.

`Include/pyarena.h` `Python/pyarena.c`

`PyArena` structure

- `PyArena_New()`
- `PyArena_Free()`
- `PyArena_AddPyObject()`



### Parse Tree to AST

`Python/ast.c`

- `PyAST_FromNode()` 
  - `PyAST_FromNodeObject()`
    - `ast_for_xxx` => `xxx_ty`



### Control Flow Graphs (CFG)

a directed graph: models the flow of a program using basic blocks

Python bytecode: intermediate representation (IR)

Basic blocks: a block of IR

- single entry point
- possibly multiple exit points

Code is directly generated from the basic blocks (with jump targets adjusted based on the output order) by doing a post-order depth-first search on the CFG following the edges.



### AST to CFG to Bytecode

1. transforms the AST into Python bytecode with control flow represented by the edges of the CFG.
2. creates the namespace: variables can be classified as local, free/cell for closures, or global
3. flattens the CFG into a list and calculates jump offsets: a post-order depth-first search



`Python/compile.c`

- `PyAST_CompileObject()`
  - `PySymtable_BuildObject()`: `Python/symtable.c`
    - `symtable_visit_xxx` => symbol table
  - `compiler_mod()`
    - `compiler_body(struct compiler *c, asdl_seq *stmts)`
      - `VISIT(c, stmt, stmt_ty) for stmt_ty in stmts`
    - `assemble(compiler c)` => `PyCodeObject *co`
      - `dfs(c, entryblock, &a, nblocks)`
      - `assemble_jump_offsets(&a, c)`
      - Emit code in reverse postorder from dfs: `assemble_emit`
      - `co = makecode(c, &a)`



### Code Objects

`Include/code.h`
    `PyCodeObject`

`Python/ceval.c`
- `_PyEval_EvalFrameDefault()`
