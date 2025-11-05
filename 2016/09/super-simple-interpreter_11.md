# Super Simple Interpreter

Sunday, September 11, 2016

---

I was wondering how to write a simple interpreter in assembly for a 6510 processor ([#c64](https://twitter.com/hashtag/C64?src=hash)) and I dawned upon probably the simplest interpreter ever (aside from a [brainfuck](https://en.wikipedia.org/wiki/Brainfuck) interpreter).

Here is the basic pseudo code of the program:
```
define array of lines
define dictionary of variables
define left variable
define right variable
define counter and set to 1

loop
    write '> '
    get line
    append line to array of lines if line not empty
    break loop if line is 'run'
end loop

loop for each line
    split line by spaces

    if first element is 'print' or 'write' then
        if second element is single quote do
            get second to last element
            join with spaces
            write string excluding first and last character
        end
        else do
            find second element from dictionary of variables
            get value associated with element
            turn value to string
            write value string
        end
        if first element is 'print' do
            write '\n'
        end
    end

    else if second element is '=' do
        if third element is number do
            set left to number
        end
        else do
            get variable name
            find value from variable dictionary
            set left to value
        end
        if line is > 3 elements do
            if fifth element is number do
                set right to number
            end
            else do
                get variable name
                find value from variable dictionary
                set right to value
            end
            if fourth element is '+' do
                left = left + right
            end
            if fourth element is '-' do
                left = left - right
            end
            if fourth element is '*' do
                left = left * right
            end
            if fourth element is '/' do
                left = left / right
            end
        end
        set dictionary position of first element to value of left
    end

    else do
        write 'Error on line ' + counter
        break loop
    end

    counter = counter + 1
end loop
```

The syntax of this interpreted language is this: \
Each operation is its own line. \
The mathematical operations are limited to +, -, *, / \
All numbers are integers and operations return integers. \
Strings are only allowed in print/write statements. \
Strings are enveloped in single quotes. \
Variable names must be single letters. \
All individual parts of a line must be separated by 1 space. \
Write writes to screen without newline. \
Print is like write but with a newline. \
No direct string concatenation, use write statements for that.

Example program:
```
a = 5
write 'A is: '
print a
b = a + 5
b = b / 7
write 'B is: '
print b
```

Example output:
```
A is: 5
B is: 1
```

To test its simple nature, I wrote the original version of the interpreter in Python 2.7 and was able to make it in 26 lines of code. Here is the source:
```python
import sys
varis = {}
lines = []
write = sys.stdout.write
while 1: #get input
    inp = raw_input("> ")
    if inp == "run": break
    lines.append(inp)
for y in range(len(lines)): #parse program
    x = lines[y].split()
    if x[0] == "print" or x[0] == "write":
        if x[1][0] == "'": write(" ".join(x[1:])[1:-1])
        else: write(str(varis[x[1]]))
        if x[0] == "print": write("\n")
    elif len(x) > 2 and x[1] == "=":
        if ord(x[2][0]) > 96 and ord(x[2][0]) < 123: left = varis[x[2]]
        else: left = int(x[2])
        if len(x) > 3:
            if ord(x[4][0]) > 96 and ord(x[4][0]) < 123: right = varis[x[4]]
            else: right = int(x[4])
            if x[3] == "+": left += right
            elif x[3] == "-": left -= right
            elif x[3] == "*": left *= right
            elif x[3] == "/": left /= right
        varis[x[0]] = left
    else: print "Error: line "+str(y+1)
```

I then wrote a modified version of it in [Axon](https://www.beyon-d.net/doc/docSkySpark/AxonLang.html) (the main programming language I use at work) and I was able to make it in 31 lines of code. Here is the source:
```
(x) => do
  vars: {}
  x = x.replace("\n",";").split(";")
  t: []
  cou: 1
  left: ""
  right: ""
  x.each y=> do
    t = y.split(" ")
    if(t[0] == "print") do
      if(t[1][0..0] == "'") echo(t[1..-1].concat(" ")[1..-2])
      else echo(vars.trap(t[1]))
    end
    else if(t[1] == "=") do
      if(t[2][0] > 47 and t[2][0] < 58) left = parseInt(t[2])
      else left = vars.trap(t[2])
      if(t.size > 3) do
        if(t[4][0] > 47 and t[4][0] < 58) right = parseInt(t[4])
        else right = vars.trap(t[4])
        if(t[3] == "+") left = left + right
        else if(t[3] == "-") left = left - right
        else if(t[3] == "*") left = left * right
        else if(t[3] == "/") left = floor(left / right)
      end
      vars = vars.set(t[0], left)
    end
    else echo("Error: line "+cou)
    cou = cou + 1
  end
end
```

The differences in each of the implementations is that the Python one is more like a command line in which you type in your lines and then type in "run" when you want to execute the program, and it supports both "write" and "print". The Axon implementation only has "print" and is a function that takes in the whole program as a string parameter, so I made that to also work with semicolons between the statements.

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2016/09/11 at 2:53 PM PDT</sub>
