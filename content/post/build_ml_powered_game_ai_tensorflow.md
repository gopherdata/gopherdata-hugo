+++
title = "Building an ML-Powered Game AI using TensorFlow in Go"
subtitle = ""
date = "2017-08-10T08:43:45+10:00"
math = false
draft = false

+++
*Based on a lightning talk given at GopherCon 2017 "Building an ML-Powered Game AI using TensorFlow in Go" [Video](https://www.youtube.com/watch?v=oiorteQg9n0) / [Slides](https://github.com/gophercon/2017-talks/tree/master/lightningtalks/PeteGarcin-BuildingMLPoweredGameAIwithTensorFlow)*

(Author: Pete Garcin, Developer Advocate @ [ActiveState](https://activestate.com), @rawktron on [Twitter](https://twitter.com/rawktron) and @peteg on Gophers Slack)  

For GopherCon, we wanted to demonstrate some of the capabilities of the emerging machine learning and data science ecosystem in Go. Originally built as a demo for PyCon, I had put together a simple arcade space shooter game that features enemies powered by machine learning. It was a fun way to get folks engaged at conferences and to learn about the growing library of tools that are available. It also gave me an opportunity to build something non-trivial using machine learning techniques, and my background in games made this kind of interactive demo a good fit.

NeuroBlast is a vertically scrolling space shooter where you control a ship that tries to defeat increasing waves of enemies. Normally, these enemies fly in predefined formations, with predefined firing patterns, and come in waves. The big difference in NeuroBlast is that the enemies use machine learning to determine what their firing pattern should be.

## Under the Hood: TensorFlow
Go is one of the languages that has a TensorFlow client available, and so it was a great opportunity to port the original Python game to Go and demonstrate that it’s also possible to deploy trained models in Go applications.

It is worth noting that the Go TensorFlow client currently does not support training models, and so we relied on a model that was previously trained using the Python version of the game and then exported using the `SavedModelBuilder` functionality in order to load it in Go. This will export a TensorFlow graph as a protocol buffer and allow it to be loaded in Go using the `LoadSavedModel` function.

For the game portion, I used a library called [Pixel](https://github.com/faiface/pixel) which is still early in development but has a really active community, and offered excellent stability and performance. I was pretty performance conscious when building and porting the game, so there are certain limitations such as non-pixel-perfect collisions, in order to ensure that the game could run acceptably under all conditions.

### Training the Neural Net
Our Neural Net is ultimately a very simple one -- four inputs and a single output neuron. It will use supervised learning to do binary classification on a simple problem: was each shot a hit or a miss? It utilizes the delta between player and enemy position, and player and enemy velocity as the inputs. The single output neuron will fire if its activation value is >= 0.5 and will not fire if it is < 0.5.

When building the network, I initially had only a single hidden layer with 4 nodes but found that after training it, it was somewhat erratic. It seemed like it was very sensitive to the training data and would not ‘settle’ on a particular strategy in any consistent way. I experimented with a few different configurations, and ultimately settled on the one we used for the demo. It’s quite likely not the optimal setup, and may have more layers than is necessary. What appealed to me though was that even with a very small amount of training data, and regardless of how you trained it, it would consistently settle into a similar behaviour pattern which made it great for a floor demo where anyone could play or train the game.

![Neural Network Visualization](/img/gameai/NNViz.png "Neural Net Visualization")
*The inputs are the four nodes at the top, with the output node (primed to fire here) at the bottom. Thicker lines represent higher weights. Activation values appear in the node centers.*

The visualization was cobbled together by myself to run using the Pixel immediate mode drawing functions. Inspired by [this blog post](https://medium.com/deep-learning-101/how-to-generate-a-video-of-a-neural-network-learning-in-python-62f5c520e85c), the visualization here shows connections between nodes as either red or green lines. Green lines indicate positive weights that will bias the network towards “shooting”, and red values inhibit shooting. Thicker lines indicate higher weight values and thus “stronger” connections.

After training, the network consistently seems to converge on the following strategy:

If the player is within a reasonable cone of forward “vision” then fire indiscriminately.
If the player is not within that reasonable forward cone, then do not fire at all.

At first, it was very interesting to me that the network did not settle on the more obvious “just fire constantly” strategy, but given that it does receive training data that indicates that “misses” are undesirable, it makes sense that it would avoid firing shots with a low probability of hitting.

![Positive Network Output](/img/gameai/GopherPos.png "Positive Network Output")
*In this image, notice that because I’m in a forward cone, most of the enemies are firing at my indiscriminately.*

![Negative Network Output](/img/gameai/GopherNeg.png "Negative Network Output")
*In this image, you’ll notice none of the enemies are firing because I am clearly outside their range of possible success. However, if you look at the activation values, you’ll see that the enemy who has just come onto the screen is about to blast me because his output is 0.83. Yikes!*

In the training mode, the enemies fire randomly, and then each shot taken by the enemy is recorded as a hit or a miss along with its initial relative position/velocity values.

It’s worth noting that in early iterations of the game, I was passing in raw pixel values for positions and velocities. This meant that there was a really wide variation between the values in the input and I found that the network would just not really converge to a consistent behaviour. So, I normalized the input data to be roughly between 0.0-1.0 and found that it basically instantly converged to a usable behaviour. So, lesson for you kids: normalize your input data!

It's also important that you make all your input values framerate independent since the model is being trained in Python, any discrepancies between either the co-ordinate space or velocity values when running in Go will result in getting incorrect results back from the network.

Once the network is trained, when you play the game, every instance of an enemy spaceship uses its own instance of the neural network to make decisions about when it should fire.

### Using the model in Go

As mentioned above, we exported our model from Python using the SavedModelBuilder and then need to load this in Go. The `SavedModelBuilder` will export the model into a folder, and you only need to specify that folder when loading in Go, along with the tag for your model. There are two available tags - TRAIN and SERVING. In our case, I used TRAIN but for deployment, it is suggested that you use the SERVING tag.

```Go
    // bundle contains Session + Graph
    bundle, err := tf.LoadSavedModel("exported_brain", []string{"train"}, nil)
```

This code will load your saved model and return a struct that contains pointers to the TensorFlow graph, and a new TensorFlow session that you can use for evaluation. However, at this stage many are left wondering - so, now what?

This is where current limitations in the Go binding and documentation show themselves. The key next step is to access the nodes in the TensorFlow graph for the input and output nodes. Right now, the only way to do this is to print the graph node names from Python when you are exporting. You do have the option of labelling nodes as you output them, but don't have the ability to access those nodes by their labels from Go.

In this case, once we have the names of the nodes, we need to access them in Go via the `Operation` method on the TensorFlow graph. Remember that because TensorFlow is storing our network as a series of computation operations, that's why it is called `Operation`.

```Go
    inputop := bundle.Graph.Operation("dense_1_input")
    outputop := bundle.Graph.Operation("dense_5/Sigmoid")
```

So now, we have both a Session with our Graph, and we have also found our input and output Operations, so now we're ready to send data to the graph and use it to evaluate that data. In our case, since we've trained the graph using the relative position and velocity in the game, each frame, for each enemy we will take their relative position and velocity to the player and feed it into the network to get a decision back on whether or not we should fire at the player.  

Our first step in having our enemies "think" is to build a new 'Tensor' (which is like a vector) to feed into the network:

```Go
    var column *tf.Tensor
    column, err = tf.NewTensor([1][4]float32{{dx, dy, du, dv}})
    results, err := bundle.Session.Run(map[tf.Output]*tf.Tensor{inputop.Output(0): column}, []tf.Output{outputop.Output(0)}, nil)
```

What you see above is that we're building a tensor from the relative position (dx, dy) and the relative velocity (du, dv) and then running our TensorFlow session, specifying which nodes are the input and output nodes. The session will then return the results in the form of an array. In our case, since we only have one output node, the array only has one entry.  

Our enemies have almost made a decision - all we need to do is read the output value from their neuron. If value at the output node is greater than or equal to 0.5 we use that threshold to determine that we should fire at the player:

```Go
    for _, result := range results {
        if result.Value().([][]float32)[0][0] >= 0.5 && enemy.canfire {
           // FIRE!!
        }
    }
```

And that's it! This is literally the only logic governing the attack behaviour of the enemies, and despite its simplicity, this network generates a compelling and interesting set of enemy behaviour. By applying this repeatedly to all enemies, we've been able to create an enemy AI for our game.  

There are obviously lots of easy and obvious ways to improve this, and to make it more sophisticated - but as a demonstration of utilizing a model loaded at runtime in Go and 'deployed' into our game - it demonstrates the ease and power of these techniques and how accessible the tools are making them.

## Next Steps

The Machine Learning and Data Science ecosystem in Go is growing, and there are lots of exciting opportunties to contribute to a wide variety of projects - including TensorFlow.

In the near future, I plan to push this to GitHub as I had a number of requests at both GopherCon and PyCon to do so and would love to see others learn from this project as well as help to develop and expand its capabilities. 

*Update 08/29/17 - Full Source Code is now live on [GitHub](https://github.com/ActiveState/neuroblast)!*

If you want to ask questions about this project, feel free to hit me up on Twitter [@rawktron](https://twitter.com/rawktron).
