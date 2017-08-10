+++
date = "2017-04-26T12:00:00"
draft = false
tags = ["academic", "hugo"]
title = "Building a distributed Trump finder"
math = false
summary = """A step-by-step guide to building a distributed facial recognition system with Pachyderm and Machine Box.
"""

[header]
image = "headers/trump_finder.jpg"
caption = ""

+++

(Author: Daniel Whitenack, @dwhitena on [Twitter](https://twitter.com/dwhitena) and Gophers Slack)

If you haven't heard, there is a new kid on the machine learning block named [Machine Box](https://machinebox.io/).  It's pretty cool, and you should check it out.  Machine Box provides pre-built Docker images that enable easy, production ready, and reproducible machine learning operations.  For example, you can get a "facebox" Docker image from Machine Box for facial recognition. When you run "facebox," you get a full JSON API that lets you to easily "teach" facebox certain people's faces, identify those faces in images, and persist the "state" of trained facial recognition models.

Experimenting with "facebox" got me thinking about how it could be integrated into some of my workflows.  In particular, I wanted to see how Machine Box images could be utilized as part of distributed data processing pipeline built with [Pachyderm](http://pachyderm.io/).  Pachyderm builds, runs, and manages pipelines, such as machine learning workflows, based on Docker images.  Thus, an integration of Machine Box images seems to be only natural.

And that's how the first (to my knowledge) distributed, Docker-ized, "Trump-finding" data pipeline came to be. In the sections below we will walk through the creation of a facial recognition pipeline that is able to find and tag the location of Donald Trump's face in images.  Actually, this pipeline could be used to identify any faces, and we will illustrate this flexibility by updating the pipeline to learn a second face, Hillary Clinton.

The below sections assume that you have a Pachyderm cluster running, that `pachctl`(the Pachyderm CLI tool) is connected to that cluster, and that you have signed up for a Machine Box key (which you can do for free).  All the code and more detailed instructions can be found [here](https://github.com/dwhitena/pach-machine-box).

**Create the pipeline inputs**:

To train our facial recognition model, identify faces in images, and tag those faces with certain labels, we need to create three "data repositories" that will be the inputs to our Pachyderm pipeline:

1. `training` - which includes images of faces that we use to "teach" facebox
2. `unidentified` - which includes images with faces we want to detect and identify
3. `labels` - which includes label images that we will overlay on the unidentified images to indicate identified faces

This can be done with `pachctl`:

```sh
➔ pachctl create-repo training
➔ pachctl create-repo unidentified
➔ pachctl create-repo labels
➔ pachctl list-repo
NAME                CREATED             SIZE                
labels              3 seconds ago       0 B                 
unidentified        11 seconds ago      0 B                 
training            17 seconds ago      0 B
➔ cd data/train/faces1/
➔ ls
trump1.jpg  trump2.jpg  trump3.jpg  trump4.jpg  trump5.jpg
➔ pachctl put-file training master -c -r -f .
➔ pachctl list-repo
NAME                CREATED             SIZE                
training            5 minutes ago       486.2 KiB           
labels              5 minutes ago       0 B                 
unidentified        5 minutes ago       0 B                 
➔ pachctl list-file training master
NAME                TYPE                SIZE                
trump1.jpg          file                78.98 KiB           
trump2.jpg          file                334.5 KiB           
trump3.jpg          file                11.63 KiB           
trump4.jpg          file                27.45 KiB           
trump5.jpg          file                33.6 KiB
➔ cd ../../labels/
➔ ls
clinton.jpg  trump.jpg
➔ pachctl put-file labels master -c -r -f .
➔ cd ../unidentified/
➔ ls
image1.jpg  image2.jpg
➔ pachctl put-file unidentified master -c -r -f .
➔ pachctl list-repo
NAME                CREATED             SIZE                
unidentified        7 minutes ago       540.4 KiB           
labels              7 minutes ago       15.44 KiB           
training            7 minutes ago       486.2 KiB
```

**Train, or "teach," facebox**:

Next, we create a Pachyderm pipeline stage that will take the `training` data as input, provide those training images to facebox, and output the "state" of a trained model.  This is done by providing Pachyderm with a pipeline spec, [train.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/train.json).  This pipeline spec specifies the Docker image to use for data processing, the commands to execute in that Docker container, and what data is input to the pipeline.  

In our particular case, `train.json` specifies that we should use an image based on the facebox image from Machine Box and execute a number of cURL commands to post the training data to facebox.  Once those training images are provided to and processed by facebox, we specify another cURL command to export the state of the facebox model (for use later in our pipeline).

We are using a little bash magic here to perform these operations.  However, it is very possible that, in the future, Machine Box will provide a more standardized command line implementation for these sorts of use cases.

```sh
create-MB-pipeline.sh  identify.json  tag.json  train.json
➔ ./create-MB-pipeline.sh train.json
➔ pachctl list-pipeline
NAME                INPUT               OUTPUT              STATE               
model               training            model/master        running    
➔ pachctl list-job
ID                                   OUTPUT COMMIT STARTED       DURATION RESTART PROGRESS STATE            
3425a7a0-543e-4e2a-a244-a3982c527248 model/-       9 seconds ago -        1       0 / 1    running
➔ pachctl list-job
ID                                   OUTPUT COMMIT                          STARTED       DURATION  RESTART PROGRESS STATE            
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a 5 minutes ago 5 minutes 1       1 / 1    success
➔ pachctl list-repo
NAME                CREATED             SIZE                
model               5 minutes ago       4.118 KiB           
unidentified        18 minutes ago      540.4 KiB           
labels              18 minutes ago      15.44 KiB           
training            19 minutes ago      486.2 KiB           
➔ pachctl list-file model master
NAME                TYPE                SIZE                
state.facebox       file                4.118 KiB
```

As you can see the output of this pipeline is a `.facebox` file that contained the trained state of our facebox model.

**Use the trained facebox to identify faces**:

We then launch another Pachyderm pipeline, based on an [identify.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/identify.json) pipeline specification, to identify faces within the `unidentified` images.  This pipeline will take the persisted state of our model in `model` along with the `unidentified` images as input.  It will also execute cURL commands to interact with facebox, and it will output indications of identified faces to JSON files, one per `unidentified` image.

```sh
➔ ./create-MB-pipeline.sh identify.json
➔ pachctl list-job
ID                                   OUTPUT COMMIT                          STARTED       DURATION  RESTART PROGRESS STATE            
281d4393-05c8-44bf-b5de-231cea0fc022 identify/-                             6 seconds ago -         0       0 / 2    running
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a 8 minutes ago 5 minutes 1       1 / 1    success
➔ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED            DURATION   RESTART PROGRESS STATE            
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 About a minute ago 53 seconds 0       2 / 2    success
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    9 minutes ago      5 minutes  1       1 / 1    success
➔ pachctl list-repo
NAME                CREATED              SIZE                
identify            About a minute ago   1.932 KiB           
model               10 minutes ago       4.118 KiB           
unidentified        23 minutes ago       540.4 KiB           
labels              23 minutes ago       15.44 KiB           
training            24 minutes ago       486.2 KiB           
➔ pachctl list-file identify master
NAME                TYPE                SIZE                
image1.json         file                1.593 KiB           
image2.json         file                347 B
```

If we look at the JSON output for, e.g., `image1.jpg`, we can see that there is a portion of the file that clearly identifies Donald Trump in the image along with the location and size of his face in the image:

```
{
	"success": true,
	"facesCount": 13,
	"faces": [
		...
		...
		{
			"rect": {
				"top": 175,
				"left": 975,
				"width": 108,
				"height": 108
			},
			"id": "58ff31510f7707a01fb3e2f4d39f26dc",
			"name": "trump",
			"matched": true
		},
		...
		...
	]
}
```

**Tagging identified faces in the images**:

We are most of the way there! We have identified Trump in the `unidentified` images, but the JSON output isn't the most visually appealling.  As such, let's overlay a label on the images at the location of Trump's face.  

To do this, we can use a [simple Go program](https://github.com/dwhitena/pach-machine-box/blob/master/tagimage/main.go) to draw the label image on the `unidentified` image at the appropriate location.  This part of the pipeline is specified by a [tag.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/tag.json) pipeline specification, and can be created as follows:

```sh
➔ pachctl create-pipeline -f tag.json
➔ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
cd284a28-6c97-4236-9f6d-717346c60f24 tag/-                                     2 seconds ago  -          0       0 / 2    running
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 5 minutes ago  53 seconds 0       2 / 2    success
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    13 minutes ago 5 minutes  1       1 / 1    success
➔ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      25 seconds ago 11 seconds 0       2 / 2    success
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 5 minutes ago  53 seconds 0       2 / 2    success
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    14 minutes ago 5 minutes  1       1 / 1    success
➔ pachctl list-repo
NAME                CREATED             SIZE                
tag                 30 seconds ago      591.3 KiB           
identify            5 minutes ago       1.932 KiB           
model               14 minutes ago      4.118 KiB           
unidentified        27 minutes ago      540.4 KiB           
labels              27 minutes ago      15.44 KiB           
training            27 minutes ago      486.2 KiB           
➔ pachctl list-file tag master
NAME                TYPE                SIZE                
tagged_image1.jpg   file                557 KiB             
tagged_image2.jpg   file                34.35 KiB           
```

As you can see, we now have two "tagged" versions of the images in the output `tag` data repository.  If we get these images, we can see that... Boom! Our Trump finder works:

![alt text](https://raw.githubusercontent.com/dwhitena/pach-machine-box/master/tagged_images1.jpg)

**Teaching a new face, updating the output**:

Our pipeline isn't restricted to Trump or any one face.  Actually, we can teach facebox another face by updating our `training`.  Moreover, because Pachyderm versions your data and knows what data is new, it can automatically update all our results once facebox learns the new face:

```sh
➔ cd ../data/train/faces2/
➔ ls
clinton1.jpg  clinton2.jpg  clinton3.jpg  clinton4.jpg
➔ pachctl put-file training master -c -r -f .
➔ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
56e24ac0-0430-4fa4-aa8b-08de5c1884db model/-                                   4 seconds ago  -          0       0 / 1    running
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      6 minutes ago  11 seconds 0       2 / 2    success
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 11 minutes ago 53 seconds 0       2 / 2    success
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    20 minutes ago 5 minutes  1       1 / 1    success
➔ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED            DURATION   RESTART PROGRESS STATE            
6aa6c995-58ce-445d-999a-eb0e0690b041 tag/7cbd2584d4f0472abbca0d9e015b9829      5 seconds ago      1 seconds  0       2 / 2    success
8a7961b7-1085-404a-b0ee-66034fae7212 identify/1bc94ec558e44e0cb45ed5ab7d9f9674 59 seconds ago     54 seconds 0       2 / 2    success
56e24ac0-0430-4fa4-aa8b-08de5c1884db model/002f16b63a4345a4bc6bdf5510c9faac    About a minute ago 19 seconds 0       1 / 1    success
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      8 minutes ago      11 seconds 0       2 / 2    success
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 13 minutes ago     53 seconds 0       2 / 2    success
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    21 minutes ago     5 minutes  1       1 / 1    success
➔ pachctl list-file tag master
NAME                TYPE                SIZE                
tagged_image1.jpg   file                557 KiB             
tagged_image2.jpg   file                36.03 KiB
```

Now if we look at our images, we find that everything has been updated without any annoying manual work on our hands:

![alt text](https://raw.githubusercontent.com/dwhitena/pach-machine-box/master/tagged_images2.jpg)

**Conclusion/Resources**:

As you can see, Machine Box and Pachyderm make it really quick and easy to deploy a distributed, machine learning data pipeline.  Be sure to:

- Visit [this repo](https://github.com/dwhitena/pach-machine-box) to get the code and pipeline specs, so you can create your own Trump finder!
- Join the [Pachyderm Slack team](http://slack.pachyderm.io/) to get help implementing your ML pipelines, and participate in the discussion in the #data-science channel on Gophers Slack.
- Follow [Pachyderm on Twitter](https://twitter.com/pachydermIO),
- Sign up for a free [Machine Box](https://machinebox.io/) API key, and
- Follow [Machine Box on Twitter](https://twitter.com/machineboxio).
