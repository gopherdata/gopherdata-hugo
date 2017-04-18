+++
title = "Deep Learning from Scratch in Go - Part 1: Equations Are Graphs"
subtitle = ""
date = "2017-04-19T08:43:45+10:00"
math = true
draft = true

+++

(Author: Chewxy, @chewxy on [Twitter](https://twitter.com/chewxy) and Gophers Slack)

Welcome to the first part of many about writing deep learning algorithms in Go. The goal of this series is to go from having no knowledge at all to implementing some of the latest developments in this area. 

[Deep learning](https://en.wikipedia.org/wiki/Deep_learning) is not new. In fact the idea of deep learning was spawned in the early 1980s. What's changed since then is our computers - they have gotten much much more powerful.  In this blog post we'll start with something familiar, and edge towards building a conceptual model of deep learning. We won't define deep learning for the first few posts, so don't worry so much about the term.

There are a few terms of clarification to be made before we begin proper. In this series, the word "graph" refers to the concept of graph as used in [graph theory](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)). For the other kind of "graph" which is usually used for data visualization, I'll use the term "chart".

## Computation ##
I'm going to start by making a claim: all programs can be represented as graphs. This claim is not new, of course. Nor is it bold or revolutionary. It's the fundamental theory that computer scientists have been working on ever since the birth of the field of computation. But you may have missed it. If you have missed it, the logic goes as such:

1. All modern computer programs run on what essentially is a [Turing Machine](https://en.wikipedia.org/wiki/Turing_machine).
2. All Turing machines are equivalent to untyped lambda calculus (this is commonly known as the [Church-Turing thesis](https://en.wikipedia.org/wiki/Church_Turing_thesis))
3. Lambda calculus can be represented as graphs.
4. Therefore all programs can be represented as graphs.

To make this idea more concrete, let's look at a simple program:

```go
func main() {
	fmt.Printf("%v", 1+1)	
}
``` 

This generates an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) like so (the AST was generated with a library built on top of [goast-viewer](https://github.com/yuroyoro/goast-viewer)):


<div style="margin-left:auto; margin-right:auto;">
<svg style="width:100%; height:auto;"
 viewBox="0.00 0.00 1167.98 548.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 544)">
<title>%3</title>
<polygon fill="white" stroke="none" points="-4,4 -4,-544 1163.98,-544 1163.98,4 -4,4"/>
<!-- mainFn -->
<g id="node1" class="node"><title>mainFn</title>
<ellipse fill="none" stroke="black" cx="445.942" cy="-522" rx="52.7911" ry="18"/>
<text text-anchor="middle" x="445.942" y="-518.3" font-family="Times,serif" font-size="14.00">func main()</text>
</g>
<!-- mainBody -->
<g id="node2" class="node"><title>mainBody</title>
<ellipse fill="none" stroke="black" cx="445.942" cy="-450" rx="61.1893" ry="18"/>
<text text-anchor="middle" x="445.942" y="-446.3" font-family="Times,serif" font-size="14.00">*ast.BlocStmt</text>
</g>
<!-- mainFn&#45;&gt;mainBody -->
<g id="edge1" class="edge"><title>mainFn&#45;&gt;mainBody</title>
<path fill="none" stroke="black" d="M445.942,-503.697C445.942,-495.983 445.942,-486.712 445.942,-478.112"/>
<polygon fill="black" stroke="black" points="449.442,-478.104 445.942,-468.104 442.442,-478.104 449.442,-478.104"/>
</g>
<!-- bodyList -->
<g id="node3" class="node"><title>bodyList</title>
<ellipse fill="none" stroke="black" cx="445.942" cy="-378" rx="46.2923" ry="18"/>
<text text-anchor="middle" x="445.942" y="-374.3" font-family="Times,serif" font-size="14.00">[]ast.Stmt</text>
</g>
<!-- mainBody&#45;&gt;bodyList -->
<g id="edge2" class="edge"><title>mainBody&#45;&gt;bodyList</title>
<path fill="none" stroke="black" d="M445.942,-431.697C445.942,-423.983 445.942,-414.712 445.942,-406.112"/>
<polygon fill="black" stroke="black" points="449.442,-406.104 445.942,-396.104 442.442,-406.104 449.442,-406.104"/>
</g>
<!-- list0 -->
<g id="node4" class="node"><title>list0</title>
<ellipse fill="none" stroke="black" cx="445.942" cy="-306" rx="61.99" ry="18"/>
<text text-anchor="middle" x="445.942" y="-302.3" font-family="Times,serif" font-size="14.00">*ast.ExprStmt</text>
</g>
<!-- bodyList&#45;&gt;list0 -->
<g id="edge3" class="edge"><title>bodyList&#45;&gt;list0</title>
<path fill="none" stroke="black" d="M445.942,-359.697C445.942,-351.983 445.942,-342.712 445.942,-334.112"/>
<polygon fill="black" stroke="black" points="449.442,-334.104 445.942,-324.104 442.442,-334.104 449.442,-334.104"/>
</g>
<!-- call -->
<g id="node5" class="node"><title>call</title>
<ellipse fill="none" stroke="black" cx="445.942" cy="-234" rx="59.2899" ry="18"/>
<text text-anchor="middle" x="445.942" y="-230.3" font-family="Times,serif" font-size="14.00">*ast.CallExpr</text>
</g>
<!-- list0&#45;&gt;call -->
<g id="edge4" class="edge"><title>list0&#45;&gt;call</title>
<path fill="none" stroke="black" d="M445.942,-287.697C445.942,-279.983 445.942,-270.712 445.942,-262.112"/>
<polygon fill="black" stroke="black" points="449.442,-262.104 445.942,-252.104 442.442,-262.104 449.442,-262.104"/>
</g>
<!-- fn -->
<g id="node6" class="node"><title>fn</title>
<ellipse fill="none" stroke="black" cx="303.942" cy="-162" rx="73.387" ry="18"/>
<text text-anchor="middle" x="303.942" y="-158.3" font-family="Times,serif" font-size="14.00">*ast.SelectorExpr</text>
</g>
<!-- call&#45;&gt;fn -->
<g id="edge5" class="edge"><title>call&#45;&gt;fn</title>
<path fill="none" stroke="black" d="M416.174,-218.326C395.302,-208.037 367.156,-194.162 344.272,-182.881"/>
<polygon fill="black" stroke="black" points="345.662,-179.664 335.145,-178.382 342.567,-185.943 345.662,-179.664"/>
</g>
<!-- args -->
<g id="node9" class="node"><title>args</title>
<ellipse fill="none" stroke="black" cx="587.942" cy="-162" rx="46.2923" ry="18"/>
<text text-anchor="middle" x="587.942" y="-158.3" font-family="Times,serif" font-size="14.00">[]ast.Expr</text>
</g>
<!-- call&#45;&gt;args -->
<g id="edge8" class="edge"><title>call&#45;&gt;args</title>
<path fill="none" stroke="black" d="M475.71,-218.326C497.582,-207.544 527.443,-192.823 550.866,-181.277"/>
<polygon fill="black" stroke="black" points="552.721,-184.265 560.142,-176.704 549.625,-177.986 552.721,-184.265"/>
</g>
<!-- fnPkg -->
<g id="node7" class="node"><title>fnPkg</title>
<ellipse fill="none" stroke="black" cx="92.9418" cy="-90" rx="92.8835" ry="18"/>
<text text-anchor="middle" x="92.9418" y="-86.3" font-family="Times,serif" font-size="14.00">*ast.Ident (Name: fmt)</text>
</g>
<!-- fn&#45;&gt;fnPkg -->
<g id="edge6" class="edge"><title>fn&#45;&gt;fnPkg</title>
<path fill="none" stroke="black" d="M262.255,-147.17C229.431,-136.281 183.405,-121.012 147.625,-109.141"/>
<polygon fill="black" stroke="black" points="148.418,-105.717 137.825,-105.89 146.214,-112.361 148.418,-105.717"/>
</g>
<!-- fnFn -->
<g id="node8" class="node"><title>fnFn</title>
<ellipse fill="none" stroke="black" cx="303.942" cy="-90" rx="100.182" ry="18"/>
<text text-anchor="middle" x="303.942" y="-86.3" font-family="Times,serif" font-size="14.00">*ast.Ident (Name: Printf)</text>
</g>
<!-- fn&#45;&gt;fnFn -->
<g id="edge7" class="edge"><title>fn&#45;&gt;fnFn</title>
<path fill="none" stroke="black" d="M303.942,-143.697C303.942,-135.983 303.942,-126.712 303.942,-118.112"/>
<polygon fill="black" stroke="black" points="307.442,-118.104 303.942,-108.104 300.442,-118.104 307.442,-118.104"/>
</g>
<!-- args0 -->
<g id="node10" class="node"><title>args0</title>
<ellipse fill="none" stroke="black" cx="587.942" cy="-90" rx="165.971" ry="18"/>
<text text-anchor="middle" x="587.942" y="-86.3" font-family="Times,serif" font-size="14.00">*ast.BasicLit (Kind: STRING) (Value: %v)</text>
</g>
<!-- args&#45;&gt;args0 -->
<g id="edge9" class="edge"><title>args&#45;&gt;args0</title>
<path fill="none" stroke="black" d="M587.942,-143.697C587.942,-135.983 587.942,-126.712 587.942,-118.112"/>
<polygon fill="black" stroke="black" points="591.442,-118.104 587.942,-108.104 584.442,-118.104 591.442,-118.104"/>
</g>
<!-- args1 -->
<g id="node11" class="node"><title>args1</title>
<ellipse fill="none" stroke="black" cx="868.942" cy="-90" rx="97.4827" ry="18"/>
<text text-anchor="middle" x="868.942" y="-86.3" font-family="Times,serif" font-size="14.00">*ast.BinaryExpr (Op: +)</text>
</g>
<!-- args&#45;&gt;args1 -->
<g id="edge10" class="edge"><title>args&#45;&gt;args1</title>
<path fill="none" stroke="black" d="M625.839,-151.559C671.25,-140.247 748.104,-121.102 803.251,-107.364"/>
<polygon fill="black" stroke="black" points="804.426,-110.679 813.284,-104.865 802.734,-103.886 804.426,-110.679"/>
</g>
<!-- lhs -->
<g id="node12" class="node"><title>lhs</title>
<ellipse fill="none" stroke="black" cx="718.942" cy="-18" rx="141.075" ry="18"/>
<text text-anchor="middle" x="718.942" y="-14.3" font-family="Times,serif" font-size="14.00">*ast.BasicLit (Kind: INT) (Value: 1)</text>
</g>
<!-- args1&#45;&gt;lhs -->
<g id="edge11" class="edge"><title>args1&#45;&gt;lhs</title>
<path fill="none" stroke="black" d="M834.904,-73.1159C813.785,-63.26 786.413,-50.4867 763.567,-39.825"/>
<polygon fill="black" stroke="black" points="764.951,-36.6089 754.409,-35.5516 761.991,-42.9522 764.951,-36.6089"/>
</g>
<!-- rhs -->
<g id="node13" class="node"><title>rhs</title>
<ellipse fill="none" stroke="black" cx="1018.94" cy="-18" rx="141.075" ry="18"/>
<text text-anchor="middle" x="1018.94" y="-14.3" font-family="Times,serif" font-size="14.00">*ast.BasicLit (Kind: INT) (Value: 1)</text>
</g>
<!-- args1&#45;&gt;rhs -->
<g id="edge12" class="edge"><title>args1&#45;&gt;rhs</title>
<path fill="none" stroke="black" d="M902.979,-73.1159C924.099,-63.26 951.47,-50.4867 974.317,-39.825"/>
<polygon fill="black" stroke="black" points="975.892,-42.9522 983.474,-35.5516 972.932,-36.6089 975.892,-42.9522"/>
</g>
</g>
</svg>
</div>

By this, we can also say that any equation that can be represented as a computer program, and a computer program can be represented as a graph. In particular, let's zoom in `1+1` part:

This corresponds to this part of the cleaned up graph (with unnecessary nodes removed):

<div style="margin-left:auto; margin-right:auto;">
<svg style="width:100%; height:auto;"
 viewBox="0.00 0.00 590.07 116.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 112)">
<title>%3</title>
<polygon fill="white" stroke="none" points="-4,4 -4,-112 586.075,-112 586.075,4 -4,4"/>
<!-- args1 -->
<g id="node1" class="node"><title>args1</title>
<ellipse fill="none" stroke="black" cx="291.037" cy="-90" rx="97.4827" ry="18"/>
<text text-anchor="middle" x="291.037" y="-86.3" font-family="Times,serif" font-size="14.00">*ast.BinaryExpr (Op: +)</text>
</g>
<!-- lhs -->
<g id="node2" class="node"><title>lhs</title>
<ellipse fill="none" stroke="black" cx="141.037" cy="-18" rx="141.075" ry="18"/>
<text text-anchor="middle" x="141.037" y="-14.3" font-family="Times,serif" font-size="14.00">*ast.BasicLit (Kind: INT) (Value: 1)</text>
</g>
<!-- args1&#45;&gt;lhs -->
<g id="edge1" class="edge"><title>args1&#45;&gt;lhs</title>
<path fill="none" stroke="black" d="M257,-73.1159C235.88,-63.26 208.509,-50.4867 185.663,-39.825"/>
<polygon fill="black" stroke="black" points="187.047,-36.6089 176.505,-35.5516 184.087,-42.9522 187.047,-36.6089"/>
</g>
<!-- rhs -->
<g id="node3" class="node"><title>rhs</title>
<ellipse fill="none" stroke="black" cx="441.037" cy="-18" rx="141.075" ry="18"/>
<text text-anchor="middle" x="441.037" y="-14.3" font-family="Times,serif" font-size="14.00">*ast.BasicLit (Kind: INT) (Value: 1)</text>
</g>
<!-- args1&#45;&gt;rhs -->
<g id="edge2" class="edge"><title>args1&#45;&gt;rhs</title>
<path fill="none" stroke="black" d="M325.075,-73.1159C346.195,-63.26 373.566,-50.4867 396.412,-39.825"/>
<polygon fill="black" stroke="black" points="397.988,-42.9522 405.57,-35.5516 395.028,-36.6089 397.988,-42.9522"/>
</g>
</g>
</svg>

</div>


The graph is traversed in a depth-first manner, starting from the top. The values of the program flow from bottom to up. When the program runs, it starts right at the top. The node will not be resolved until the dependent nodes have been evaluated. The arrows point to what each node depends on. So for example, the value of the `*ast.BinaryExpr` node is dependent on the values of `*ast.BasicLit (Kind: INT)`. Since we know both values are `1`, and we know what `+` does, we know that the value at the node `*ast.BinaryExpr` is `2`.

## Equations As Graphs ##

Now why did we spend all that time show 1+1 in graph form? Well, it's because deep learning is really in its core, just a bunch of mathematical equations. Wait, don't go yet! It's not that scary. I am personally of the opinion that one can't really do deep learning (or any machine learning, really) without understanding the mathematics behind it. And in my experience there hasn't been a better way to learn it than visually, if only to internalize the concepts.

Most deep learning libraries like [Tensorflow](https://tensorflow.org), [Theano](https://deeplearning.org/theano), or even my own for Go - [Gorgonia](https://github.com/chewxy/gorgonia), rely on this core concept that equations are representable by graphs. More importantly, these libraries expose the equation graphs as objects that can be manipulated by the programmer.

So instead of the program above, we'd create something like this:

```go
func main() {
	// create a graph
	g := G.NewGraph()                         
	
	// create a node called "x" with the value 1
	x := G.NodeFromAny(g, 1, G.WithName("x")) 
	
	// create a node called "y" with the value 1
	y := G.NodeFromAny(g, 1, G.WithName("y")) 
	
	// z := x + y
	z := G.Must(G.Add(x, y))                  

	// create a VM to execute the graph
	vm := G.NewTapeMachine(g) 
	// Run the VM. Errors are not checked.
	vm.RunAll()               

	// print the value of z
	fmt.Printf("%v", z.Value()) 
}
```

The equation graph looks like this:

<div style="margin-left:auto; margin-right:auto;">
<svg style="width:100%; height:auto;"
 viewBox="0.00 0.00 715.00 360.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 356)">
<title>fullGraph</title>
<polygon fill="white" stroke="none" points="-4,4 -4,-356 1109.3,-356 1109.3,4 -4,4"/>
<g id="clust1" class="cluster"><title>cluster_expressionGraph</title>
<polygon fill="none" stroke="black" points="8,-192 8,-344 574,-344 574,-192 8,-192"/>
<text text-anchor="middle" x="291" y="-328.8" font-family="Times,serif" font-size="14.00">expressionGraph</text>
</g>
<!-- Node_0xc420146300 -->
<g id="node1" class="node"><title>Node_0xc420146300</title>
<polygon fill="none" stroke="#ff0000" points="206,-287.5 206,-308.5 254,-308.5 254,-287.5 206,-287.5"/>
<text text-anchor="start" x="225.5" y="-294.3" font-family="monospace" font-size="14.00">2</text>
<polygon fill="none" stroke="#ff0000" points="254,-287.5 254,-308.5 558,-308.5 558,-287.5 254,-287.5"/>
<text text-anchor="start" x="257" y="-294.3" font-family="monospace" font-size="14.00">+ false(%a8d96c31, %82d6f1c8) :: int</text>
<polygon fill="none" stroke="#ff0000" points="206,-266.5 206,-287.5 254,-287.5 254,-266.5 206,-266.5"/>
<text text-anchor="start" x="221.5" y="-273.3" font-family="monospace" font-size="14.00">Op</text>
<polygon fill="none" stroke="#ff0000" points="254,-266.5 254,-287.5 558,-287.5 558,-266.5 254,-266.5"/>
<text text-anchor="start" x="323" y="-273.3" font-family="monospace" font-size="14.00">+ false :: a → a → a</text>
<polygon fill="none" stroke="#ff0000" points="206,-245.5 206,-266.5 558,-266.5 558,-245.5 206,-245.5"/>
<text text-anchor="start" x="373.5" y="-252.3" font-family="monospace" font-size="14.00">()</text>
<polygon fill="none" stroke="#ff0000" points="206,-224.5 206,-245.5 558,-245.5 558,-224.5 206,-224.5"/>
<text text-anchor="start" x="373.5" y="-231.3" font-family="monospace" font-size="14.00">&#45;1</text>
<polygon fill="none" stroke="#ff0000" points="206,-203.5 206,-224.5 254,-224.5 254,-203.5 206,-203.5"/>
<text text-anchor="start" x="209" y="-210.3" font-family="monospace" font-size="14.00">Value</text>
<polygon fill="none" stroke="#ff0000" points="254,-203.5 254,-224.5 558,-224.5 558,-203.5 254,-203.5"/>
<text text-anchor="start" x="377" y="-210.3" font-family="monospace" font-size="14.00">2 :: int</text>
</g>
<!-- Node_0xc420146000 -->
<g id="node2" class="node"><title>Node_0xc420146000</title>
<polygon fill="lightyellow" stroke="none" points="217.5,-4 217.5,-88 338.5,-88 338.5,-4 217.5,-4"/>
<polygon fill="none" stroke="#00ff00" points="218,-67 218,-88 266,-88 266,-67 218,-67"/>
<text text-anchor="start" x="237.5" y="-73.8" font-family="monospace" font-size="14.00">0</text>
<polygon fill="none" stroke="#00ff00" points="266,-67 266,-88 339,-88 339,-67 266,-67"/>
<text text-anchor="start" x="269" y="-73.8" font-family="monospace" font-size="14.00">x :: int</text>
<polygon fill="none" stroke="#00ff00" points="218,-46 218,-67 339,-67 339,-46 218,-46"/>
<text text-anchor="start" x="270" y="-52.8" font-family="monospace" font-size="14.00">()</text>
<polygon fill="none" stroke="#00ff00" points="218,-25 218,-46 339,-46 339,-25 218,-25"/>
<text text-anchor="start" x="270" y="-31.8" font-family="monospace" font-size="14.00">&#45;1</text>
<polygon fill="none" stroke="#00ff00" points="218,-4 218,-25 266,-25 266,-4 218,-4"/>
<text text-anchor="start" x="221" y="-10.8" font-family="monospace" font-size="14.00">Value</text>
<polygon fill="none" stroke="#00ff00" points="266,-4 266,-25 339,-25 339,-4 266,-4"/>
<text text-anchor="start" x="273.5" y="-10.8" font-family="monospace" font-size="14.00">1 :: int</text>
</g>
<!-- Node_0xc420146300&#45;&gt;Node_0xc420146000 -->
<g id="edge1" class="edge"><title>Node_0xc420146300:anchor:s&#45;&gt;Node_0xc420146000:anchor:n</title>
<path fill="none" stroke="black" d="M382,-202.5C382,-137.689 288.68,-155.213 278.837,-99.0855"/>
<polygon fill="black" stroke="black" points="282.315,-98.6762 278,-89 275.339,-99.2553 282.315,-98.6762"/>
<text text-anchor="middle" x="385.75" y="-191.3" font-family="Times,serif" font-size="14.00"> 0 </text>
</g>
<!-- Node_0xc420146240 -->
<g id="node3" class="node"><title>Node_0xc420146240</title>
<polygon fill="lightyellow" stroke="none" points="426.5,-4 426.5,-88 547.5,-88 547.5,-4 426.5,-4"/>
<polygon fill="none" stroke="#00ff00" points="427,-67 427,-88 475,-88 475,-67 427,-67"/>
<text text-anchor="start" x="446.5" y="-73.8" font-family="monospace" font-size="14.00">1</text>
<polygon fill="none" stroke="#00ff00" points="475,-67 475,-88 548,-88 548,-67 475,-67"/>
<text text-anchor="start" x="478" y="-73.8" font-family="monospace" font-size="14.00">y :: int</text>
<polygon fill="none" stroke="#00ff00" points="427,-46 427,-67 548,-67 548,-46 427,-46"/>
<text text-anchor="start" x="479" y="-52.8" font-family="monospace" font-size="14.00">()</text>
<polygon fill="none" stroke="#00ff00" points="427,-25 427,-46 548,-46 548,-25 427,-25"/>
<text text-anchor="start" x="479" y="-31.8" font-family="monospace" font-size="14.00">&#45;1</text>
<polygon fill="none" stroke="#00ff00" points="427,-4 427,-25 475,-25 475,-4 427,-4"/>
<text text-anchor="start" x="430" y="-10.8" font-family="monospace" font-size="14.00">Value</text>
<polygon fill="none" stroke="#00ff00" points="475,-4 475,-25 548,-25 548,-4 475,-4"/>
<text text-anchor="start" x="482.5" y="-10.8" font-family="monospace" font-size="14.00">1 :: int</text>
</g>
<!-- Node_0xc420146300&#45;&gt;Node_0xc420146240 -->
<g id="edge2" class="edge"><title>Node_0xc420146300:anchor:s&#45;&gt;Node_0xc420146240:anchor:n</title>
<path fill="none" stroke="black" d="M382,-202.5C382,-137.404 476.218,-155.453 486.155,-99.1258"/>
<polygon fill="black" stroke="black" points="489.656,-99.2565 487,-89 482.68,-98.6742 489.656,-99.2565"/>
<text text-anchor="middle" x="374.5" y="-191.3" font-family="Times,serif" font-size="14.00"> 1 </text>
</g>
<!-- outsideRoot -->
<!-- insideInputs -->
<!-- outsideRoot&#45;&gt;insideInputs -->
<!-- outsideExprG -->
<!-- outsideRoot&#45;&gt;outsideExprG -->
<!-- insideExprG -->
<!-- insideInputs&#45;&gt;insideExprG -->
<!-- outsideExprG&#45;&gt;insideExprG -->
</g>
</svg>


</div>


## Why Graph Objects? ##

So far you might be thinking - if all computer programs are graphs, and all mathematical equations are graphs, we could just program the mathematical equations in, right? Why write the above example with object graphs when a simple `fmt.Printf("%v", 1+1)` would do? Afterall, the graphs are mostly the same. Having a graph object at this point seem like a silly amount of overhead.

You would be right. For simple equations, having a programmer manipulatable graph object is really overkill (unless you live in Java land, where it's classes all the way down). 

I would however, posit at least three advantages of having a graph object. All of these have got to do with reducing human errors.

### Numerical Stability ###

Consider the equation $y = log(1 + x)$. This equation is not [numerically stable](https://en.wikipedia.org/wiki/Numerical_stability) - for very small values of `x`, the answer will most likely be wrong. This is because of the way `float64` is designed - a `float64` does not have enough bits to be able to tell apart `1` and `1 + 10e-16`. In fact, the correct way to do $latex y = log(1 + x)$ is to use the built in library function `math.Log1p`. It can be shown in this simple program:

```go
func main() {
	fmt.Printf("%v\n", math.Log(1.0+10e-16))
	fmt.Printf("%v\n", math.Log1p(10e-16))
}

1.110223024625156e-15 // wrong
9.999999999999995e-16 // correct
```

Now, of course the programmer may be well aware of this issue and opt to use `math.Log1p` when implementing neural network algorithms, but I'm sure you'd agree that having a library that automatically converts `log(1+x)` to use `math.Log1p(x)` be rather awesome? It reduces the element of human error.

### Machine-Learning Specific Optimizations ###

Consider a variant of the first program in this post:

```go
func a() int {
	return 1 + 1
}
```

This is the same program compiled down to assembly:

```
"".a t=1 size=10 args=0x8 locals=0x0
	(main.go:5)	TEXT	"".a(SB), $0-8
	(main.go:5)	FUNCDATA	$0, gclocals·2a5305abe05176240e61b8620e19a815(SB)
	(main.go:5)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	(main.go:6)	MOVQ	$2, "".~r0+8(FP)
	(main.go:6)	RET
```

In particular, pay attention to the second last line: `MOVQ	$2, "".~r0+8(FP)`. The function has been optimized in such a way that `2` is returned. No addition operation will be performed at run time. This is because the compiler knows, *at compile time*, that 1 + 1 = 2. By replacing the expression with a constant, the compiler is saving on computation cycles at run time. If you're interested in building compilers, this is known as [constant folding](https://en.wikipedia.org/wiki/Constant_folding).

So, we've established that compilers are smart enough to do optimizations. But the Go compiler (and in fact most non-machine-learning specific compilers) isn't smart enough to handle values that are used for machine learning. For machine learning, we frequently use array-based values, like a slice of `float64`s, or a matrix of `float32`s. 

Imagine if you will, if you're not doing `1 + 1`. Instead you're doing `[]int{1, 1, 1} + []int{1,1,1}`. The compiler wouldn't be able to optimize this and just replace it with `[]int{2, 2, 2}`. But building a graph object that can be optimized allows users to do just that. Gorgonia currently doesn't do constant folding yet (earlier versions had constant folding but it is quite difficult to get right), but it comes with other forms of graph optimizations like [common expression elimination](https://en.wikipedia.org/wiki/Common_subexpression_elimination), some amount of variable elimination and some minimal form of tree shaking. Other more mature libraries like TensorFlow or Theano comes with very many optimization algorithms for their graphs.

Again, one could argue that this could be done by hand, and coding it into the program would be more than doable. But is this really where you'd rather spend your time and effort? Or would you rather be creating the coolest new deep learning stuff?

### Backpropagation ###

Lastly, and most importantly, the graph object is most important in the implementation of backpropagation. In the following posts, I will talk about backpropagation and how it relates to computing partial derivatives. It's there we shall see the true power of having graph objects that are manipulatable by the programmer.

In a far future post, I shall also touch on the capacity to generate better code, using the graph object.


## Why Go? ##

If you're working on a truly homoiconic language such as lisp or Julia, you probably wouldn't need a graph object. If you could have access to the program's internal data structures and they're modifiable on the fly at run time, you would be able to augment plenty of the operations on the fly (yes, you can do the same for Go, but why would you?). This would make backpropagation algorithms a lot simpler to perform at runtime. Unfortunately this isn't the case. Which is why we'd have to build up extra data structures for deep learning. 

Do note that this isn't a knock on Go or Python or Lua. All of these languages have their strengths and weaknesses. But why do deep learning related work in Go when there are more mature libraries in Python or Lua? Well, one of the major reasons I developed Gorgonia was the ability to deploy everything neatly into one single binary. Doing that with Python or Lua would take an immense amount of effort. By contrast, deploying Go programs are a breeze. 

I believe that Go for data science is an amazing idea. It is typesafe (enough for me), and it's compiled down to binary. Go allows for better mechanical sympathy, which I believe is key to having faster and better AI out there. Afterall, we are ALL bound by our hardware. I just wish there were better higher level data structures for me to express my ideas. There weren't, so I built them. And I hope you use them.