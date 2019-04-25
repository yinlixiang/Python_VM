# Python Grammar


<https://cpython-devguide.readthedocs.io/grammar/#>

[Modifying the Python language in 6 minutes](<https://hackernoon.com/modifying-the-python-language-in-7-minutes-b94b0a99ce14>)



CPython works in four steps:

- lexing
- parsing
- compiling
- interpreting

## Python Parser Generator

`Python/pgen`: Python’s parser generator

- `make regen-grammar`
- `make regen-token`
-  `make regen-ast`

### Token

Definition: `Grammar/Tokens`

Run `make regen-token`, generates:

- `Include/token.h`
- `Parser/token.c`
- `Lib/token.py`
- `Doc/library/token-list.inc`



Lexer:

- `Parser/tokenizer.c`: The CPython lexer
- `Lib/tokenize.py`: a lexical scanner for Python source code



### Grammar

Definition: `Grammar/Grammar`

Run `make regen-grammar`, generates:

- `Include/graminit.h`
- `Python/graminit.c`



`Modules/parsermodule.c`: give a parse tree produced by Python code as a tuple to the compiler



### Parse Trees

`Include/node.h`

`Parser/node.c`



`Parser/parser.c`



`Parser/parsetok.c`: Parser-tokenizer link implementation





### AST

Definition: `Parser/Python.asdl`

Run `make regen-ast`, generates:

- `Include/Python-ast.h`
- `Python/Python-ast.c`



`Python/ast.c`: transform a concrete syntax tree (CST) to an abstract syntax tree (AST)

`Lib/ast.py`



## UML

注: VS Code中安装下PlantUML可查看生成的UML图。

```plantuml
@startuml
class pythonrun {
    Python/pythonrun.c
    ==
    + PyRun_FileExFlags()
    + PyParser_ASTFromFileObject(FILE)
    - run_mod()
    - run_eval_code_obj()
    ==
    PyArena_New()
    PyParser_ParseFileObject(FILE): node
    parsetok.PyAST_FromNodeObject(node): mod_ty
    PyAST_CompileObject()
    PyEval_EvalCode()
    PyArena_Free()
}

pythonrun --> pyarena
pythonrun --> parsetok

class pyarena {
    Python/pyarena.c
    ==
    + PyArena
    + PyArena_New()
    + PyArena_Free()
}

class parsetok {
    Include/parsetok.h
    Parser/parsetok.c
    ==
    + PyParser_ParseFileObject(FILE): node
    - parsetok(tokenizer, grammar): node
    ==
    tokenizer.PyTokenizer_FromFile(): tokenizer
    while tok = tokenizer.PyTokenizer_Get():
        PyParser_AddToken(tok): node
}

parsetok --> tokenizer
parsetok --> parser

class parser {
    Parser/parser.h
    Parser/parser.c
    ==
    - grammar *p_grammar
    - node *p_tree: Top of parse tree
    - stack p_stack: Stack of parser states
    + PyParser_New(grammar, node): node not in
    + PyParser_AddToken(token)
    + PyGrammar_AddAccelerators(grammar)
    + PyParser_Delete()
}

parser --> grammar
parser --> node

class tokenizer {
    Include/tokenizer.h
    Parser/tokenizer.c
    ==
    - FILE *fp
    - char *cur
    + PyTokenizer_FromFile()
    + PyTokenizer_Get()
    + PyTokenizer_Free()
}

tokenizer --> token

class token {
    Grammar/Tokens
    Include/token.h
    Parser/token.c
    ==
    + ISTERMINAL()
    + ISNONTERMINAL()
    + ISEOF()
    + PyToken_OneChar()
    + PyToken_TwoChars()
    + PyToken_ThreeChars()
}

class node {
    Include/node.h
    Parser/node.c
    ==
    + PyNode_New()
    + PyNode_AddChild()
    + PyNode_Free()
    + NCH(node)
    + CHILD(node, int)
    + RCHILD(node, int)
    + TYPE(node)
    + STR()
    + LINENO()
    + REQ()
}

class grammar {
    Grammar/Grammar
    Include/graminit.h
    Python/graminit.c
    Include/grammer.h
    Parser/grammer1.c
    ==
    - dfa   *g_dfa
    - labellist g_ll
    + PyGrammar_FindDFA(grammar, int)
    + PyGrammar_LabelRepr(label)   
}

grammar o-- dfa
grammar o-- label

class dfa {

}

class label {

}
@enduml
```