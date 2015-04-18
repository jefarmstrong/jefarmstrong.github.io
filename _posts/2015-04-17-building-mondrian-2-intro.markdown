---
layout: post
title:  "Building Mondrian2 - Intro"
date:   2015-04-17
categories: code 
---

<div class="row">
    <div class="col-sm-12">
        <p>
            I recently decided to create a sequel to Mondrian, my first puzzle game. For those of you that don't know, Mondrian is a sliding tile puzzle game inspired by the artwork of <a href="https://www.artsy.net/artist/piet-mondrian">Piet Mondrian</a>.

            The original game was very succesful and garnered some excellent reviews:
        </p>

        <blockquote class="bg-success">
            <p><b>"A work of Art"</b><i>"...its character, art style, and terrific gameplay justify the price. Mondrain can rest in peace knowing that his iDevice game has been transformed into a 5-Dimple work of art."</i></p>
            <footer>AppSmile.com</footer>
        </blockquote>

        <blockquote class="bg-success">
          <p><b>✭✭✭✭ 4 stars</b> <i>"Mondrian is a great game injecting loads of personality, and style into a genre that usually is just associated with blue blocks."</i></p>
          <footer>TouchGen.net</footer>
        </blockquote>

        <p>
            I decided to update the game, and in the process simplify it. The original game had this narrative that the ghost of Mondrian needed you to fix his paintings that where all over the world in order for him to escape some sort of artist limbo. As part of that story the game included the "ghost" of Mondrian who would both encourage you and insult you as you went along. 
        </p>

        <p>
            As part of the design of Mondrian2 I decided to simplify the game down to it's essence of beautiful hard puzzles coupled with a very cranky narrator.
        </p>
    </div>
</div>
<div class="row">
    <div class="col-sm-6">
        <h3>Current Screenshot</h3>
        <p>Here is a <b>work-in-progress</b> screenshot of the game screen. Keeping it simple and clean so far. That's Mondrian peeking up from the bottom.</p>
    </div>
    <div class="col-sm-6">
        <img src="/images/mondrian2/mondrian_screenshot_1.png" class="img-responsive"/>
    </div>
</div>
<div class="row">
    <div class="col-sm-12">
        <h3>Finding Best Solutions</h3>
        <p>
            One of requirements of the game is to know the best possible solution to each puzzle. The puzzles seems simple on the outset, but often, finding the optimal solution can be very challenging. When I first released Mondrian I thought I had the best solutions for each puzzle and one of the metrics that the game tracked was how many people solved each puzzle and the number of moves they took. With several puzzles I found that someone had found a solution in one or two moves better than I had. I didn't have their exact moves, just the count, so on those puzzles I would go back and rack my brain for days trying to cut out just one move. I found it a lot of fun knowing that there was <b>some</b> way to solve this puzzle in just 1 less move.
        </p>
        <p>
            The next blog post will deal with the app I wrote to solve the puzzles. It uses the A* (A star) algorithm and was written in Node.js and AngularJS.
        </p>
        <h3>Automated Insults</h3>
        <p>
            Another aspect of the original game that people enjoyed was the cranky Mondrian who would insult you for not finding a solution and begrudgingly encourage you when you did.
        </p>
        <p>
            For the sequal I wanted to have a lot more insults in the game. My brother <a href="http://jonarmstrong.com">Jon Armstrong</a> wrote all the original comments by hand. For the sequal I wanted to combine hand crafted insults with an algorithm to generate thousands of unique ones to add to the freshness. I'll discuss that in a future blog post.
        </p>
        <h3>Next Up</h3>
        <p>
            <a href="/posts/building-mondrian2-astar">Using A* to solve puzzles</a>
        </p>        


    </div>

</div>


