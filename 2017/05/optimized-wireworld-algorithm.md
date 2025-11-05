# Optimized WireWorld Algorithm

Saturday, May 20, 2017

---

As usual, this entry was written because it is an uncommon subject that I just happen to be working on.

I've been slowly working on a virtual reality world and decided that it would be nice to add some technology to the simulation. I decided that a custom version of the [WireWorld](https://en.wikipedia.org/wiki/Wireworld) cellular automaton in 3 dimensions would be perfect since it is Turing complete, can run relatively fast, and would be rather simple to integrate with any other in-game technologies.

It had to be customized because it had to be fast (to prevent simulator sickness in VR), but there is very little interest in this automaton and all the code I've seen uses the traditional implementation. This is why I came up with this version (which someone probably has done and I just didn't know about it).

In this blog post, I will be describing an optimized version of the algorithm for WireWorld. In theory this version is a little slower than the classical 2 array version of WireWorld on small simulations, but should slowly become faster than the traditional implementation as the simulation gets bigger.

The rules of the automaton state that when a cell is an electron head, it will degrade into an electron tail. When the cell is an electron tail, it will degrade into a conductor. When the cell is a conductor, it can turn into an electron head only if one or two of its immediate neighbors are electron heads.

The way this is usually implemented is with an array where each element holds the state of the cell. A value of 0 is nothing, 1 is an electron head, 2 is an electron tail, and 3 is a conductor. The simulation goes through and writes the updated states to a second array in order to retain the new states without altering the current frame. Then the second array is copied back onto the first and is displayed and the process starts all over again. This works fine, but requires you to iterate through each array element and it if it has live neighbors and degrade its state until it becomes a conductor. Going through each element for every single frame is generally an O(n<sup>2</sup>) time complexity, but is better on a small scale because it would use less memory and less time in general.

The way I will write about here will only go through the cells that actually have a state that isn't 0. Instead of holding the states in an array, the cells are held in a list of data structures that contain the x & y coordinates, the state, the addresses of the immediate neighbors, and a count of the live neighbors. The simulation goes through each cell a first time and if the cell is an electron head, then it will go through the list of neighbors and update the live neighbor count if the neighbor is a conductor. Then each cell will be iterated through a second time to degrade the state and then change its state based on if it had the 1 or 2 neighbors or not. This way all the empty "cells" that would have been in an array are skipped over since they are not in the list.

Of course, each cell in the second implementation takes more time and memory to execute on a small scale, but on a large scale there would be much more empty space that could be skipped and at a certain point the second implementation should be able to run faster than the first implementation. The time complexity for the second implementation should be something like O(n). In practice, this is only better in large scales because even though O(n) is less than O(n<sup>2</sup>), it uses more memory and computing power per cell, but doesn't rise in power and memory as fast as O(n<sup>2</sup>) would.

Enough of this theory stuff, lets get some actual pseudo code.
```
define cell data type
    x position
    y position
    state
    dynamic list 'neighbors'
    live neighbor count
 
initialize dynamic list 'wires'
set erase mode to false
 
define find function, input x and y coordinates
    loop through each element in 'wires', return index if coordinates match
    return -1 if no cell found at those coordinates
 
define get neighbors function, input cell
    loop through each coordinate around the cell's position
        if cells exist at those coordinates, add reference to cell's neighbor list
define add cell function, input x and y coordinates
    add new blank cell to wires list
    run get neighbors function on this new cell
    iterate through the neighbor list and add this cell to those neighbor's neighbor list
define delete cell function, input cell
    loop through each cell in cell's neighbor list
        remove the reference to this cell from the neighbor's neighbor list
    remove reference to this cell from the wires list
infinite loop
    clear screen
    if enter is pressed, toggle erase mode
    get mouse position and mouse buttons
    draw mouse cursor onto screen 
 
    if mouse click
        get index of cell at mouse coordinates with find function
        if left click and not erase mode and no cell existing at mouse coords
            call add cell function with mouse coordinates
        if left click and erase mode and cell existing at mouse coords
            call delete cell function with cell from index in wires list
        if right click and cell existing at mouse coordinates
            set cell at index to have stage of 1 (electron head)
    loop through cells in wires list
        if cell state is electron head
            loop through neighbors that are conductors
                increment live neighbor count of that neighbor cell
 
    loop through cells in wires list
        if cell is not 3 (conductor)
            increment cell state
        if cell's live neighbor count is 1 or 2
            change state to 2 (electron head )
        clear cell's live neighbor count to 0
        set color equal to black for 0 (void), blue for 1 (electron head), red for 2 (electron tail), yellow for 3 (conductor)
        draw cell on screen with coordinates and color
```

