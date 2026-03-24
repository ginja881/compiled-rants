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
typedef struct RawToken_* RawToken;

struct TokenQueue_ {
    RawToken token_head;
    RawToken token_tail;
    size_t size;
};

struct Lexer_ {
     struct TokenQueue_* queue;
     size_t pos;
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
// Recursive Descent
BinOp get_op(Token operator) {
    if (operator == PLUS)
       return OP_ADD;
    else if (operator == SUB)
       return OP_SUB;
    else if (operator == MUL)
       return OP_MUL;
    return OP_DIV;
}

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

int check_next_stm(Lexer lexer) {
      RawToken current_token = peek(lexer);
      switch (current_token->token) {
          case ID:
          case NUM:
          case PRINT:
          case L_PAREN:
              return TRUE;
          default:
              break;
      }
      return FALSE;
}

A_Exp parse_expression(Lexer lexer) {

     if (match(L_PAREN, lexer) == TRUE) {
          eat_token(lexer);
          if (check_next_stm(lexer) == TRUE) {
               A_Stm eseq_stm = parse_statement(lexer);

               if (match(COMMA, lexer) == TRUE) {

                    eat_token(lexer);

                    A_Exp eseq_expression = parse_expression(lexer);
                    if (match(R_PAREN, lexer) == FALSE)
                       error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
                    eat_token(lexer);
                    return eseq_exp(eseq_stm, eseq_expression);
               }
               else {
                   if (match(R_PAREN, lexer) == FALSE)
                      error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
                   eat_token(lexer);
                   return eseq_exp(eseq_stm, num_exp(0));
               }
          }
          else {
              A_Exp expression = parse_expression(lexer);
              if (match(R_PAREN, lexer) == FALSE)
                  error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
              eat_token(lexer);
              return expression;
          }
     }
     return parse_term(lexer);

}
A_ExpList parse_expression_list(Lexer lexer) {


      A_ExpList head = NULL;
      A_ExpList current_node = head;
      while (is_queue_empty(lexer) != TRUE && match(R_PAREN, lexer) != TRUE) {
          if (head == NULL) {
              head = exp_list(parse_expression(lexer));
              current_node = head;
          }
          else {
             current_node->tail = exp_list(parse_expression(lexer));
             current_node = current_node->tail;
          }

          if (match(COMMA, lexer) == TRUE)
             eat_token(lexer);
          else
             break;
      }
      return head;
}
A_Stm parse_statement(Lexer lexer) {
     A_Stm main_stm = NULL;
     RawToken current_token = peek(lexer);

     if (current_token->token == PRINT) {
          eat_token(lexer);


          if (is_queue_empty(lexer) == TRUE)
             error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
          else if (peek(lexer)->token != L_PAREN)
              error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);


          eat_token(lexer);
          A_ExpList expression_list = parse_expression_list(lexer);
          if (match(R_PAREN, lexer) == TRUE) {
             eat_token(lexer);
             main_stm = print_stm(expression_list);
          }
          else
             error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
     }
     else if (current_token->token == ID) {
           string id = current_token->text;
           eat_token(lexer);

           if (is_queue_empty(lexer) == TRUE)
              error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);
           else if (peek(lexer)->token !=ASSIGN)
              error(SYNTAX_ERROR, peek(lexer)->pos, peek(lexer)->text, peek(lexer)->line_pos);

           eat_token(lexer);

           A_Exp main_exp = parse_expression(lexer);

           main_stm = assign_stm(id, main_exp);
     }
     else if (current_token->token == NUM || current_token->token == ID || current_token->token == L_PAREN) {
          A_Exp main_exp = parse_expression(lexer);
          main_stm = expression_stm(main_exp);
     }
     else
         error(SYNTAX_ERROR, current_token->pos, current_token->text, current_token->line_pos);

     return main_stm;
}

A_Stm parse_source_code(Lexer lexer) {
      /*
       * @brief main parsing algorithm
       * @param lexer Lexical analyzer object
       */

      // Root and current_stm for tracking
      A_Stm root = NULL;
      A_Stm current_stm = NULL;

      // Sentinel for consuming/matching tokens
      RawToken current_token = peek(lexer);
      while (is_queue_empty(lexer) == FALSE && current_token->token != END_OF_FILE) {
               // Make STM

               current_stm = parse_statement(lexer);
               // Check for compound statement
               current_token = peek(lexer);

               if (match(SEMI_COLON, lexer) == TRUE) {
                    eat_token(lexer);
                    // Root case
                    if (root == NULL)
                        root = current_stm;
                    else
                        root = compound_stm(root, current_stm);
               }
               else
                  error(SYNTAX_ERROR, current_token->pos, current_token->text, current_token->line_pos);
               // Update reference to stream front
               current_token = peek(lexer);

      }
      return root;
}

