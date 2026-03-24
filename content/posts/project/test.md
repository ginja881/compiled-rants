---
title: "Toy Interpreter"
author: ""
tags: ["project"]
description: ""
showFullContent: false
readingTime: false
hideComments: true
color: "" #color from the theme settings
draft: false
---

As of 3/22/26, I have finished my first programming language by writing an interpreter in C. 
This is the first of many projects that will shoot my skills straight into the motha fuckin' stratosphere!!
More importantly, after not programming in C for almost 2-3 years, this project serves to start using the language again and 
as a gateway into compilation.

The github repository for this project is located at https://github.com/ginja881/toy_interpreter.

## Side Note

For those that do not come from a semantics background, the grammars of any programming language
can be thought of as abstract guidelines that state how we can work with a language. 

In the case of the toy language for my interpreter, the author of my book 


## Implementation
Written in at most 1k lines of C code, the interpreter is implemented in three stages:

1. Lexical Analysis,
2. Parsing,
3. Evaluation. 

### Lexical Analysis
By definition, lexical analysis is the process of processing text and "tokenizing" them into objects that we refer to as lexical tokens. 
Most modern lexical analyzers employ regular expressions (commonly referred as regex, more here https://en.wikipedia.org/wiki/Regular_expression) that are supposed to represent the many cases of text that would be classified as a token and utilize them on a finite state machine (formally known as deterministic finite automata).

In classical lexical analysis, we analyze text by considering two rules:
**Longest matching**: considering a substring of text, lexical analysis returns the token that is the longest match to the substring.
**Rule priority**: typically, we return the first regular expression that the substring of input matches.
As we analyze the text and produce tokens, we create a stream of tokens that will be passed down to the parsing phase.

I implemented lexical analysis through following the two rules and by implementing a simple queue focused around these tokens:

```
enum Token {
     L_PAREN,
     R_PAREN,
     ID,
     ASSIGN,
     PRINT,
     NUM,
     MUL, 
     PLUS, 
     SUB,
     DIV,
     COMMA,
     SEMI_COLON,
     END_OF_FILE
};
```

This specific phase within my interpreter was a great exercise on text processing and
start to developing the necessary predictive meaning needed for front-end compiler algorithms. 


### Parsing

(Note: My exprience in parsing theory is limited, so my explanation of this phase may not be the best.)

Typically, after lexical analysis has produced an ideal stream of tokenized text, we pass tokens from the stream into a module
known as the "parser" to understand more about the structure of the program and produce a famous tree representation known as an 
"Abstract Syntax Tree."

In the case of my interpreter, the parsing phase was implemented through using a famous and elegant solution known as "recursive descent."

Considering the grammar of the programming language, recursive descent is a decent solution because we can understand
structures of programs by implementing functions corresponding to general rules of the grammar and enabling recursion when needed.
For example, in my toy language, I simply used the recursive approach to effectively implement precedence for expressions:

```
int match(Token token_type, Lexer lexer) {
      /*
       * @brief matching for parser
       * @param token_type type of token
       * @param lexer lexical analyzer object
       */
      if (peek(lexer) == NULL) return FALSE;
      return (peek(lexer)->token == token_type ? TRUE : FALSE);
}
A_Exp parse_literal(Lexer lexer) {
       A_Exp exp = NULL;
       
       RawToken current_token = peek(lexer);
       if (match(ID, lexer) == TRUE) {
           exp = id_exp(current_token->text);
	   eat_token(lexer);
      }
       else if (match(NUM, lexer) == TRUE) {
           exp = num_exp(atoi(current_token->text));
	   eat_token(lexer);
       }
       else if (match(L_PAREN, lexer) == TRUE) {
            eat_token(lexer);
	    exp = parse_expression(lexer);
	    if (match(R_PAREN, lexer) == FALSE) {
	       current_token = peek(lexer);
	       error(SYNTAX_ERROR, current_token->pos, current_token->text, current_token->line_pos);
	    }
	    eat_token(lexer);

       }
       else
           error(SYNTAX_ERROR, current_token->pos, current_token->text, current_token->line_pos);
       return exp;

}

A_Exp parse_factor(Lexer lexer) {
      A_Exp left = parse_literal(lexer);
      while (match(PLUS, lexer) == TRUE || match(SUB, lexer) == TRUE) {
           RawToken current_token = eat_token(lexer);
           BinOp operator = get_op(current_token->token);
	   A_Exp right = parse_expression(lexer);
           left = op_exp(left, operator, right);
      }
      return left;
}

A_Exp parse_term(Lexer lexer) {
     A_Exp left = parse_factor(lexer);
     while (match(MUL, lexer) == TRUE || match(DIV, lexer) == TRUE) {
         RawToken current_token = eat_token(lexer);
	 BinOp operator = get_op(current_token->token);
	 A_Exp right = parse_expression(lexer);
	 left = op_exp(left, operator, right);
     }
     return left;
}

```



### Evaluation


Even though typical interpreters perform an additional phase known as "semantic analysis" to where we perform type checks, scope resolutions, and similar tasks on the AST generated from parsing, the toy language's grammar that I was working with does not even come with types or any other features warranting semantic analysis. 

As a result, I skipped to immediately processing/walking the AST by evaluating and interpreting the contents of the tree.

This phase was a bit interesting to debug by dealing with special cases of expressions: ESEQs. For context,
ESEQs are compact expressions in parentheses that have a statement and expression separated by a comma.
Since my evaluator was recursive, I had issues with passing up-to-date versions of data structures between calls, which yielded
errors with getting values for variables. Luckily, this bug was resolved by enforcing more control on my evaluator through switch statements due to their strict machine implementation.


### Overall

This project was very exciting (and sometimes annoying) to produce!!
As a result of my effort, I hope to break more into compilers by truly understanding
how higher level languages are translated into an understandable format for computers.