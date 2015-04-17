---
layout: post
title:  "Building Mondrian2 - A* (A star)"
date:   2015-04-17
categories: code 
---
<div class="row">
    <div class="col-sm-12">
        <p>
            There are many algorithms used to solve path finding problems. One of the most popular is the A* (pronounced A star) algorithm. It's really very simple, yet can yield some intelligent looking results.
        </p>
        <p>
            I'll use the context of the Mondrian puzzles to describe the A* algorithm, so here is a little background.
        </p>
    </div>
</div>
<div class="row">
    <div class="col-sm-6">
        <img class="img-responsive" src="/images/mondrian2/puzzle_start.png"/>
        This is an example starting position of a puzzle. Black tiles don't move, all the colored tiles do, but they slide until they hit either an edge or another tile.
    </div>
    <div class="col-sm-6">
        <img class="img-responsive" src="/images/mondrian2/puzzle_end.png"/>
        This is the goal of this puzzle. It can be reached in 22 moves.
    </div>
</div>            
<div class="row" style="margin-top:20px;">
    <div class="col-sm-12">
        <p>
            So, here in a nutshell, is the <a href="http://en.wikipedia.org/wiki/A*_search_algorithm">A* algorithm</a>. You start with the begining position, we'll call that a board state. From that state, you generate a list of all posible valid moves. So, in our case, that yellow tile on the bottom left has no valid moves since it's surrounded by other tiles. The red tile to it's right has one valid move as do all the tiles along the bottom. The yellow tile at the top has two valid moves. So, we end up with six valid moves for that first state. For each move we create a new state and we give that new state a score (the heuristic score). We put all the new states in an ordered list based on their score. Then we take the best scoring state off that list and repeat the process. That's it, really.
        </p>
        <p>
            Ok, there's a little more to it, but not much. Obviously the heuristic score is key. If it's really good then you are guaranteed to get to the optimal solution in the shortest time, if it's bad, then the whole thing becomes a brute force search. The heuristic for the Mondrian solver that I wrote uses a modified distance algorithm between each tile's position and it's goal position. There are some problems with this that I'll go into later, but for the most part it works. 
        </p>
        <p>
            There is also a "closed set" where each board state that has been evaluated is saved (or in our case a hash of that board state). When new state are generated from a list of moves each one is checked against the closed set to see if we have already seen it. If we have and the number of moves to reach this state is the same or more that the one in the closed set, then it is disgarded. 
        </p>
    </div>
</div>
<div class="row" style="margin-top:20px;">
    <div class="col-sm-12">
        <img class="img-responsive" src="/images/mondrian2/mondrian_maker_1.png"/>
        Here is a screen shot of the AngularJS app I built to design and solve the puzzles. Using the palette on the left you can design the puzzle starting position and the goal position. When you click 'Solve it' the board layouts are sent the the server and the A* algorithm starts attempting to solve it. The client then pings the server for updates. 
    </div>
</div>            
<div class="row" style="margin-top:20px;">
    <div class="col-sm-12">
        <p>
            Here's the core loop that runs on the server. (Node.js)
        </p>
        <div>
        <pre>       
        var nextState = this.openStates.shift();
        var state = this.closedSet[nextState.hash];
        if (this.isOverMoveCount(state)) {
            return;
        }
        if (state.moveCount() !== nextState.moveCount) {
            return;
        }
        var _this = this;
        var movable = state.currentMovableObjects();
        _.forEach(movable, function(obj) {
            var moves = state.validMovesForObjectAtIndex(obj.index);
            _.forEach(moves, function(move){
                var aState = state.createByApplyingMove(move);
                aState = _this.insertIntoClosedSet(aState);
                if (!aState) {
                    return true;
                }
                _this.evaluatePosition(aState);
                if (aState.score === 0) {
                    if (aState.moveCount() < _this.bestSolutionMoves) {
                        _this.bestSolutionMoves = aState.moveCount();
                    }
                    _this.solutions.push(aState);
                }else {
                    if (! _this.isOverMoveCount(aState)) {
                        _this.insertIntoOpenStates(aState);
                    }
                }
            });
        });
        </pre>
        </div>

    </div>
</div>            
<div class="row" style="margin-top:20px;">
    <div class="col-sm-12">
        <img class="img-responsive" src="/images/mondrian2/mondrian_maker_2.png"/>
        <p>
            Here the solver is running. It's looked at over 75 thousand board states and has over 61 thousand in the priority queue. At this point it's already found 3 solutions with 29 moves being the best so far. 
        </p>
        <p>
            But wait, you say, did I say that with a good heuristic you will find the best solution first? Well, yes, this points out an issue with the heuristic score. Given the nature of these puzzles and the very large search spaces we are dealing with there are some moves that don't appear to be good moves initially, but which turn out to be good later. For example, in the puzzle shown I'll give you a hint. The best first move that I have found is to move the top yellow tile to the left. But when the heuristic scoring algorithm looks at that resulting board state it doesn't score that particularly well because you are moving a tile from it's goal position.  
        </p>
        <p>
            So here's what happens to that first "best" move. It gets scored low compared to one or more other moves in that first set of six moves. In the next iteration of the loop one of the other board states (which scored the best) is pulled off the queue and all the valid moves from that state are generated (with 6 moveable tiles the maximum moves for each state is 24, but that never happens since each tile usually can't move in all four directions). For this puzzle in particular we average about 13.5 moves per state. So, as you can see very quickly that initial poorly scoring move gets pushed farther and farther down in the priority queue. 
        </p>
        <p>            
            We can balance that somewhat by also applying a move count penalty to the heuristic so that states with a lot of moves score less and less well. 
        </p>
    </div>
</div>            