```



### Evaluation


Even though typical interpreters perform an additional phase known as "semantic analysis" to where we perform type checks, scope resolutions, and similar tasks on the AST generated from parsing, the toy language's grammar that I was working with does not even come with types or any other features warranting semantic analysis. 

As a result, I skipped to immediately processing/walking the AST by evaluating and interpreting the contents of the tree.

This phase was a bit interesting to debug by dealing with special cases of expressions: ESEQs. For context,
ESEQs are compact expressions in parentheses that have a statement and expression separated by a comma.
Since my evaluator was recursive, I had issues with passing up-to-date versions of data structures between calls, which yielded
errors with getting values for variables. Luckily, this bug was resolved by enforcing more control on my evaluator through switch statements due to their strict machine implementation.

The following snippet is my evaluator:
```
#include "eval.h"

ExpressionResult interpExp(A_Exp exp, HashTable symbol_table) {
       /*
        * @brief evaluator for expressions
        * @param exp A_Exp object
        */


        ExpressionResult result;
        result.table = symbol_table;



        if (exp->kind == Num_Exp)
           result.val = exp->u.num_exp.num;
        else if (exp->kind == ID_Exp)
            result.val = lookup(exp->u.id_exp.id, result.table);
        else if (exp->kind == Op_Exp) {
              A_Exp left = exp->u.op_exp.exp1;
              A_Exp right = exp->u.op_exp.exp2;

              ExpressionResult left_result = interpExp(left, result.table);
              ExpressionResult right_result = interpExp(right, left_result.table);
              result.table = right_result.table;
              symbol_table = result.table;
              BinOp operator = exp->u.op_exp.op;

              switch (operator) {
                  case OP_ADD:
                      result.val = left_result.val +  right_result.val;
                      break;
                  case OP_SUB:
                      result.val = left_result.val - right_result.val;
                      break;
                  case OP_MUL:
                      result.val = left_result.val * right_result.val;
                      break;
                  case OP_DIV:
                      result.val = (int)(left_result.val / right_result.val);
                      break;
              }
        }
        else if (exp->kind == Eseq_Exp) {

             A_Stm eseq_stm = exp->u.eseq_exp.stm;

             result.table = interpStatement(eseq_stm, result.table);

             ExpressionResult eseq_exp_result = interpExp(
                exp->u.eseq_exp.exp,
                result.table
             );


             result.table = eseq_exp_result.table;
             symbol_table = result.table;
             result.val = eseq_exp_result.val;

        }
        return result;
}

HashTable interpExpList(A_ExpList exp_list, HashTable symbol_table, int print) {
       /*
        * @brief evaluator for expresion lists.
        * @param exp_list A_ExpList object
        */
        A_ExpList current_exp_list_node = exp_list;
        while (current_exp_list_node != NULL) {
             A_Exp current_exp = current_exp_list_node->exp;
             ExpressionResult current_exp_result = interpExp(current_exp, symbol_table);
             symbol_table = current_exp_result.table;

             if (print == TRUE)
                 printf("\n %d \n", current_exp_result.val);
             current_exp_list_node = current_exp_list_node->tail;
        }

        return symbol_table;
}

HashTable interpStatement(A_Stm statement, HashTable symbol_table) {
       /*
        * @brief evaluator for statements
        * @param statement A_Stm object
        */

        if(statement->kind == ExpStm) {
             ExpressionResult result = interpExp(
                statement->u.expression_stm.exp,
                symbol_table
             );
             symbol_table = result.table;

        }
        else if (statement->kind == AssignStm) {
             string id = statement->u.assign_stm.id;
             ExpressionResult result = interpExp(
                  statement->u.assign_stm.exp,
                  symbol_table
             );

             symbol_table = update(id, result.val, result.table);

        }
        else if (statement->kind == PrintStm) {
             A_ExpList expression_list = statement->u.print_stm.exp_list;
             symbol_table = interpExpList(expression_list, symbol_table, TRUE);

        }

        return symbol_table;
}

HashTable interpProgram(A_Stm AST_Root, HashTable symbol_table) {
      /*
       * @brief main evaluator for program. runtime is sum of runtimes that each state takes.
       * @param AST_Root root of AST made by parser
       */


       if (AST_Root == NULL) return symbol_table;

       switch(AST_Root->kind) {
            case CompoundStm:
                symbol_table = interpProgram(AST_Root->u.compound_stm.stm1, symbol_table);
                return interpProgram(AST_Root->u.compound_stm.stm2, symbol_table);
            default:
                break;
       }
       return interpStatement(AST_Root, symbol_table);

}
```

### Overall

This project was very exciting (and sometimes annoying) to produce!!
As a result of my effort, I hope to break more into compilers by truly understanding
how higher level languages are translated into an understandable format for computers.