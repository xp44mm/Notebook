# Expression Simplification and Differentiation

新建Visual F# 类库，命名：AlgebraicExpressions

```
install-package fsharp.core
install-package fslexyacc
```

添加`Expr.fs`文件：

```F#
namespace Symbolic.Expressions

type Expr =
    | Num of decimal
    | Var of string
    | Neg of Expr
    | Add of Expr list
    | Sub of Expr * Expr list
    | Prod of Expr * Expr
    | Frac of Expr * Expr
    | Pow of Expr * decimal
    | Sin of Expr
    | Cos of Expr
    | Exp of Expr

    static member StarNeeded e1 e2 =
        match e1, e2 with
        | Num _, Neg _ | _, Num _ -> true
        | _ -> false

    member self.IsNumber =
        match self with
        | Num _ -> true | _ -> false

    member self.NumOf =
        match self with
        | Num num -> num | _ -> failwith "NumOf: Not a Num"

    member self.IsNegative =
        match self with
        | Num num | Prod (Num num, _) -> num < 0M
        | Neg e -> true | _ -> false

    member self.Negate =
        match self with
        | Num num -> Num (-num)
        | Neg e -> e
        | exp -> Neg exp


```

添加`ExprParser.fsy`文件：

```perl
%{
open System
open Symbolic.Expressions
%}

%token <int> INT
%token <float> FLOAT
%token <string> ID
%token EOF LPAREN RPAREN PLUS MINUS TIMES DIV HAT SIN COS E
%left ID
%left prec_negate
%left LPAREN
%left PLUS MINUS
%left TIMES DIV
%left HAT
%start expr
%type <Expr> expr
%%

expr:
	| exp EOF { $1 }

number:
	| INT { Convert.ToDecimal($1) }
	| FLOAT { Convert.ToDecimal($1) }
	| MINUS INT %prec prec_negate { Convert.ToDecimal(-$2) }
	| MINUS FLOAT %prec prec_negate { Convert.ToDecimal(-$2) }

exp:
	| number { Num $1 }
	| ID { Var $1 }
	| exp PLUS exp { Add [$1; $3] }
	| exp MINUS exp { Sub ($1, [$3]) }
	| exp TIMES exp { Prod ($1, $3) }
	| exp DIV exp { Frac ($1, $3) }
	| SIN LPAREN exp RPAREN { Sin $3 }
	| COS LPAREN exp RPAREN { Cos $3 }
	| E HAT exp { Exp $3 }
	| term { $1 }
	| exp HAT number { Pow ($1, $3) }
	| LPAREN exp RPAREN { $2 }
	| MINUS LPAREN exp RPAREN { Neg $3 }

term:
	| number ID { Prod (Num $1, Var $2) }
	| number ID HAT number { Prod (Num $1, Pow (Var $2, $4)) }
	| ID HAT number { Prod (Num 1M, Pow (Var $1, $3)) }

```

添加`ExprLexer.fsl`文件：

```perl
{
module Symbolic.Expressions.ExprLexer
open System
open Symbolic.Expressions
open Symbolic.Expressions.ExprParser
open Microsoft.FSharp.Text.Lexing
let lexeme = LexBuffer<_>.LexemeString
let special s =
    match s with
    | "+" -> PLUS | "-" -> MINUS
    | "*" -> TIMES | "/" -> DIV
    | "(" -> LPAREN | ")" -> RPAREN | "^" -> HAT
    | _ -> failwith "Invalid operator"
let id s =
    match s with
    | "sin" -> SIN | "cos" -> COS
    | "e" -> E | id -> ID id
}

let digit   = ['0'-'9']
let int     = digit+
let float   = int ('.' int)? (['e' 'E'] int)?
let alpha   = ['a'-'z' 'A'-'Z']
let id      = alpha+ (alpha | digit | ['_' '$'])*
let ws      = ' ' | '\t'
let nl      = '\n' | '\r' '\n'
let special = '+' | '-' | '*' | '/' | '(' | ')' | '^'

rule main = parse
 | int     { INT (Convert.ToInt32(lexeme lexbuf)) }
 | float   { FLOAT (Convert.ToDouble(lexeme lexbuf)) }
 | id      { id (lexeme lexbuf) }
 | special { special (lexeme lexbuf) }
 | ws | nl { main lexbuf }
 | eof     { EOF }
 | _       { failwith (lexeme lexbuf) }

```

关闭VS，找到项目文件`AlgebraicExpressions.fsproj`，用记事本打开，我们这里需要修改项目包含的文件列表，位于`<ItemGroup>`中:

```xml
  <ItemGroup>
    <Compile Include="AssemblyInfo.fs" />
    <Content Include="packages.config" />
    <Compile Include="Expr.fs" />
    <None Include="ExprParser.fsy" />
    <None Include="ExprLexer.fsl" />
  </ItemGroup>

```

修改为：

```xml
  <ItemGroup>
    <Compile Include="AssemblyInfo.fs" />
    <Content Include="packages.config" />
    <Compile Include="Expr.fs" />
    <FsYacc Include="ExprParser.fsy">
      <OtherFlags>--module Symbolic.Expressions.ExprParser</OtherFlags>
    </FsYacc>
    <FsLex Include="ExprLexer.fsl">
      <OtherFlags>--unicode</OtherFlags>
    </FsLex>
    <Compile Include="ExprParser.fs" />
    <Compile Include="ExprLexer.fs" />
  </ItemGroup>

```

重新用VS打开项目，会发现解决方案资源管理器中，多出两个文件`ExprParser.fs`、`ExprLexer.fs`。

继续添加文件，以完成功能。

添加`ExprUtils.fs`文件：

