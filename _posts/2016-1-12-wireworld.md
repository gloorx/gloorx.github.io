---
tag: CODE 
title: Wireworld
---

Source code : [https://github.com/pueue/celluler_automata](https://github.com/pueue/celluler_automata)

###2016/1/12
In 2D array called matrix, every cell is an object which have two member variable current_status and next_status. It works. You can watch how to running on console.

I visualize each cell using Pygame.

###2016/1/13
If I expand the matrix like 100x100 size, this code work slowly. I must try to solve it.

I merge two function which take O(n). Now each cell calculates and visualize at once. But the code is still slow.

Numpy array make it little faster.