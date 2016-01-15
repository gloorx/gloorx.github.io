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

###2016/1/15
There is a great improvement of speed. I used two matrix, one has current status of cells, an other has next status of cells. But I think a single matrix is easier to access than two matrix. So I create class *Cell* having current status and next status.

One more thing. I find map function of Numpy, *Numpy.vectorize*, that make it faster 5 times. But I have no idea why original python map function doesn't work.

...Am I stupid? I forgot I don't have to access every place in matrix. I make a list which has only Cells. Then 1000x1000-size matrix works as faster as 10x10, if both have same amount Cells.