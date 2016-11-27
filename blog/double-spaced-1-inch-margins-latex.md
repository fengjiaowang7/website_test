% Double-spacing and 1-inch Margins in LaTeX
% 2010-04-10

Once or twice a year, I inevitably have to write a paper for a professor who wants 1-inch margins and double-spacing. Being a LaTeX user, I generally don’t concern myself with document formatting — I’ve always thought that’s the point of LaTeX: let it worry about formatting for you. As a result, I use formatting commands extraordinarily infrequently, and never actually learn them. So instead of playing around with various commands every few months until my memory gets it right, I’m going to put the appropriate snippet of code here. I know where to find it next time.

The code goes in the preamble. The first line sets the margins to 1-inch. The next two lines together make the document double-spaced.

```latex
\usepackage{fullpage}
\usepackage{setspace}
\doublespacing
```

As with almost anything in LaTeX, there are tons of ways of accomplishing the same thing. This is just the simplest and clearest method I’m aware of.
