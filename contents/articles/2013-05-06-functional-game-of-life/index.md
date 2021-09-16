---
title: Functional Game of Life part 1 (C#)
author: Liam McLennan
date: 2013-05-06 20:32
template: article.jade
---

<style>
.cell {
    border: 2px solid black;
    width: 18px;
    height: 18px;
    position: absolute;background-color: #eee;
}
.on {
    background-color: #333;
}
</style>

<img src="Gospers_glider_gun.gif" align="left" style="margin:20px;"/> I thought it might be interesting to solve the [Conway's Game of Life](http://en.wikipedia.org/wiki/Conway's_Game_of_Life) cellular automaton in a functional manner. Game of life is an interesting problem for experimentation because it is simple and well understood. 

Where to Start?
----------------

I'll start by defining a way to render the world. I want to start here because it is useful to have a nice way to visualise the state of the system, also it conveniently forces me to define my data model. I will model the system as a collection of living cells, each with X and Y coordinates (zero based). The living cells define the boundaries of the system. Dead cells are implicitly the cells within the system boundary that are not living. A system defined as:

<div style="clear: both;"></div>


    new Cell(1,0), new Cell(0,1), new Cell(2,1), new Cell(7,16)

will produce [the following output](http://jsfiddle.net/P44aM/1/) (living cells are darker):

<div style="clear:both;height: 400px;">
<div style="position:relative;">
    <div class="cell " style="left: 20px; top: 20px;">&nbsp;</div>
    <div class="cell on" style="left: 20px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 20px; top: 340px;">&nbsp;</div>
    <div class="cell on" style="left: 40px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 40px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 20px;">&nbsp;</div>
    <div class="cell on" style="left: 60px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 60px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 80px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 100px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 120px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 320px;">&nbsp;</div>
    <div class="cell " style="left: 140px; top: 340px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 20px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 40px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 60px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 80px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 100px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 120px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 140px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 160px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 180px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 200px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 220px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 240px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 260px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 280px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 300px;">&nbsp;</div>
    <div class="cell " style="left: 160px; top: 320px;">&nbsp;</div>
    <div class="cell on" style="left: 160px; top: 340px;">&nbsp;</div>
</div>
</div>


To get to this I started with a test (using [giv.n](http://nuget.org/packages/Giv.n/) and nunit).

    [TestFixture]
    public class RenderingTests
    {
        private List<Cell> _cells;
        private string _result;

        [Test]
        public void RenderASmallSystem()
        {
            Giv.n(ASmallSystem);
            Wh.n(ItIsRendered);
            Th.n(() => ItShouldRenderDimensions(8, 17))
                .And(() => LivingCellCountIs(_cells.Count));
        }

        private void ASmallSystem()
        {
            _cells = new List<Cell> { new Cell(1,0), new Cell(0,1), new Cell(2,1), new Cell(7,16) };
        }

        private void ItIsRendered()
        {
            _result = GameOfLife.Render(_cells);
        }

        private void ItShouldRenderDimensions(int x, int y)
        {
            Assert.AreEqual(x*y, CountOccurances(_result,"<div"));
        }

        private void LivingCellCountIs(int count)
        {
            Assert.AreEqual(count, CountOccurances(_result,"on"));
        }

        private static int CountOccurances(string source, string sub)
        {
            return source.Select((c, i) => source.Substring(i)).Count(s => s.StartsWith(sub));
        }
    }

and the implementation so far:

    public class GameOfLife
    {
        public static string Render(List<Cell> cells)
        {
            var largestX = cells.Max(c => c.X);
            var largestY = cells.Max(c => c.Y);
            Func<int,int,bool, string> buildCell = (x,y,on) => 
                string.Format("<div class=\"cell {2}\" style=\"left: {0}px; top: {1}px;\">&nbsp;</div>", 20 * (x+1), 20 * (y+1), @on ? "on" : "");

            var results = from x in Enumerable.Range(0, largestX+1)
                          from y in Enumerable.Range(0, largestY+1)
                          select buildCell(x, y, IsOn(cells, x, y));
            return string.Join("\n", results);
        }

        private static bool IsOn(IEnumerable<Cell> cells, int x, int y)
        {
            return cells.Any(c => c.X == x && c.Y == y);
        }
    }

    public struct Cell
    {
        public int X;
        public int Y;

        public Cell(int x, int y)
        {
            X = x;
            Y = y;
        }
    }

Source
-------

The source for this article is in a [public git repo](https://github.com/liammclennan/functional-game-of-life), under the tag `data-and-render`.

Next
----

The next post in this series will look at randomly seeding the system with living cells, so that the cellular evolution can be started. 
