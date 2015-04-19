---
layout: post
title:  "Building Mondrian2 - A* (A star)"
date:   2015-04-17
categories: code 
---
<div class="row">
    <div class="col-sm-12">
        <p>
            <a href="/posts/building-mondrian-2-intro/">First post, if you missed it</a>
        </p>
        <h3>Solving Puzzles with A*</h3>
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
            Here's the core A* loop that runs on the server. (Node.js)
        </p>
        <div>
        <pre>       
        var nextState = this.openStates.shift();
        var state = this.closedSet[nextState.hash];
        if (this.isOverMoveCount(state)) {
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
        <ol>
            <li>The this.openStates is a sorted array with the lowest scoring state first (the heuristic gives a score of zero when the state matches the goal). </li>
            <li>We get the actual state out of the closedSet object.</li>
            <li>If we have already found at least one solution we throw out any states that are over that move count.</li>
            <li>Then we get all the movable tiles and generate all the valid moves for each.</li>
            <li>By applying the move we generate a new board state, which we save into the closed set.</li>
            <li>Then we score that state and if it scores a zero, we found a solution, otherwise we insert it in the correct position (based on it's score) into the openStates array.</li>
            <li>We repeat this until there are no more states in openStates. This can take a very long time on boards with many tiles and lots of moves, so often I'll just stop it once it's run for close to a million iterations.</li>
        </ol>
    </div>
</div>            
<div class="row">
    <div class="col-sm-6">
        <h3>Example Solution</h3>
        <p>Here's an example of a simpler puzzle that the A* algorithm found a solution for in 17 moves. I left it running and it eventually found a better solution in 15 moves. As you can see, this simple algorithm can produce some intelligent looking solutions.</p>
    </div>
    <div class="col-sm-6">
        <img src="/images/mondrian2/solution2.gif" class="img-responsive"/>
    </div>
</div>

<div class="row" style="margin-top:20px;">
    <div class="col-sm-12">
        <img class="img-responsive" src="/images/mondrian2/mondrian_maker_2.png"/>
        <p>
            Here the solver is running. It's looked at over 75 thousand board states and has over 61 thousand in the priority queue. At this point it's already found 3 solutions with 29 moves being the best so far. 
        </p>
        <h3>The Heuristic</h3>
        <p>
            But wait, you say, did I say that with a good heuristic you will find the best solution first? Well, yes, this points out an issue with the heuristic score. Given the nature of these puzzles and the very large search spaces we are dealing with there are some moves that don't appear to be good moves initially, but which turn out to be good later. For example, in the puzzle shown above I'll give you a hint. The best first move that I have found is to move the top yellow tile to the left. But when the heuristic scoring algorithm looks at that resulting board state it doesn't score that particularly well because you are moving a tile from it's goal position.  
        </p>
        <p>Here's the heuristic scoring code. It goes through all the tiles in the goal and finds matching colored titles in the current board state. The lowest score found is used for that tile. If the resulting score in more than zero (the solution) then we also give a penalty for the number of moves.</p>
<pre>       
evaluatePosition: function(puzzleState, options) {
    options = options || {};
    var score = 0;
    puzzleState.outOfPosition = 0;
    var _this = this;
    _.forEach(this.goalState, function(goalObj) {
        if (isGameObject(goalObj)) {
            var matching = puzzleState.objectsMatchingColor(goalObj.color);
            var scr = Number.MAX_VALUE;
            _.forEach(matching, function(obj2) {
                var dist = _this.distance(goalObj, obj2);
                scr = Math.min(scr, dist);
            });
            score += scr;
            if (scr) puzzleState.outOfPosition++;
        }
    });
    var penalty = puzzleState.moveCount() * this.movePenalty;
    if (score > 0) score += penalty;
    puzzleState.score = score;
    return score;
}
</pre>
<p>I've tried different algorithms for the distance function but the standard Euclidean distance seems to work fine.</p>
<pre>
distance: function(obj1, obj2) {
    var p1 = rowColForIndex(obj1.index);
    var p2 = rowColForIndex(obj2.index);
    return Math.sqrt( Math.pow(p1.row-p2.row, 2) + Math.pow(p1.col-p2.col, 2) );
}    
</pre>

        <p>
            So here's what happens to that first "best" move. It gets scored low compared to one or more other moves in that first set of six moves. In the next iteration of the loop one of the other board states (which scored the best) is pulled off the queue and all the valid moves from that state are generated (with 6 moveable tiles the maximum moves for each state is 24, but that never happens since each tile usually can't move in all four directions). For this puzzle in particular we average about 13.5 moves per state. So, as you can see very quickly that initial poorly scoring move gets pushed farther and farther down in the priority queue. With the penalty for moves the initial poorly scoring state will end up at the top of the priority, but sometimes it can take a long time.
        </p>
        <h3>Future Directions</h3>
        <p>            
            Some things that I have tried to overcome the "lost" counter-intuitive moves: 
            <ul>
                <li>Perform short brute force searches from a given position to see if a better scoring position arises. This didn't help that much since often the benefit of a counter-intuative move doesn't become apparent till much later.</li>
                <li>Creating a "heat-map" of good positions on any given puzzle board. These would obviously by the goal positions, but also positions that help other tiles get into those goal positions. This seems like a promising idea, but so far it hasn't garnered much.</li>
                <li>Manually help the algorithm by providing some initial moves or by starting the puzzle at manually created waypoints. This approach has worked on several puzzles but the issue is always if the manual overrides are creating a local maximum.</li>                
            </ul>
        </p>
        <h3>Node.js Details</h3>
        The node app is just an express backend that serves up the AngularJS app, and a controller that handles kicking off the A* program. 
        The A* code is forked from the main server and communication is handled via messages. Also, each loop of the core algorithm is run once and the kicked off again with a call to setImmediate() to keep the thread from locking up.
    </div>
</div>            
