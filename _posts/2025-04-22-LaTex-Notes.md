---
layout: post
title: LaTex Notes
---

# REF

https://www.latex-project.org/

https://www.tug.org/interest.html#free

https://miktex.org/

https://strawberryperl.com/

# ENV

1. 安装 latex，比如 miktex for windows

2. 安装 perl，比如 strawberry perl

3. VS Code 插件 latex workshop

4. 文件后缀 .tex, build 后查看 pdf 等输出

# Grammar

https://www.learnlatex.org/zh-hans/

https://www.colorado.edu/aps/sites/default/files/attached-files/latex_primer.pdf


## Basic Example

```latex
\documentclass{article}

\usepackage{amsmath} % for math features

\title{Sample LaTeX Document}
\author{Your Name}
\date{\today}

\begin{document}

\maketitle

\section{Introduction}

This is a simple LaTeX document compiled in VS Code.

\section{Mathematics}

An inline equation: \( E = mc^2 \).

A displayed equation:

\[
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
\]

\end{document}
```

The `\documentclass{}` command sets the overall style (article, book, report, etc.).

Content goes between `\begin{document}` and `\end{document}`.

Commands start with a backslash `\` and often take arguments in curly braces `{}`.

Blank lines start new paragraphs; use `\\` for line breaks without new paragraphs.

## Text Format

**Bold**: `\textbf{...}`

_Italics_: `\textit{...}`

Underline: `\underline{...}`

Emphasis (context-sensitive italics): `\emph{...}`

Comments start with % and continue to end of line

## Math

Inline math: 

`$ ... $` or `$$ ... $$` or `\begin{math} ... \end{math}`

Display math (centered equations): 

`$$ ... $$` or `\begin{equation} ... \end{equation}`

Use `^` for superscripts, `_` for subscripts, and commands like `\frac{a}{b}` for fractions

Greek letters and math operators have backslash commands like `\alpha`, `\sin`, `\int`

## Paragraphs and Line Breaks

New paragraphs by inserting a blank line in source

Use `\\` or \newline for line breaks without new paragraphs

LaTeX ignores extra spaces and line breaks inside paragraphs
