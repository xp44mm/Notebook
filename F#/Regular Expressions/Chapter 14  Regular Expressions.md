# Chapter 14:  Regular Expressions

## Overview

Regular expressions are a standard way to search for and  optionally replace occurrences of substrings and text patterns. If you aren't  familiar with regular expressions, just think of the wildcard characters you use  at the command prompt to indicate a group of files (as in \*.txt) or the special  characters you can use with the Like operator in Microsoft Visual Basic (see [Chapter 2](BBL0015.html#67), "Basic  Language Concepts") or in SQL queries:

```SQL
SELECT name, city FROM customers WHERE name LIKE "A%"
```

Many computer scientists have thoroughly researched regular  expressions, and a few programming languages—most notably Perl and Awk—are  heavily based on regular expressions. In spite of their usefulness in virtually  every text-oriented task (including parsing log files and extracting information  from HTML files), regular expressions are relatively rarely used by Microsoft  Windows programmers, probably because they are based on a rather obscure  syntax.

You can regard regular expressions as a highly specific  programming language, and you know that all languages take time to learn and  have idiosyncrasies. But when you see how much time regular expressions can save  you—and I am talking about both coding time and CPU time—you'll probably agree  that the effort you expend learning their contorted syntax is well worth it.

This isn't the first time I mention regular expressions in this  book. For example, I mentioned regular expressions in the section titled "[String Constants and  Functions](BBL0019.html#243)" in [Chapter 3](BBL0018.html#210), where I compare the Like operator to regular  expressions, and in [Chapter 4](BBL0021.html#289), where I introduce Microsoft Visual Studio 2005  searches based on regular expressions. However, Visual Studio uses a notation  that differs from the one I explain in this chapter, so you can't apply the  actual patterns you'll learn in this chapter to searches in the code editor,  even though all basic concepts are the same.

Note: To avoid long lines, code samples in this chapter assume  that the following Imports statements are used at the file or project level: 
```F#
open System.Globalization 
open System.IO 
open System.Text 
open System.Text.RegularExpressions 
```
