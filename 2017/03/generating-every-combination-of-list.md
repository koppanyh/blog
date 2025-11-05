# Generating Every Combination of a List

Saturday, March 11, 2017

---

So today at work I had to modify a program to generate relative links to points in a database based on a customer's specifications. \
The chosen implementation required that I come up with every possible combination for a list containing unique elements to be only used if absolutely necessary. \
If the elements had to be used, then the specification called for using the minimum amount at the lowest level of our tree data structure as possible.

No problem, I was sure that generating all the combinations (not permutations) for a list in lexicographical order wouldn't be so hard.

I couldn't have been more wrong.

In popular languages, like Python, there's always some library that can help out with a task like this. Unfortunately, I didn't have the luxury of using a library because the language used at my work is called [Axon](https://www.beyon-d.net/doc/docSkySpark/AxonLang.html) and it only has whatever "libraries" we make for it.

Therefore, I had to make my own functions to handle this, but there were limitations.
* The functions had to work without while loops because Axon doesn't support while loops.
* I should avoid recursion as much as possible because it is particularly difficult in Axon, and the program would be running on embedded devices with low resources.
* The combinations should be autmatically sorted using the lexicographic order that they came in from the input list.
* No combinations could be repeated in a different order, e.g. [a,b] <=> [b,a]
* It had to be fast because it would be part of a tool and users can't stand waiting.

Now, I won't show the code I wrote for work, because it's a little proprietary and seeing it in Axon would mean nothing to most people.

Instead I will show it in Python, this is the language that the algorithm was originally prototyped in. (My boss walked in on me prototyping the algorithm and he was like "You've been staring at the same line for 20 mintues already", which it true. I even worked on this through lunch.)
```python
1  def combos(i,j,l): #i is input list, j is r in nCr, l is n in nCr
2   if j == 1:
3    return [[x] for x in xrange(l)] #[[0],[1],[2],...]
4   if j == l:
5    return [[x for x in xrange(l)]] #[[0,1,2,3,...]]
6   #remove lists that have too big starting number
7   i = [x for x in i if x[0] <= l-j]
8   t = []
9   for x in i:
10   if x[-1]+1 < l: #further remove lists that are too big
11    for y in xrange(x[-1]+1,l):
12     t.append(x+[y])
13  return t #returns list of index lists
14 
15 def genCombination(arr): # 2**n - 1 total combinations
16  o = []
17  s = len(arr)
18  g = []
19  #generate list combinations from lists of index lists
20  for x in xrange(1, s+1):
21    o = combos(o,x,s)
22   for y in o:
23    g.append([arr[z] for z in y])
24  return g
25 
26 #pretty print the combinations
27 for x in genCombination(['a','b','c','d']):
28  print x
```

Here is the output
```
['a']
['b']
['c']
['d']
['a', 'b']
['a', 'c']
['a', 'd']
['b', 'c']
['b', 'd']
['c', 'd']
['a', 'b', 'c']
['a', 'b', 'd']
['a', 'c', 'd']
['b', 'c', 'd']
['a', 'b', 'c', 'd']
```

I can't totally explain why it works, even though I made it, but I can explain a few of the design decisions.

The functions are designed to take the previous answer and use that to calculate the next answer, so no jumping to random combinations. It has to be sequential, which for my purposes was adequate.

Line 2 and 4 are just if statements that return a list of lists. \
This is because if you want the combination of nCr where r = 1, then it will look like [0],[1],[2],[3],[4],...,[n-1]. \
If you want the combination of nCr where r = n, then it will look like [0,1,2,3,...,n-1]. \
This is mainly for speed, no point in calculating it if you know what it's supposed to be.

Then there's a line that removes rows from the previous input that can't be used. \
This is because in cases like the rows of the output with only 2 elements, you can't add to the [d] row because there is nothing after d to add.

Then there is a nest of 2 for loops. \
I'm not really sure what happens here. I'm not even sure how I figured this out, but this is where the magic happens. \
It also contains an if statement to further filter out rows that can't work. (It originally didn't have an if statement there, but that gave me problems in Axon so I added it there to make it easier to port it to other languages.)

The combos() function that I just described doesn't actually make the list of combinations from the input list, instead it makes the list of combinations of indexes for the input list.

The genCombination() function is the one that starts the list for the previous input scheme, gets the combination for a different value of r in nCr, and maps the input list to the lists of indexes.

Then it returns the lists of the combinations of the inputs with the specifications listed above.

The listed output is exactly what you would get if you used ['a','b','c','d'] as your input list.

I hope this is helpful to someone. \
It would have helped me quite a bit if I could find something like this when I had to incorporate this feature into the program I had to modify. Instead I wasted about a whole work day making this and porting it to Axon and debugging it.

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2017/03/11 at 12:17 AM PST</sub>