The Python source for this looks like this:
(Left click adds a conductor, right click activates it, pressing enter turns on delete mode)
```python
#!/usr/bin/env python
import pygame
from pygame.locals import *
pygame.display.init()
pygame.font.init()
screen = pygame.display.set_mode((640,480))
pygame.display.set_caption("")
font = pygame.font.Font(None,20)
clock = pygame.time.Clock()
           # 0   1   2      3          4
wires = [] #[xp, yp, state, neighbors, live]
# 0 black void, 1 blue electron head, 2 red electron tail, 3 yellow wire
erase = False

def findWire(xp,yp):
 for w in xrange(len(wires)):
  if wires[w][0]==xp and wires[w][1]==yp: return w
 return -1
def updateNeighbors(ww):
 for x in xrange(-1,2):
  for y in xrange(-1,2):
   wp = findWire(ww[0]+x,ww[1]+y)
   if (x,y) != (0,0) and wp != -1:
    ww[3].append(wires[wp])
def insertWire(wx,wy):
 wires.append([wx,wy,3,[],0])
 updateNeighbors(wires[-1])
 for ww in wires[-1][3]:
  ww[3].append(wires[-1])
def deleteWire(ww):
 for w in ww[3]: #remove references from neighbors
  for wg in xrange(len(w[3])):
   if ww[0]==w[3][wg][0] and ww[1]==w[3][wg][1]:
    w[3].pop(wg)
    break
 wires.pop(findWire(ww[0], ww[1])) #remove reference from wires

while True:
 for event in pygame.event.get():
  if event.type == QUIT: exit()
  elif event.type == KEYDOWN:
   if event.key == K_RETURN: erase = not erase
   elif event.key == K_ESCAPE: exit()
 screen.fill((0,0,0))
 
 mx,my = pygame.mouse.get_pos()
 b = pygame.mouse.get_pressed()
 if erase: c = (100,0,0)
 else: c = (50,50,50)
 pygame.draw.rect(screen,c,(mx/5*5,my/5*5,5,5))
 mx /= 5
 my /= 5
 w = findWire(mx,my)
 if b[0]:
  if erase and w != -1: deleteWire(wires[w])
  elif not erase and w == -1: insertWire(mx,my)
 elif b[2] and w != -1:
  wires[w][2] = 1

 for w in wires:
  if w[2] == 1: #inc neighbor counters
   for v in w[3]:
    if v[2] == 3: v[4] += 1

 for w in wires:
  if w[2] != 3: w[2] += 1
  if w[4] == 1 or w[4] == 2: w[2] = 1
  w[4] = 0
  c = [(0,0,0),(0,0,255),(255,0,0),(255,255,0)][w[2]]
  #c = [(0,0,0),(0,255,255),(0,100,0),(40,40,40)][w[2]]
  pygame.draw.rect(screen,c,(w[0]*5,w[1]*5,5,5))
 
 clock.tick()
 screen.blit(font.render("%.2f"%clock.get_fps(),-1,(255,255,255)),(0,0))
 
 pygame.display.flip()
 ```

And as always, if you have some suggestions or ways to optimize this algorithm further, please let me know and I'll be more than happy to update this post with the new information.

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2017/05/20 at 1:08 AM PDT</sub>