```F#
module Symbolic.Expressions.Utils

open Symbolic.Expressions

/// A helper function to map/select across a list while threading state
/// through the computation
let collectFold f l s =
    let l,s2 = (s, l) ||> List.mapFold (fun z x -> f x z)
    List.concat l,s2

/// Collect constants
let rec collect e =
    match e with
    | Prod (e1, e2) ->
        match collect e1, collect e2 with
        | Num n1, Num n2 -> Num (n1 * n2)
        | Num n1, Prod (Num n2, e)
        | Prod (Num n2, e), Num n1 -> Prod (Num (n1 * n2), e)
        | Num n, e | e, Num n -> Prod (Num n, e)
        | Prod (Num n1, e1), Prod (Num n2, e2) -> Prod (Num (n1 * n2), Prod (e1, e2))
        | e1', e2' -> Prod (e1', e2')
    | Num _ | Var _ as e -> e
    | Neg e -> Neg (collect e)
    | Add exprs -> Add (List.map collect exprs)
    | Sub (e1, exprs) -> Sub (collect e1, List.map collect exprs)
    | Frac (e1, e2) -> Frac (collect e1, collect e2)
    | Pow (e1, n) -> Pow (collect e1, n)
    | Sin e -> Sin (collect e)
    | Cos e -> Cos (collect e)
    | Exp _ as e -> e

/// Push negations through an expression
let rec negate e =
    match e with
    | Num n -> Num (-n)
    | Var v as exp -> Neg exp
    | Neg e -> e
    | Add exprs -> Add (List.map negate exprs)
    | Sub _ -> failwith "unexpected Sub"
    | Prod (e1, e2) -> Prod (negate e1, e2)
    | Frac (e1, e2) -> Frac (negate e1, e2)
    | exp -> Neg exp

let filterNums (e:Expr) n =
    if e.IsNumber
    then [], n + e.NumOf
    else [e], n

let summands e =
    match e with
    | Add es -> es
    | e -> [e]

/// Simplify an expression
let rec simp e =
    match e with
    | Num n -> Num n
    | Var v -> Var v
    | Neg e -> negate (simp e)
    | Add exprs ->
        let exprs2, n =
            (exprs, 0M) ||> collectFold (simp >> summands >> collectFold filterNums)
        match exprs2 with
        | [] -> Num n
        | [e] when n = 0M -> e
        | _ when n = 0M -> Add exprs2
        | _ -> Add (exprs2 @ [Num n])
    | Sub (e1, exprs) -> simp (Add (e1 :: List.map Neg exprs))
    | Prod (e1, e2) ->
        match simp e1, simp e2 with
        | Num 0M, _ | _, Num 0M -> Num 0M
        | Num 1M, e | e, Num 1M -> e
        | Num n1, Num n2 -> Num (n1 * n2)
        | e1, e2 -> Prod (e1, e2)
    | Frac (e1, e2) ->
        match simp e1, simp e2 with
        | Num 0M, _ -> Num 0M
        | e1, Num 1M -> e1
        | Num n, Frac (Num n2, e) -> Prod (Frac (Num n, Num n2), e)
        | Num n, Frac (e, Num n2) -> Frac (Prod (Num n, Num n2), e)
        | e1, e2 -> Frac (e1, e2)
    | Pow (e, 1M) -> simp e
    | Pow (e, n) -> Pow (simp e, n)
    | Sin e -> Sin (simp e)
    | Cos e -> Cos (simp e)
    | Exp e -> Exp (simp e)


let simplify e = e |> simp |> simp |> collect

let rec diff v e =
    match e with
    | Num _ -> Num 0M
    | Var v2 when v2=v -> Num 1M
    | Var _ -> Num 0M
    | Neg e -> diff v (Prod ((Num -1M), e))
    | Add exprs -> Add (List.map (diff v) exprs)
    | Sub (e1, exprs) -> Sub (diff v e1, List.map (diff v) exprs)
    | Prod (e1, e2) -> Add [Prod (diff v e1, e2); Prod (e1, diff v e2)]
    | Frac (e1, e2) -> Frac (Sub (Prod (diff v e1, e2), [Prod (e1, diff v e2)]),Pow (e2, 2M))
    | Pow (e1, n) -> Prod (Prod (Num n, Pow (e1, n - 1M)), diff v e1)
    | Sin e -> Prod (Cos e, diff v e)
    | Cos e -> Neg (Prod (Sin e, diff v e))
    | Exp (Var v2) as e when v2=v -> e
    | Exp (Var v2) -> Num 0M
    | Exp e -> Prod (Exp e, diff v e) 


```

添加测试所需的包：

```
install-Package xunit
install-Package xunit.runner.visualstudio
```

添加`Tester.fs`测试文件：

```F#
namespace Symbolic.Expressions

open Xunit
open Xunit.Abstractions
open Microsoft.FSharp.Text.Lexing

type Tester(output : ITestOutputHelper) =
    let parse text =
        let lex = LexBuffer<char>.FromString text
        ExprParser.expr ExprLexer.main lex

    let ProcessOneLine text =
        let e1 = parse text

        output.WriteLine <| sprintf "After parsing: %A" e1

        let e2 = Utils.simplify e1
        output.WriteLine <| sprintf "After simplifying: %A" e2

        let e3 = Utils.diff "x" e2
        output.WriteLine <| sprintf "After differentiating: %A" e3

        let e4 = Utils.simplify e3
        output.WriteLine <| sprintf "After simplifying: %A" e4


    [<Fact>]
    member this.exec() =
        ProcessOneLine "x+0"
```

如果需要，添加`App.config`文件：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="FSharp.Core" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-4.4.1.0" newVersion="4.4.1.0" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>

```

